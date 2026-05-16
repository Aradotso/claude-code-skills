---
name: claude-code-agent-harness-architecture
description: Expert guidance on AI Agent architecture, harness design patterns, and building production-grade Agent systems based on Claude Code analysis
triggers:
  - "how do I build an AI agent harness"
  - "explain agent architecture patterns"
  - "design a production agent system"
  - "implement agent tool calling system"
  - "build agent permission pipeline"
  - "create agent context management"
  - "design agent conversation loop"
  - "implement agent memory system"
---

# Claude Code Agent Harness Architecture

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## What This Skill Covers

This skill provides deep architectural knowledge for building production-grade AI Agent systems (Agent Harness), based on the comprehensive analysis of Claude Code's architecture from the claude-code-book project. You'll learn:

- **Core Agent Harness patterns**: conversation loops, tool systems, permission pipelines
- **Context & memory management**: token budgeting, compression strategies, long-term memory
- **Multi-agent orchestration**: fork patterns, coordinator models, sub-agent spawning
- **Integration patterns**: MCP protocol, hooks system, skill/plugin architecture
- **Production concerns**: streaming, performance optimization, safety guardrails

## Installation & Access

The knowledge base is available at:
- **Online book**: https://lintsinghua.github.io
- **GitHub repository**: https://github.com/lintsinghua/claude-code-book

```bash
# Clone for offline reference
git clone https://github.com/lintsinghua/claude-code-book.git
cd claude-code-book

# The book contains 15 chapters + 4 appendices with 139 architecture diagrams
# Key directories:
# - 第一部分-基础篇/ (Part 1: Fundamentals)
# - 第二部分-核心系统篇/ (Part 2: Core Systems)
# - 第三部分-高级模式篇/ (Part 3: Advanced Patterns)
# - 第四部分-工程实践篇/ (Part 4: Engineering Practice)
# - 附录/ (Appendices: tools, flags, terminology)
```

## Core Architecture Patterns

### 1. Conversation Loop (Agent Heartbeat)

The fundamental `while(true)` async generator pattern that drives all Agent interactions:

```typescript
// Core conversation loop structure
async function* conversationLoop(deps: QueryDeps): AsyncGenerator<QueryEvent> {
  while (true) {
    // 1. Prepare context window (token budget management)
    const context = await buildContext(deps);
    
    // 2. Call LLM with tools
    const response = await llmClient.chat({
      messages: context.messages,
      tools: context.availableTools,
      max_tokens: context.tokenBudget
    });
    
    // 3. Yield different event types
    if (response.type === 'text') {
      yield { type: 'text_delta', delta: response.content };
    } else if (response.type === 'tool_use') {
      // 4. Execute tool with permission check
      const result = await executeToolWithPermission(response.tool, deps);
      yield { type: 'tool_result', result };
    }
    
    // 5. Check termination conditions (10 types)
    if (shouldTerminate(response)) {
      yield { type: 'done', reason: response.stopReason };
      break;
    }
  }
}

// Five yield event types:
// - text_delta: Streaming text chunks
// - tool_use: Tool invocation request
// - tool_result: Tool execution result
// - context_compression: Token budget overflow handling
// - done: Termination with reason
```

**Ten termination reasons**:
- `end_turn` - Natural completion
- `max_tokens` - Token limit reached
- `stop_sequence` - Stop token encountered
- `tool_use` - Waiting for tool result
- `content_filter` - Safety filter triggered
- `timeout` - Time limit exceeded
- `user_interrupt` - User cancellation
- `error` - Fatal error
- `rate_limit` - API quota exceeded
- `context_overflow` - Context window exhausted

### 2. Tool System (Agent's Hands)

Tools follow a strict 5-element protocol:

```typescript
// Tool<Input, Output, Permission> protocol
interface Tool<I, O, P extends PermissionLevel> {
  // 1. Schema: Zod v4 input validation
  schema: z.ZodType<I>;
  
  // 2. Handler: Core execution logic
  handler: (input: I, deps: QueryDeps) => Promise<O>;
  
  // 3. Metadata: LLM-visible description
  metadata: {
    name: string;
    description: string;
    parameters: JSONSchema; // Auto-generated from Zod
  };
  
  // 4. Properties: Execution characteristics
  properties: {
    readOnly: boolean;        // Safe to run speculatively
    destructive: boolean;     // Irreversible side effects
    concurrencySafe: boolean; // Can run in parallel
  };
  
  // 5. Permission: Required access level
  permission: P; // 'read' | 'write' | 'execute' | 'admin' | 'auto'
}

// buildTool: Fault-safe factory pattern
const readFileTool = buildTool({
  schema: z.object({
    path: z.string(),
    encoding: z.enum(['utf-8', 'base64']).default('utf-8')
  }),
  permission: 'read',
  properties: {
    readOnly: true,
    destructive: false,
    concurrencySafe: true
  },
  handler: async ({ path, encoding }, deps) => {
    const absPath = deps.workspace.resolvePath(path);
    
    // Safety check: Must be within workspace
    if (!deps.workspace.contains(absPath)) {
      throw new ToolError('PathOutsideWorkspace', { path });
    }
    
    return await deps.fs.readFile(absPath, encoding);
  }
});

// Tool categories (12 types, 50+ total tools):
// - File I/O: read_file, write_file, list_directory
// - Shell: execute_bash, run_command
// - Search: ripgrep_search, file_search
// - Edit: apply_diff, replace_in_file
// - Browser: navigate, screenshot, click
// - Git: commit, push, checkout
// - Memory: store_memory, recall_memory
// - Agent: fork_agent, spawn_worker
// - Config: update_settings, add_skill
// - MCP: call_mcp_tool, list_mcp_resources
```

**Concurrent execution algorithm** (partition-greedy):
```typescript
// Divide tools into read-only vs. side-effecting partitions
async function executeConcurrently(tools: ToolCall[]): Promise<ToolResult[]> {
  const [readOnly, sideEffecting] = partition(
    tools,
    t => t.properties.readOnly && t.properties.concurrencySafe
  );
  
  // Execute read-only tools in parallel
  const readOnlyResults = await Promise.all(
    readOnly.map(t => t.handler(t.input, deps))
  );
  
  // Execute side-effecting tools sequentially
  const sideEffectingResults = [];
  for (const tool of sideEffecting) {
    sideEffectingResults.push(await tool.handler(tool.input, deps));
  }
  
  return [...readOnlyResults, ...sideEffectingResults];
}
```

### 3. Permission Pipeline (4-Stage Guardrails)

```typescript
// Four-stage permission pipeline
type PermissionStage = 
  | 'classification'  // Speculative 2s pre-approval
  | 'validation'      // Schema + safety checks
  | 'authorization'   // User consent (if needed)
  | 'execution';      // Actual tool call

async function permissionPipeline(
  toolCall: ToolCall,
  mode: PermissionMode
): Promise<ToolResult> {
  // Stage 1: Classification (speculative, 2-second timeout)
  const classification = await Promise.race([
    classifyRisk(toolCall),
    timeout(2000, { risk: 'unknown' })
  ]);
  
  // Stage 2: Validation
  const validated = await validateToolCall(toolCall, {
    schema: toolCall.tool.schema,
    bashRules: mode === 'safe' ? SAFE_BASH_RULES : null,
    pathRestrictions: deps.workspace.boundaries
  });
  
  // Stage 3: Authorization (mode-dependent)
  if (requiresConsent(toolCall, mode, classification)) {
    const consent = await deps.ui.requestPermission({
      tool: toolCall.tool.name,
      input: toolCall.input,
      risk: classification.risk,
      preview: classification.preview
    });
    
    if (!consent.approved) {
      throw new PermissionDeniedError(consent.reason);
    }
  }
  
  // Stage 4: Execution
  return await toolCall.tool.handler(validated.input, deps);
}

// Five permission modes (spectrum)
type PermissionMode = 
  | 'safe'      // Auto-approve read-only, deny write/execute
  | 'normal'    // Ask for write, auto-approve read
  | 'relaxed'   // Auto-approve most, ask for destructive
  | 'auto'      // Auto-approve all (trust agent completely)
  | 'custom';   // User-defined rules

// Bash safety rules (safe mode)
const SAFE_BASH_RULES = {
  allowedCommands: ['ls', 'cat', 'grep', 'find', 'head', 'tail'],
  deniedPatterns: [
    /rm\s+-rf/,           // Destructive deletion
    /sudo/,               // Privilege escalation
    /curl.*\|\s*bash/,    // Pipe to shell
    />\s*\/dev\/sd/       // Direct disk access
  ],
  maxCommandLength: 500
};
```

### 4. Context Management (Token Budget)

Effective window calculation and 4-level compression:

```typescript
// Effective window formula
interface ContextWindow {
  total: number;              // Model's max context (200K for Claude 3.5)
  systemPrompt: number;       // ~2K tokens
  toolDefinitions: number;    // ~50 tokens per tool × N tools
  reserved: number;           // ~4K for output buffer
  effective: number;          // Available for conversation history
}

function calculateEffectiveWindow(deps: QueryDeps): ContextWindow {
  const total = deps.model.contextLimit; // 200_000
  const systemPrompt = estimateTokens(deps.systemPrompt);
  const toolDefinitions = deps.tools.length * 50;
  const reserved = 4_000;
  
  return {
    total,
    systemPrompt,
    toolDefinitions,
    reserved,
    effective: total - systemPrompt - toolDefinitions - reserved
  };
}

// Four-level progressive compression
async function compressContext(
  messages: Message[],
  targetTokens: number
): Promise<Message[]> {
  let compressed = messages;
  let currentTokens = estimateTokens(compressed);
  
  // Level 1: Snip old image content (images are 1500+ tokens each)
  if (currentTokens > targetTokens) {
    compressed = snipImages(compressed, { keepRecent: 3 });
    currentTokens = estimateTokens(compressed);
  }
  
  // Level 2: Micro-compact (merge adjacent same-role messages)
  if (currentTokens > targetTokens) {
    compressed = microCompact(compressed);
    currentTokens = estimateTokens(compressed);
  }
  
  // Level 3: Collapse (summarize tool results)
  if (currentTokens > targetTokens) {
    compressed = await collapseToolResults(compressed, {
      summarizeOlderThan: 10, // Message index threshold
      maxSummaryTokens: 200
    });
    currentTokens = estimateTokens(compressed);
  }
  
  // Level 4: Auto-compact (LLM summarization)
  if (currentTokens > targetTokens) {
    compressed = await autoCompact(compressed, {
      targetRatio: 0.5, // Compress to 50%
      preserveRecent: 5 // Keep last 5 exchanges intact
    });
  }
  
  return compressed;
}

// Circuit breaker pattern (prevent infinite compression)
class CompressionCircuitBreaker {
  private failureCount = 0;
  private lastReset = Date.now();
  
  async attempt<T>(fn: () => Promise<T>): Promise<T> {
    if (this.failureCount >= 3) {
      if (Date.now() - this.lastReset < 60_000) {
        throw new Error('Compression circuit breaker open');
      }
      this.reset();
    }
    
    try {
      const result = await fn();
      this.reset();
      return result;
    } catch (error) {
      this.failureCount++;
      throw error;
    }
  }
  
  private reset() {
    this.failureCount = 0;
    this.lastReset = Date.now();
  }
}
```

### 5. Memory System (Long-term Memory)

Four closed-type memory categories:

```typescript
// Memory types (closed set - only these four)
type MemoryType = 
  | 'code_patterns'    // Architecture decisions, idioms
  | 'user_preferences' // Settings, workflow habits
  | 'project_context'  // Goals, constraints, history
  | 'task_progress';   // Ongoing work state

interface Memory {
  type: MemoryType;
  key: string;           // Unique identifier
  content: string;       // Natural language
  metadata: {
    created: string;     // ISO timestamp
    accessed: string;    // Last retrieval
    confidence: number;  // 0-1, for pruning
  };
}

// "Only store what cannot be derived" principle
async function shouldStore(content: string, deps: QueryDeps): Promise<boolean> {
  // Don't store if:
  // - Can be read from files
  // - Can be inferred from code
  // - Is ephemeral state
  
  const derivable = [
    () => deps.workspace.files.includes(content), // File content
    () => /^(let|const|function)/.test(content),  // Code snippets
    () => isTemporary(content)                     // Timestamps, PIDs
  ];
  
  return !derivable.some(check => check());
}

// MEMORY.md index structure
/*
# Agent Memory

## Code Patterns
- [architecture-decision-rest-api] Decided to use REST over GraphQL
- [error-handling-strategy] All errors use custom Error subclasses

## User Preferences
- [editor-choice] User prefers VSCode with Vim keybindings
- [commit-style] Conventional commits with emoji prefixes

## Project Context
- [migration-status] Currently migrating from Express to Fastify
- [tech-debt] Known issue: N+1 queries in user dashboard

## Task Progress
- [feature-auth] OAuth2 implementation 60% complete
*/

// Fork memory mechanism (sub-agent inherits parent context)
async function forkAgent(parentDeps: QueryDeps): Promise<Agent> {
  const childDeps = {
    ...parentDeps,
    memory: {
      // Byte-level context inheritance
      messages: structuredClone(parentDeps.memory.messages),
      // Selective memory copying (only project_context + code_patterns)
      stored: parentDeps.memory.stored.filter(
        m => ['project_context', 'code_patterns'].includes(m.type)
      ),
      // Fresh task_progress
      taskState: {}
    }
  };
  
  return new Agent(childDeps);
}
```

### 6. Hook System (Lifecycle Extension)

26 lifecycle events across 5 categories:

```typescript
// Five hook categories
type HookCategory = 
  | 'lifecycle'    // conversation_start, query_begin, query_end
  | 'tool'         // tool_before, tool_after, tool_error
  | 'content'      // message_sent, message_received, context_compressed
  | 'system'       // config_changed, skill_loaded, mcp_connected
  | 'agent';       // agent_forked, agent_terminated

interface Hook {
  event: string;        // e.g., 'tool_before_execute'
  handler: string;      // Path to executable
  config?: {
    priority: number;   // 0-5 (higher = earlier)
    async: boolean;     // Run without blocking
    timeout: number;    // Max execution time (ms)
  };
}

// JSON response protocol
interface HookResponse {
  allow?: boolean;        // For before_* hooks: veto execution
  modify?: {              // For content hooks: transform data
    input?: unknown;
    output?: unknown;
  };
  metadata?: Record<string, unknown>; // Arbitrary data for next hooks
}

// Example: Audit hook (log all tool executions)
// .claude/hooks/audit-tools.ts
export default async function auditHook(event: HookEvent): Promise<HookResponse> {
  if (event.type === 'tool_before_execute') {
    await fs.appendFile(
      '.claude/audit.log',
      JSON.stringify({
        timestamp: new Date().toISOString(),
        tool: event.data.tool.name,
        input: event.data.input,
        user: process.env.USER
      }) + '\n'
    );
  }
  
  return { allow: true }; // Don't block execution
}

// Security: Three-layer sandbox
// 1. Process isolation (separate Node.js process)
// 2. Permission restrictions (can't access parent env vars)
// 3. Timeout enforcement (default 5s)
```

### 7. Sub-Agent Fork Pattern

Byte-level context inheritance with recursive protection:

```typescript
// Three agent sources
type AgentSource = 
  | 'builtin'    // Four built-in agents: coordinator, worker, researcher, reviewer
  | 'skill'      // Defined in SKILL.md files
  | 'fork';      // Cloned from current agent

// Fork with context inheritance
async function forkCurrentAgent(
  purpose: string,
  deps: QueryDeps
): Promise<Agent> {
  // Recursive fork depth protection
  if (deps.forkDepth >= 3) {
    throw new Error('Max fork depth exceeded (prevents infinite recursion)');
  }
  
  const childAgent = new Agent({
    ...deps,
    forkDepth: deps.forkDepth + 1,
    
    // Inherit full message history (byte-level copy)
    context: {
      messages: structuredClone(deps.context.messages),
      tokenBudget: deps.context.tokenBudget * 0.8 // 80% of parent budget
    },
    
    // Inherit selective memory
    memory: forkMemory(deps.memory, ['project_context', 'code_patterns']),
    
    // New task-specific system prompt
    systemPrompt: `${deps.systemPrompt}\n\nForked for: ${purpose}`,
    
    // Inherit parent's tool access
    tools: deps.tools,
    permissions: deps.permissions
  });
  
  return childAgent;
}

// Four built-in agents
const BUILTIN_AGENTS = {
  coordinator: {
    role: 'Orchestrate multiple workers, never execute tools directly',
    tools: ['fork_agent', 'send_message_to_agent', 'collect_results'],
    constraints: ['no_file_access', 'no_shell_access']
  },
  worker: {
    role: 'Execute specific tasks assigned by coordinator',
    tools: ['all'], // Full tool access
    constraints: ['report_progress_every_5min']
  },
  researcher: {
    role: 'Gather information, no side effects',
    tools: ['read_file', 'search', 'browse', 'read_memory'],
    constraints: ['read_only_mode']
  },
  reviewer: {
    role: 'Analyze code, suggest improvements',
    tools: ['read_file', 'run_linter', 'run_tests'],
    constraints: ['no_modifications']
  }
};
```

### 8. MCP Integration (External Protocol Bridge)

Model Context Protocol for external tool/resource integration:

```typescript
// Eight transport protocols
type MCPTransport = 
  | 'stdio'      // Standard I/O pipes
  | 'http'       // REST API
  | 'websocket'  // WebSocket
  | 'grpc'       // gRPC
  | 'ipc'        // Inter-process communication
  | 'tcp'        // Raw TCP
  | 'udp'        // UDP datagrams
  | 'sse';       // Server-sent events

// Five-state connection management
type MCPConnectionState = 
  | 'disconnected'
  | 'connecting'
  | 'connected'
  | 'error'
  | 'reconnecting';

class MCPConnection {
  state: MCPConnectionState = 'disconnected';
  retryCount = 0;
  maxRetries = 3;
  
  async connect(config: MCPServerConfig) {
    this.state = 'connecting';
    
    try {
      const transport = createTransport(config.transport);
      await transport.initialize();
      
      // Handshake: Exchange capabilities
      const serverInfo = await transport.request({
        method: 'initialize',
        params: {
          clientInfo: { name: 'claude-code', version: '1.0' },
          capabilities: ['tools', 'resources', 'prompts']
        }
      });
      
      this.state = 'connected';
      return serverInfo;
    } catch (error) {
      this.state = 'error';
      
      if (this.retryCount < this.maxRetries) {
        this.retryCount++;
        this.state = 'reconnecting';
        await sleep(1000 * this.retryCount); // Exponential backoff
        return this.connect(config);
      }
      
      throw error;
    }
  }
}

// Three-segment tool naming (namespace isolation)
// Format: mcp://{server_name}/{tool_name}
const mcpTool = {
  name: 'mcp://github/create_issue',
  handler: async (input, deps) => {
    const connection = deps.mcpConnections.get('github');
    return await connection.call('create_issue', input);
  }
};

// Bridge pattern (bidirectional communication)
class MCPBridge {
  // Forward: Agent → MCP Server
  async callTool(server: string, tool: string, input: unknown) {
    const conn = this.connections.get(server);
    return await conn.request({
      method: 'tools/call',
      params: { name: tool, arguments: input }
    });
  }
  
  // Reverse: MCP Server → Agent (notifications)
  async handleNotification(notification: MCPNotification) {
    if (notification.method === 'resources/updated') {
      // Invalidate cached resources
      await this.invalidateCache(notification.params.uri);
    }
  }
}
```

## Practical Patterns

### Building a Custom Tool

```typescript
import { buildTool } from './tool-factory';
import { z } from 'zod';

// Example: Jira integration tool
const createJiraIssueTool = buildTool({
  schema: z.object({
    project: z.string(),
    summary: z.string(),
    description: z.string(),
    issueType: z.enum(['Bug', 'Task', 'Story']).default('Task'),
    priority: z.enum(['Low', 'Medium', 'High']).default('Medium')
  }),
  
  permission: 'write', // Requires write access
  
  properties: {
    readOnly: false,
    destructive: false, // Can be undone (delete issue)
    concurrencySafe: true // Multiple issues can be created in parallel
  },
  
  handler: async (input, deps) => {
    // Use environment variables for secrets
    const jiraClient = new JiraClient({
      host: process.env.JIRA_HOST,
      auth: {
        username: process.env.JIRA_USERNAME,
        apiToken: process.env.JIRA_API_TOKEN
      }
    });
    
    try {
      const issue = await jiraClient.issues.create({
        fields: {
          project: { key: input.project },
          summary: input.summary,
          description: input.description,
          issuetype: { name: input.issueType },
          priority: { name: input.priority }
        }
      });
      
      return {
        success: true,
        issueKey: issue.key,
        url: `${process.env.JIRA_HOST}/browse/${issue.key}`
      };
    } catch (error) {
      throw new ToolError('JiraCreateFailed', {
        message: error.message,
        statusCode: error.statusCode
      });
    }
  }
});
```

### Implementing a Permission Strategy

```typescript
// Custom permission strategy with role-based access
class RBACPermissionStrategy implements PermissionStrategy {
  private userRole: 'developer' | 'senior' | 'admin';
  
  constructor(role: string) {
    this.userRole = role as any;
  }
  
  async authorize(toolCall: ToolCall): Promise<AuthResult> {
    const permissions = {
      developer: {
        allowed: ['read_file', 'search', 'run_tests'],
        denied: ['execute_bash', 'write_file', 'git_push']
      },
      senior: {
        allowed: ['read_file', 'write_file', 'execute_bash', 'git_commit'],
        denied: ['git_push', 'delete_file', 'modify_settings']
      },
      admin: {
        allowed: ['*'], // All tools
        denied: []
      }
    };
    
    const config = permissions[this.userRole];
    
    if (config.denied.includes(toolCall.tool.name)) {
      return { allowed: false, reason: 'Role restriction' };
    }
    
    if (config.allowed.includes('*') || config.allowed.includes(toolCall.tool.name)) {
      // Additional check for destructive operations
      if (toolCall.tool.properties.destructive) {
        return {
          allowed: true,
          requireConfirmation: this.userRole !== 'admin'
        };
      }
      
      return { allowed: true };
    }
    
    return { allowed: false, reason: 'Tool not in allowed list' };
  }
}
```

### Context Compression Strategy

```typescript
// Aggressive compression for long conversations
class AggressiveCompressionStrategy {
  async compress(messages: Message[], targetTokens: number): Promise<Message[]> {
    let result = messages;
    const steps = [
      { name: 'Remove images', fn: this.removeAllImages },
      { name: 'Summarize tool outputs', fn: this.summarizeToolOutputs },
      { name: 'Collapse redundant exchanges', fn: this.collapseRedundant },
      { name: 'Semantic compression', fn: this.semanticCompress }
    ];
    
    for (const step of steps) {
      const currentTokens = estimateTokens(result);
      
      if (currentTokens <= targetTokens) {
        break; // Target reached
      }
      
      console.log(`Compression step: ${step.name} (${currentTokens} → target ${targetTokens})`);
      result = await step.fn(result, targetTokens);
    }
    
    return result;
  }
  
  private async semanticCompress(
    messages: Message[],
    targetTokens: number
  ): Promise<Message[]> {
    // Use LLM to summarize conversation history
    const summary = await llmClient.chat({
      messages: [
        {
          role: 'user',
          content: `Summarize this conversation history in ${targetTokens * 0.3} tokens:\n\n${formatMessages(messages)}`
        }
      ]
    });
    
    return [
      {
        role: 'assistant',
        content: `[Conversation summary]: ${summary.content}`
      },
      ...messages.slice(-5) // Keep last 5 messages intact
    ];
  }
}
```

## Configuration

### Setting Up Agent Harness

```typescript
// .claude/config.json
{
  "version": "1.0",
  "agent": {
    "model": "claude-3-5-sonnet-20241022",
    "contextWindow": 200000,
    "permissionMode": "normal", // safe | normal | relaxed | auto
    "features": {
      "toolCalling": true,
      "memorySystem": true,
      "subAgents": true,
      "mcpIntegration": true
    }
  },
  "tools": {
    "enabled": [
      "read_file",
      "write_file",
      "execute_bash",
      "ripgrep_search",
      "git_*" // Wildcard pattern
    ],
    "disabled": [
      "delete_file" // Explicitly disabled
    ],
    "concurrency": {
      "maxParallel": 5,
      "partitionStrategy": "read-write-split"
    }
  },
  "context": {
    "compression": {
      "strategy": "progressive", // progressive | aggressive | minimal
      "levels": ["snip", "micro-compact", "collapse", "auto-compact"],
      "targetUtilization": 0.9 // 90% of effective window
    }
  },
  "memory": {
    "storageBackend": "filesystem", // filesystem | sqlite | postgres
    "indexFile": ".claude/MEMORY.md",
    "pruneStrategy": {
      "confidenceThreshold": 0.3,
      "maxAge": "30d"
    }
  },
  "hooks": {
    "enabled": true,
    "directory": ".claude/hooks",
    "defaultTimeout": 5000
  },
  "mcp": {
    "servers": {
      "github": {
        "transport": "http",
        "url": "https://localhost:3000/mcp",
        "auth": {
          "type": "bearer",
          "token": "${GITHUB_TOKEN}" // Environment variable reference
        }
      },
      "database": {
        "transport": "stdio",
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-postgres"],
        "env": {
          "POSTGRES_URL": "${DATABASE_URL}"
        }
      }
    }
  }
}
```

### Feature Flags (89 total)

```typescript
// Runtime feature flags (13 categories)
const FEATURE_FLAGS = {
  // Core features
  'agent.tool_calling': true,
  'agent.sub_agents': true,
  'agent.memory_system': true,
  
  // Safety features
  'safety.permission_pipeline': true,
  'safety.bash_validation': true,
  'safety.path_restrictions': true,
  
  // Performance
  'perf.lazy_tool_loading': true,
