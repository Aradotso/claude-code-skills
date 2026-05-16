---
name: claude-code-agent-harness-architecture
description: Deep architectural knowledge of Agent Harness design patterns from the Claude Code book - build, analyze, and optimize AI agent systems
triggers:
  - how do I build an agent harness
  - explain the agent conversation loop
  - how does tool permission management work
  - show me agent context compression strategies
  - how to implement agent memory systems
  - what are agent hook lifecycle patterns
  - design a multi-agent coordinator system
  - explain MCP protocol integration for agents
---

# Claude Code Agent Harness Architecture

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

Expert knowledge for building production-grade AI Agent systems based on the "御舆: 解码 Agent Harness" (Decoding Agent Harness) book - a 420,000-word deep architectural analysis of Claude Code's Agent Harness design.

## What This Skill Covers

This skill provides architectural patterns, design decisions, and implementation strategies for building AI Agent systems (harnesses). It covers:

- **Conversation Loop Architecture** - Async generator-based main loops
- **Tool Systems** - Type-safe tool protocols and concurrent execution
- **Permission Pipelines** - Four-stage authorization systems
- **Context Management** - Token budget optimization and compression
- **Memory Systems** - Long-term agent memory patterns
- **Hook Systems** - Lifecycle extension points
- **Sub-Agent Orchestration** - Fork patterns and coordinator modes
- **MCP Integration** - Model Context Protocol bridging
- **Performance Optimization** - Streaming, lazy loading, caching

## Core Concepts

### Agent Harness Architecture

An Agent Harness is the runtime framework that makes an AI agent operate. Key components:

```
┌─────────────────────────────────────────┐
│         Agent Harness Core              │
├─────────────────────────────────────────┤
│  ┌──────────────────────────────────┐  │
│  │   Conversation Loop (Main)       │  │
│  │   - Async Generator Pattern      │  │
│  │   - Event Yield System           │  │
│  └──────────────────────────────────┘  │
│  ┌──────────────────────────────────┐  │
│  │   Tool System                    │  │
│  │   - Tool<I,O,P> Protocol         │  │
│  │   - Concurrent Execution         │  │
│  └──────────────────────────────────┘  │
│  ┌──────────────────────────────────┐  │
│  │   Permission Pipeline            │  │
│  │   - Pre-Request → Classification │  │
│  │   - Approval → Post-Validation   │  │
│  └──────────────────────────────────┘  │
│  ┌──────────────────────────────────┐  │
│  │   Context Manager                │  │
│  │   - Token Budget Tracking        │  │
│  │   - Progressive Compression      │  │
│  └──────────────────────────────────┘  │
└─────────────────────────────────────────┘
```

### Five Design Principles

1. **Fault-Safe First** - Every component has failure modes and recovery paths
2. **Progressive Enhancement** - Core features work without optional dependencies
3. **Explicit Over Implicit** - Configuration visible, no magic defaults
4. **Composable Primitives** - Small, single-responsibility modules
5. **Observable by Default** - Built-in logging, tracing, metrics

## Building a Basic Agent Harness

### 1. Conversation Loop Implementation

The heart of any agent - an async generator that yields events:

```typescript
import type { QueryDeps } from './types';

async function* conversationLoop(
  deps: QueryDeps
): AsyncGenerator<ConversationEvent, void, void> {
  while (true) {
    // 1. Get user input
    yield { type: 'waiting_for_input' };
    const userMessage = await deps.input.next();
    
    if (!userMessage) {
      yield { type: 'termination', reason: 'user_exit' };
      break;
    }

    // 2. Add to context
    deps.context.addMessage({ role: 'user', content: userMessage });

    // 3. Check token budget before API call
    if (deps.context.estimatedTokens > deps.tokenBudget.effective) {
      await compressContext(deps);
    }

    // 4. Call LLM
    yield { type: 'llm_request_start' };
    const response = await deps.llm.chat({
      messages: deps.context.messages,
      tools: deps.tools.getAvailable(),
    });

    // 5. Handle response
    if (response.type === 'tool_calls') {
      yield { type: 'tool_execution_start', calls: response.toolCalls };
      
      for (const call of response.toolCalls) {
        // Permission check (4-stage pipeline)
        const permitted = await checkPermission(call, deps);
        if (!permitted) {
          yield { type: 'permission_denied', toolName: call.name };
          continue;
        }

        // Execute tool
        const result = await deps.tools.execute(call);
        yield { type: 'tool_result', call, result };
        deps.context.addToolResult(call, result);
      }
      
      continue; // Loop back for next turn
    }

    if (response.type === 'message') {
      yield { type: 'assistant_message', content: response.content };
      deps.context.addMessage({ role: 'assistant', content: response.content });
      
      // Check for termination signals
      if (shouldTerminate(response, deps)) {
        yield { type: 'termination', reason: 'task_complete' };
        break;
      }
    }

    // 6. Check iteration budget
    if (deps.context.turnCount >= deps.maxTurns) {
      yield { type: 'termination', reason: 'max_turns_exceeded' };
      break;
    }
  }
}
```

**Five Event Types:**
- `waiting_for_input` - Agent ready for user input
- `llm_request_start` - Before LLM API call
- `tool_execution_start` - Before tool execution
- `assistant_message` - LLM text response
- `termination` - Conversation end (10 possible reasons)

### 2. Tool System with Type Safety

Type-safe tool definition using Zod schemas:

```typescript
import { z } from 'zod';

// Tool protocol: Tool<Input, Output, Params>
interface Tool<I, O, P = void> {
  name: string;
  description: string;
  inputSchema: z.ZodType<I>;
  execute: (input: I, params: P) => Promise<O>;
  metadata: ToolMetadata;
}

interface ToolMetadata {
  readOnly: boolean;          // Side-effect free?
  destructive: boolean;        // Irreversible changes?
  concurrencySafe: boolean;    // Parallel execution safe?
  requiresApproval: boolean;   // Always ask user?
  estimatedLatency: number;    // Ms estimate
}

// Build tool with fault-safe factory
function buildTool<I, O, P = void>(
  definition: ToolDefinition<I, O, P>
): Tool<I, O, P> {
  return {
    ...definition,
    execute: async (input, params) => {
      try {
        // Validate input
        const validated = definition.inputSchema.parse(input);
        
        // Execute with timeout
        const result = await Promise.race([
          definition.execute(validated, params),
          timeoutPromise(definition.metadata.estimatedLatency * 2),
        ]);
        
        return result;
      } catch (error) {
        // Log and wrap errors
        console.error(`Tool ${definition.name} failed:`, error);
        throw new ToolExecutionError(definition.name, error);
      }
    },
  };
}

// Example: File read tool
const readFileTool = buildTool({
  name: 'read_file',
  description: 'Read file contents',
  inputSchema: z.object({
    path: z.string().min(1),
    encoding: z.enum(['utf8', 'base64']).default('utf8'),
  }),
  execute: async ({ path, encoding }) => {
    const fs = await import('fs/promises');
    const content = await fs.readFile(path, encoding);
    return { path, content, size: content.length };
  },
  metadata: {
    readOnly: true,
    destructive: false,
    concurrencySafe: true,
    requiresApproval: false,
    estimatedLatency: 50,
  },
});
```

**Concurrent Tool Execution (Partition-Greedy Algorithm):**

```typescript
async function executeToolsConcurrently(
  calls: ToolCall[],
  tools: Map<string, Tool<any, any>>
): Promise<ToolResult[]> {
  // Partition: concurrent-safe vs sequential
  const safeCalls = calls.filter(c => 
    tools.get(c.name)?.metadata.concurrencySafe
  );
  const unsafeCalls = calls.filter(c => 
    !tools.get(c.name)?.metadata.concurrencySafe
  );

  // Execute safe tools in parallel
  const safeResults = await Promise.all(
    safeCalls.map(call => tools.get(call.name)!.execute(call.input))
  );

  // Execute unsafe tools sequentially
  const unsafeResults = [];
  for (const call of unsafeCalls) {
    const result = await tools.get(call.name)!.execute(call.input);
    unsafeResults.push(result);
  }

  return [...safeResults, ...unsafeResults];
}
```

### 3. Four-Stage Permission Pipeline

```typescript
type PermissionMode = 'auto' | 'confirm_once' | 'confirm_always' | 'deny' | 'review';

interface PermissionPipeline {
  // Stage 1: Pre-request filtering
  preRequest(call: ToolCall): Promise<PreRequestDecision>;
  
  // Stage 2: Classification (2-second timeout)
  classify(call: ToolCall): Promise<RiskLevel>;
  
  // Stage 3: User approval (if needed)
  getApproval(call: ToolCall, risk: RiskLevel): Promise<boolean>;
  
  // Stage 4: Post-validation
  postValidate(call: ToolCall, result: any): Promise<boolean>;
}

async function checkPermission(
  call: ToolCall, 
  deps: QueryDeps
): Promise<boolean> {
  const pipeline = deps.permissionPipeline;

  // Stage 1: Pre-request rules
  const preCheck = await pipeline.preRequest(call);
  if (preCheck === 'deny') {
    return false;
  }
  if (preCheck === 'allow') {
    // Skip classification for explicitly allowed patterns
    return true;
  }

  // Stage 2: Speculative classification (with timeout)
  const risk = await Promise.race([
    pipeline.classify(call),
    new Promise<RiskLevel>(resolve => 
      setTimeout(() => resolve('medium'), 2000)
    ),
  ]);

  // Stage 3: Approval based on mode and risk
  const mode = deps.permissionMode;
  let needsApproval = false;

  if (mode === 'confirm_always') {
    needsApproval = true;
  } else if (mode === 'confirm_once' && !deps.approvedTools.has(call.name)) {
    needsApproval = true;
  } else if (risk === 'high' || risk === 'critical') {
    needsApproval = true;
  }

  if (needsApproval) {
    const approved = await pipeline.getApproval(call, risk);
    if (!approved) return false;
    
    if (mode === 'confirm_once') {
      deps.approvedTools.add(call.name);
    }
  }

  return true; // Post-validation happens after execution
}
```

**Rule Matching (Bash-style patterns):**

```typescript
interface PermissionRule {
  pattern: string;  // e.g., "bash:rm *", "read_file:/etc/**"
  action: 'allow' | 'deny';
  priority: number;
}

function matchRule(call: ToolCall, rules: PermissionRule[]): 'allow' | 'deny' | 'unmatched' {
  const sortedRules = rules.sort((a, b) => b.priority - a.priority);
  
  for (const rule of sortedRules) {
    const [toolPattern, argsPattern] = rule.pattern.split(':');
    
    // Match tool name
    if (!minimatch(call.name, toolPattern)) continue;
    
    // Match arguments (if pattern provided)
    if (argsPattern) {
      const argsString = JSON.stringify(call.input);
      if (!minimatch(argsString, argsPattern)) continue;
    }
    
    return rule.action;
  }
  
  return 'unmatched';
}
```

### 4. Context Management and Compression

**Token Budget Tracking:**

```typescript
interface TokenBudget {
  total: number;        // Total limit (e.g., 200K)
  reserved: number;     // Reserved for system (e.g., 10K)
  effective: number;    // total - reserved
  current: number;      // Currently used
  remaining: number;    // effective - current
}

class ContextManager {
  private messages: Message[] = [];
  private budget: TokenBudget;

  estimateTokens(text: string): number {
    // Rough estimate: 1 token ≈ 4 chars for English
    return Math.ceil(text.length / 4);
  }

  get estimatedTokens(): number {
    return this.messages.reduce((sum, msg) => 
      sum + this.estimateTokens(JSON.stringify(msg)), 0
    );
  }

  async ensureBudget(): Promise<void> {
    if (this.estimatedTokens <= this.budget.effective) {
      return; // Within budget
    }

    // Progressive compression: 4 levels
    await this.compress();
  }

  private async compress(): Promise<void> {
    // Level 1: Snip - Remove duplicate/verbose content
    this.snipRedundantContent();
    if (this.estimatedTokens <= this.budget.effective) return;

    // Level 2: MicroCompact - Summarize old turns
    await this.microCompact();
    if (this.estimatedTokens <= this.budget.effective) return;

    // Level 3: Collapse - Aggressive summarization
    await this.collapse();
    if (this.estimatedTokens <= this.budget.effective) return;

    // Level 4: AutoCompact - Emergency truncation
    this.autoCompact();
  }

  private snipRedundantContent(): void {
    // Remove duplicate tool results, long code blocks
    const seen = new Set<string>();
    this.messages = this.messages.filter(msg => {
      if (msg.role === 'tool_result') {
        const hash = hashObject(msg.content);
        if (seen.has(hash)) return false;
        seen.add(hash);
      }
      return true;
    });
  }

  private async microCompact(): Promise<void> {
    // Summarize messages older than N turns
    const keepRecent = 10;
    const toCompress = this.messages.slice(0, -keepRecent);
    
    if (toCompress.length === 0) return;

    const summary = await this.summarizeMessages(toCompress);
    this.messages = [
      { role: 'system', content: `Previous context: ${summary}` },
      ...this.messages.slice(-keepRecent),
    ];
  }

  private async collapse(): Promise<void> {
    // Aggressive: Keep only task-critical messages
    const critical = this.messages.filter(msg =>
      msg.role === 'system' || 
      msg.metadata?.taskCritical === true
    );
    
    const summary = await this.summarizeMessages(
      this.messages.filter(m => !critical.includes(m))
    );
    
    this.messages = [
      { role: 'system', content: summary },
      ...critical.slice(-5), // Last 5 critical messages
    ];
  }

  private autoCompact(): void {
    // Emergency: Hard truncate to fit budget
    const targetTokens = this.budget.effective * 0.8;
    while (this.estimatedTokens > targetTokens && this.messages.length > 1) {
      this.messages.shift(); // Remove oldest
    }
  }
}
```

**Circuit Breaker Pattern (prevents compression loops):**

```typescript
class CompressionCircuitBreaker {
  private failures = 0;
  private lastAttempt = 0;
  private state: 'closed' | 'open' | 'half_open' = 'closed';

  async compress(fn: () => Promise<void>): Promise<void> {
    if (this.state === 'open') {
      const now = Date.now();
      if (now - this.lastAttempt < 60000) { // 1 min cooldown
        throw new Error('Circuit breaker OPEN: too many compression failures');
      }
      this.state = 'half_open';
    }

    try {
      await fn();
      this.failures = 0;
      this.state = 'closed';
    } catch (error) {
      this.failures++;
      this.lastAttempt = Date.now();
      
      if (this.failures >= 3) {
        this.state = 'open';
      }
      throw error;
    }
  }
}
```

### 5. Memory System (Long-term Storage)

**Four Memory Types:**

```typescript
interface MemorySystem {
  // 1. User Preferences (survives across sessions)
  preferences: Map<string, any>;
  
  // 2. Learned Patterns (ML-extracted insights)
  patterns: Pattern[];
  
  // 3. Task History (completed tasks for reference)
  taskHistory: TaskRecord[];
  
  // 4. Custom Facts (user-provided knowledge)
  facts: Fact[];
}

interface Fact {
  id: string;
  content: string;
  source: 'user' | 'agent' | 'tool';
  timestamp: number;
  tags: string[];
  confidence: number; // 0-1
}

class MemoryManager {
  private memoryFile = '.agent/MEMORY.md';

  async load(): Promise<MemorySystem> {
    const content = await fs.readFile(this.memoryFile, 'utf8');
    return this.parseMemoryMarkdown(content);
  }

  async save(memory: MemorySystem): Promise<void> {
    const markdown = this.formatMemoryMarkdown(memory);
    await fs.writeFile(this.memoryFile, markdown);
  }

  // Only save information that cannot be re-derived
  shouldSave(fact: Fact): boolean {
    // Don't save: file contents, API responses, ephemeral state
    // Do save: user preferences, learned heuristics, task outcomes
    
    if (fact.source === 'tool') {
      return false; // Tools can be re-executed
    }
    
    if (fact.tags.includes('ephemeral')) {
      return false;
    }
    
    if (fact.confidence < 0.7) {
      return false; // Low confidence facts
    }
    
    return true;
  }

  private formatMemoryMarkdown(memory: MemorySystem): string {
    return `# Agent Memory

## Preferences
${Array.from(memory.preferences.entries())
  .map(([k, v]) => `- ${k}: ${JSON.stringify(v)}`)
  .join('\n')}

## Learned Patterns
${memory.patterns.map(p => `- ${p.description} (confidence: ${p.confidence})`).join('\n')}

## Task History (Last 20)
${memory.taskHistory.slice(-20).map(t => `- [${t.timestamp}] ${t.summary}`).join('\n')}

## Facts
${memory.facts.map(f => `- ${f.content} (${f.source}, ${f.timestamp})`).join('\n')}
`;
  }
}
```

### 6. Hook System (Lifecycle Extensions)

**26 Lifecycle Events:**

```typescript
type HookEvent =
  // Conversation hooks
  | 'conversation:start'
  | 'conversation:turn:start'
  | 'conversation:turn:end'
  | 'conversation:end'
  // Tool hooks
  | 'tool:before_execution'
  | 'tool:after_execution'
  | 'tool:error'
  // Context hooks
  | 'context:before_compression'
  | 'context:after_compression'
  | 'context:budget_exceeded'
  // Permission hooks
  | 'permission:requested'
  | 'permission:granted'
  | 'permission:denied'
  // Memory hooks
  | 'memory:loaded'
  | 'memory:saved'
  | 'memory:fact_added'
  // Agent hooks
  | 'agent:forked'
  | 'agent:terminated'
  // ... (19 more)

interface Hook {
  event: HookEvent;
  priority: number; // 0-100, higher = earlier
  handler: (payload: any) => Promise<HookResponse>;
  metadata: {
    id: string;
    source: string; // skill, plugin, user config
    enabled: boolean;
  };
}

interface HookResponse {
  continue: boolean;        // false = stop propagation
  modifiedPayload?: any;    // Transform payload for next hook
  sideEffects?: SideEffect[]; // Actions to perform
}

class HookManager {
  private hooks = new Map<HookEvent, Hook[]>();

  register(hook: Hook): void {
    const existing = this.hooks.get(hook.event) || [];
    existing.push(hook);
    existing.sort((a, b) => b.priority - a.priority);
    this.hooks.set(hook.event, existing);
  }

  async trigger(event: HookEvent, payload: any): Promise<any> {
    const hooks = this.hooks.get(event) || [];
    let currentPayload = payload;

    for (const hook of hooks) {
      if (!hook.metadata.enabled) continue;

      try {
        const response = await hook.handler(currentPayload);
        
        if (!response.continue) {
          break; // Stop propagation
        }

        if (response.modifiedPayload) {
          currentPayload = response.modifiedPayload;
        }

        if (response.sideEffects) {
          await this.executeSideEffects(response.sideEffects);
        }
      } catch (error) {
        console.error(`Hook ${hook.metadata.id} failed:`, error);
        // Continue with other hooks
      }
    }

    return currentPayload;
  }

  private async executeSideEffects(effects: SideEffect[]): Promise<void> {
    await Promise.all(effects.map(e => e.execute()));
  }
}
```

**Example: Logging Hook**

```typescript
hookManager.register({
  event: 'tool:before_execution',
  priority: 90,
  handler: async (payload: { call: ToolCall; timestamp: number }) => {
    console.log(`[${new Date(payload.timestamp).toISOString()}] Executing tool: ${payload.call.name}`);
    console.log('Input:', JSON.stringify(payload.call.input, null, 2));
    
    return {
      continue: true,
      modifiedPayload: {
        ...payload,
        metadata: { logged: true },
      },
    };
  },
  metadata: {
    id: 'tool-logger',
    source: 'core',
    enabled: true,
  },
});
```

### 7. Sub-Agent and Fork Pattern

**Fork: Byte-level Context Inheritance**

```typescript
interface ForkConfig {
  inheritContext: boolean;      // Copy parent messages?
  inheritMemory: boolean;        // Copy parent memory?
  inheritTools: boolean;         // Copy parent tool access?
  inheritPermissions: boolean;   // Copy parent permission state?
  budget: {
    tokens: number;              // Token budget for child
    maxTurns: number;            // Turn limit
    timeout: number;             // Wall-clock timeout (ms)
  };
}

class AgentOrchestrator {
  async fork(
    parentAgent: Agent,
    config: ForkConfig
  ): Promise<Agent> {
    // Prevent recursive fork bombs
    if (parentAgent.forkDepth >= 5) {
      throw new Error('Maximum fork depth exceeded');
    }

    const childDeps: QueryDeps = {
      // Inherit or create new instances
      context: config.inheritContext 
        ? parentAgent.deps.context.clone() 
        : new ContextManager(),
      
      memory: config.inheritMemory
        ? parentAgent.deps.memory.clone()
        : new MemoryManager(),
      
      tools: config.inheritTools
        ? parentAgent.deps.tools.clone()
        : new ToolRegistry(),
      
      permissionPipeline: config.inheritPermissions
        ? parentAgent.deps.permissionPipeline.clone()
        : new PermissionPipeline(),
      
      // Always create new budget
      tokenBudget: {
        total: config.budget.tokens,
        reserved: config.budget.tokens * 0.1,
        effective: config.budget.tokens * 0.9,
        current: 0,
        remaining: config.budget.tokens * 0.9,
      },
      
      maxTurns: config.budget.maxTurns,
      llm: parentAgent.deps.llm, // Share LLM client
      input: new InputStream(),  // New input stream
      output: new OutputStream(), // New output stream
    };

    const childAgent = new Agent({
      deps: childDeps,
      forkDepth: parentAgent.forkDepth + 1,
      parentId: parentAgent.id,
    });

    return childAgent;
  }
}
```

**Coordinator Pattern (Orchestration without Execution):**

```typescript
interface CoordinatorAgent extends Agent {
  // Coordinator CANNOT execute tools directly
  canExecuteTools: false;
  
  // Coordinator CAN:
  // 1. Create sub-agents
  // 2. Route tasks
  // 3. Aggregate results
  // 4. Make decisions
}

async function coordinatorLoop(deps: QueryDeps): Promise<void> {
  const orchestrator = new AgentOrchestrator();
  
  while (true) {
    const userTask = await deps.input.next();
    
    // Coordinator analyzes and breaks down task
    const plan = await planTask(userTask, deps.llm);
    
    // Create specialized sub-agents
    const workers = await Promise.all(
      plan.subtasks.map(subtask =>
        orchestrator.fork(deps as any, {
          inheritContext: false,
          inheritMemory: true,  // Share knowledge
          inheritTools: true,   // Workers can use tools
          inheritPermissions: true,
          budget: {
            tokens: 50000,
            maxTurns: 20,
            timeout: 300000, // 5 min
          },
        })
      )
    );

    // Execute subtasks in parallel
    const results = await Promise.all(
      workers.map((worker, i) => 
        worker.execute(plan.subtasks[i].prompt)
      )
    );

    // Aggregate results (coordinator's job)
    const finalResult = await aggregateResults(results, deps.llm);
    
    deps.output.write(finalResult);
  }
}
```

### 8. MCP Integration (Model Context Protocol)

**Eight Transport Protocols:**

```typescript
type MCPTransport = 
  | 'stdio'           // Standard input/output
  | 'http'            // HTTP/REST
  | 'websocket'       // WebSocket
  | 'grpc'            // gRPC
  | 'ipc'             // Inter-process communication
  | 'tcp'             // Raw TCP
  | 'unix_socket'     // Unix domain socket
  | 'sse';            // Server-Sent Events

interface MCPServer {
  id: string;
  transport: MCPTransport;
  config: MCPServerConfig;
  state: 'disconnected' | 'connecting' | 'connected' | 'error' | 'reconnecting';
  tools: MCPTool[];
}

interface MCPServerConfig {
  command?: string;       // For stdio: e.g., "npx -y @modelcontextprotocol/server-filesystem"
  args?: string[];        // Command arguments
  url?: string;           // For HTTP/WebSocket
  env?: Record<string, string>; // Environment variables
  timeout?: number;       // Connection timeout
  retryAttempts?: number; // Reconnection attempts
}

class MCPBridge {
  private servers = new Map<string, MCPServer>();
  private connections = new Map<string, MCPConnection>();

  async connectServer(config: MCPServerConfig): Promise<MCPServer> {
    const server: MCPServer = {
      id: generateId(),
      transport: this.detectTransport(config),
      config,
      state: 'connecting',
      tools: [],
    };

    try {
      const connection = await this.createConnection(server);
      this.connections.set(server.id, connection);
      
      // Discover available tools
      server.tools = await connection.listTools();
      server.state = 'connected';
      
      this.servers.set(server.id, server);
      return server;
    } catch (error) {
      server.state = 'error';
      throw new MCPConnectionError(server.id, error);
    }
  }

  private async createConnection(server: MCPServer): Promise<MCPConnection> {
    switch (server.transport) {
      case 'stdio':
        return new StdioMCPConnection(server.config);
      case 'http':
        return new HttpMCPConnection(server.config);
      case 'websocket':
        return new WebSocketMCPConnection(server.config);
      // ... other transports
      default:
        throw new Error(`Unsupported transport: ${server.transport}`);
    }
  }

  // Bridge MCP tools to native tool system
  async bridgeTools(serverId: string, toolRegistry: ToolRegistry): Promise<void> {
    const server = this.servers.get(serverId);
    if (!server) throw new Error(`Server ${serverId} not found`);

    for (const mcpTool of server.tools) {
      // Three-part naming: <server>:<transport>:<tool>
      const toolName = `${server.id}:${server.transport}:${mcpTool.name}`;
      
      const bridgedTool = buildTool({
        name: toolName,
        description: mcpTool.description,
        inputSchema: mcpTool.inputSchema,
        execute: async (input) => {
          const connection = this.connections.get(serverId);
          if (!connection) throw new Error('Server disconnected');
          
          return await connection
