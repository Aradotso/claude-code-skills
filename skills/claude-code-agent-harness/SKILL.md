---
name: claude-code-agent-harness
description: Deep architectural knowledge of AI Agent Harness design patterns, implementation strategies, and Claude Code internals for building production-grade AI agents
triggers:
  - "how do I build an agent harness"
  - "explain the agent conversation loop"
  - "implement tool system for my agent"
  - "set up agent permission pipeline"
  - "design agent context management"
  - "create MCP integration for agent"
  - "build sub-agent fork mechanism"
  - "implement agent memory system"
---

# Claude Code Agent Harness Architecture

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## Overview

This project is a comprehensive 420,000-word architectural analysis of AI Agent Harness design, using Claude Code as the reference implementation. It provides production-ready patterns for building agent systems with conversation loops, tool execution, permission management, context compression, memory systems, and multi-agent orchestration.

**Key Capabilities:**
- Agent conversation loop architecture (async generator pattern)
- Tool system design with 50+ tool implementations
- Four-stage permission pipeline
- Context management and compression strategies
- Memory systems and fork mechanisms
- MCP (Model Context Protocol) integration
- Sub-agent orchestration patterns
- Streaming architecture and performance optimization

## Installation

```bash
# Clone the reference architecture
git clone https://github.com/lintsinghua/claude-code-book.git
cd claude-code-book

# Read online (recommended for interactive diagrams)
# Visit: https://lintsinghua.github.io
```

## Core Architecture Patterns

### 1. Conversation Loop (Agent Heartbeat)

The conversation loop is an async generator that drives agent execution:

```typescript
// Core conversation loop pattern
async function* conversationLoop(
  deps: QueryDeps
): AsyncGenerator<YieldEvent, void, void> {
  while (true) {
    // 1. Generate LLM response
    const response = await deps.llm.generate({
      messages: deps.context.messages,
      tools: deps.tools.available,
    });

    // 2. Yield streaming events
    for (const chunk of response.stream) {
      yield { type: 'text_delta', delta: chunk };
    }

    // 3. Handle tool calls
    if (response.toolCalls) {
      for (const call of response.toolCalls) {
        const result = await deps.tools.execute(call);
        yield { type: 'tool_result', result };
        deps.context.addMessage({ role: 'tool', content: result });
      }
      continue; // Loop back for next turn
    }

    // 4. Check termination conditions
    if (shouldTerminate(response, deps)) {
      yield { type: 'done', reason: getTerminationReason(response) };
      break;
    }
  }
}

// Five yield event types
type YieldEvent =
  | { type: 'text_delta'; delta: string }
  | { type: 'tool_call'; call: ToolCall }
  | { type: 'tool_result'; result: ToolResult }
  | { type: 'thinking'; content: string }
  | { type: 'done'; reason: TerminationReason };

// Ten termination reasons
type TerminationReason =
  | 'max_turns'
  | 'user_interrupt'
  | 'explicit_stop'
  | 'error'
  | 'context_overflow'
  | 'permission_denied'
  | 'natural_completion'
  | 'timeout'
  | 'sub_agent_complete'
  | 'plan_complete';
```

### 2. Tool System (Agent's Hands)

Tools follow a five-element protocol:

```typescript
// Tool definition interface
interface Tool<TInput, TOutput, TParams> {
  name: string;
  description: string;
  inputSchema: z.ZodSchema<TInput>;
  execute: (input: TInput, params: TParams) => Promise<TOutput>;
  metadata: ToolMetadata;
}

interface ToolMetadata {
  readOnly: boolean;
  destructive: boolean;
  concurrencySafe: boolean;
  requiresConfirmation: boolean;
  category: ToolCategory;
}

// Fault-safe tool builder
function buildTool<TInput, TOutput>(
  config: ToolConfig<TInput, TOutput>
): Tool<TInput, TOutput> {
  return {
    name: config.name,
    description: config.description,
    inputSchema: config.schema,
    execute: async (input, params) => {
      try {
        // Validate input
        const validated = config.schema.parse(input);
        
        // Check permissions
        if (config.requiresConfirmation && !params.autoApprove) {
          const approved = await params.permissions.requestApproval({
            tool: config.name,
            input: validated,
          });
          if (!approved) throw new PermissionDeniedError();
        }

        // Execute with timeout
        return await Promise.race([
          config.handler(validated, params),
          timeout(params.timeout || 30000),
        ]);
      } catch (error) {
        return handleToolError(error, config.name);
      }
    },
    metadata: config.metadata,
  };
}

// Example tool: file reader
const readFileTool = buildTool({
  name: 'read_file',
  description: 'Read content from a file',
  schema: z.object({
    path: z.string(),
    encoding: z.enum(['utf8', 'base64']).default('utf8'),
  }),
  metadata: {
    readOnly: true,
    destructive: false,
    concurrencySafe: true,
    requiresConfirmation: false,
    category: 'filesystem',
  },
  handler: async (input, params) => {
    const fs = await import('fs/promises');
    const content = await fs.readFile(input.path, input.encoding);
    return { content, size: content.length };
  },
});
```

### 3. Permission Pipeline (Agent's Guardrails)

Four-stage permission management:

```typescript
// Permission modes spectrum
type PermissionMode =
  | 'autonomous'      // Auto-approve all
  | 'interactive'     // Prompt for destructive
  | 'strict'          // Prompt for all
  | 'read_only'       // Block destructive
  | 'sandbox';        // Isolated environment

interface PermissionPipeline {
  // Stage 1: Static rule matching
  checkRules(tool: ToolCall): Promise<RuleDecision>;
  
  // Stage 2: Speculative classification
  classify(tool: ToolCall): Promise<RiskLevel>;
  
  // Stage 3: User approval (if needed)
  requestApproval(tool: ToolCall): Promise<boolean>;
  
  // Stage 4: Audit logging
  logExecution(tool: ToolCall, result: ToolResult): Promise<void>;
}

class FourStagePermissionPipeline implements PermissionPipeline {
  async checkRules(tool: ToolCall): Promise<RuleDecision> {
    // Bash pattern matching for file paths
    for (const rule of this.rules) {
      if (matchPattern(rule.pattern, tool.input)) {
        return rule.decision;
      }
    }
    return { decision: 'defer', reason: 'no_match' };
  }

  async classify(tool: ToolCall): Promise<RiskLevel> {
    // 2-second Promise.race with LLM classifier
    const classification = await Promise.race([
      this.llmClassifier.classify(tool),
      timeout(2000, { risk: 'medium' }),
    ]);
    return classification.risk;
  }

  async requestApproval(tool: ToolCall): Promise<boolean> {
    if (this.mode === 'autonomous') return true;
    if (this.mode === 'read_only' && tool.metadata.destructive) {
      return false;
    }
    
    return await this.ui.prompt({
      message: `Allow ${tool.name}?`,
      details: tool.input,
      risk: await this.classify(tool),
    });
  }

  async logExecution(
    tool: ToolCall,
    result: ToolResult
  ): Promise<void> {
    await this.auditLog.write({
      timestamp: Date.now(),
      tool: tool.name,
      input: tool.input,
      output: result,
      approved: result.approved,
      userId: this.userId,
    });
  }
}
```

### 4. Context Management (Working Memory)

Progressive compression with circuit breaker:

```typescript
// Effective window formula
interface ContextWindow {
  total: number;
  reserved: {
    system: number;
    tools: number;
    recent: number;
  };
  available: number; // total - sum(reserved)
}

// Four-level progressive compression
class ContextManager {
  private stages: CompressionStage[] = [
    { name: 'snip', threshold: 0.7, handler: this.snipOldMessages },
    { name: 'micro_compact', threshold: 0.8, handler: this.microCompact },
    { name: 'collapse', threshold: 0.9, handler: this.collapseBlocks },
    { name: 'auto_compact', threshold: 0.95, handler: this.autoCompact },
  ];

  async compress(
    messages: Message[],
    budget: ContextWindow
  ): Promise<Message[]> {
    const usage = this.calculateUsage(messages);
    const ratio = usage / budget.available;

    // Apply compression stages progressively
    for (const stage of this.stages) {
      if (ratio >= stage.threshold) {
        messages = await stage.handler(messages, budget);
      }
    }

    // Circuit breaker if still over budget
    if (this.calculateUsage(messages) > budget.available) {
      throw new ContextOverflowError('Cannot compress within budget');
    }

    return messages;
  }

  private async snipOldMessages(
    messages: Message[],
    budget: ContextWindow
  ): Promise<Message[]> {
    // Keep system + recent + important, snip middle
    const recent = messages.slice(-budget.reserved.recent);
    const important = messages.filter(m => m.metadata?.important);
    const system = messages.filter(m => m.role === 'system');
    
    return [...system, ...important, ...recent];
  }

  private async microCompact(
    messages: Message[]
  ): Promise<Message[]> {
    // Compress tool results to summaries
    return messages.map(msg => {
      if (msg.role === 'tool' && msg.content.length > 1000) {
        return {
          ...msg,
          content: this.summarize(msg.content, 200),
          metadata: { ...msg.metadata, compressed: true },
        };
      }
      return msg;
    });
  }
}
```

### 5. Memory System (Long-term Memory)

Four closed-form memory types:

```typescript
// Memory types
interface MemorySystem {
  facts: Map<string, Fact>;        // Immutable truths
  preferences: Map<string, any>;    // User settings
  context: Map<string, Context>;    // Session state
  learned: Map<string, Learned>;    // Accumulated knowledge
}

class AgentMemory {
  private index: MemoryIndex; // MEMORY.md file

  async save(key: string, value: MemoryEntry): Promise<void> {
    // "Only save what cannot be derived"
    if (this.isDerived(value)) {
      return; // Skip redundant information
    }

    await this.store.set(key, value);
    await this.updateIndex(key, value);
  }

  async fork(parentMemory: AgentMemory): Promise<AgentMemory> {
    // Byte-level context inheritance for sub-agents
    const forked = new AgentMemory();
    
    // Copy immutable facts (reference, not clone)
    forked.facts = parentMemory.facts;
    
    // Clone mutable state
    forked.context = new Map(parentMemory.context);
    forked.preferences = new Map(parentMemory.preferences);
    
    // Start fresh learned knowledge
    forked.learned = new Map();
    
    return forked;
  }

  private async updateIndex(
    key: string,
    value: MemoryEntry
  ): Promise<void> {
    const index = await this.loadIndex();
    index.entries[key] = {
      type: value.type,
      created: value.timestamp,
      summary: this.summarize(value),
    };
    await this.saveIndex(index);
  }
}
```

### 6. Sub-Agent Fork Pattern

Recursive agent spawning with inheritance:

```typescript
interface SubAgentConfig {
  type: 'custom' | 'builtin';
  agentPath?: string;
  builtinName?: 'coordinator' | 'specialist' | 'reviewer' | 'planner';
  inheritContext: boolean;
  inheritMemory: boolean;
  inheritTools: boolean;
}

class AgentFork {
  async spawn(
    parent: Agent,
    config: SubAgentConfig
  ): Promise<Agent> {
    // Load agent definition
    const agentDef = config.type === 'builtin'
      ? await this.loadBuiltin(config.builtinName!)
      : await this.loadCustom(config.agentPath!);

    // Create child with inheritance
    const child = new Agent({
      ...agentDef,
      context: config.inheritContext
        ? parent.context.fork()
        : new Context(),
      memory: config.inheritMemory
        ? await parent.memory.fork()
        : new AgentMemory(),
      tools: config.inheritTools
        ? [...parent.tools.available]
        : agentDef.tools,
    });

    // Recursive fork prevention
    child.metadata.forkDepth = (parent.metadata.forkDepth || 0) + 1;
    if (child.metadata.forkDepth > this.maxForkDepth) {
      throw new MaxForkDepthError();
    }

    return child;
  }
}

// Coordinator-Worker pattern
async function coordinatorWorkerPattern(
  task: Task
): Promise<Result> {
  const coordinator = await agentFork.spawn(mainAgent, {
    type: 'builtin',
    builtinName: 'coordinator',
    inheritContext: true,
    inheritMemory: false,
    inheritTools: false, // Coordinators only orchestrate
  });

  const plan = await coordinator.plan(task);
  
  const workers = await Promise.all(
    plan.subtasks.map(subtask =>
      agentFork.spawn(coordinator, {
        type: 'builtin',
        builtinName: 'specialist',
        inheritContext: true,
        inheritMemory: true,
        inheritTools: true,
      })
    )
  );

  const results = await Promise.all(
    workers.map((worker, i) => worker.execute(plan.subtasks[i]))
  );

  return coordinator.synthesize(results);
}
```

### 7. MCP Integration

Model Context Protocol bridge for external tools:

```typescript
// Eight transport protocols
type MCPTransport =
  | 'stdio'
  | 'http'
  | 'websocket'
  | 'grpc'
  | 'ipc'
  | 'sse'
  | 'stdio+stderr'
  | 'custom';

// Five-state connection lifecycle
type ConnectionState =
  | 'disconnected'
  | 'connecting'
  | 'connected'
  | 'error'
  | 'reconnecting';

class MCPBridge {
  private connections = new Map<string, MCPConnection>();

  async connect(server: MCPServerConfig): Promise<void> {
    const conn = new MCPConnection(server);
    
    conn.on('state', (state) => {
      if (state === 'error') {
        this.reconnect(conn);
      }
    });

    await conn.connect();
    this.connections.set(server.name, conn);

    // Register MCP tools with three-segment naming
    const tools = await conn.listTools();
    for (const tool of tools) {
      this.registerTool({
        name: `mcp.${server.name}.${tool.name}`,
        description: tool.description,
        schema: tool.inputSchema,
        handler: (input) => conn.invoke(tool.name, input),
      });
    }
  }

  async invoke(
    toolName: string,
    input: unknown
  ): Promise<unknown> {
    // Parse three-segment name: mcp.{server}.{tool}
    const [_mcp, serverName, mcpToolName] = toolName.split('.');
    const conn = this.connections.get(serverName);
    
    if (!conn) throw new Error(`MCP server ${serverName} not connected`);
    
    return await conn.invoke(mcpToolName, input);
  }

  private async reconnect(conn: MCPConnection): Promise<void> {
    conn.state = 'reconnecting';
    await new Promise(r => setTimeout(r, 1000));
    await conn.connect();
  }
}
```

### 8. Skill System

Plugin architecture with frontmatter metadata:

```typescript
interface Skill {
  metadata: SkillMetadata;
  content: string;
  tools?: Tool[];
  hooks?: Hook[];
}

interface SkillMetadata {
  name: string;
  description: string;
  triggers: string[];
  version?: string;
  dependencies?: string[];
}

class SkillLoader {
  async load(path: string): Promise<Skill> {
    const raw = await this.readFile(path);
    const { frontmatter, content } = this.parseFrontmatter(raw);
    
    // Three-level parameter substitution
    const processed = await this.substitute(content, {
      env: process.env,
      config: this.config,
      runtime: this.getRuntimeVars(),
    });

    return {
      metadata: frontmatter as SkillMetadata,
      content: processed,
      tools: await this.extractTools(processed),
      hooks: await this.extractHooks(processed),
    };
  }

  private async substitute(
    content: string,
    context: SubstitutionContext
  ): Promise<string> {
    // Level 1: Environment variables
    content = content.replace(/\$\{env:(\w+)\}/g, (_, key) => 
      context.env[key] || ''
    );
    
    // Level 2: Config values
    content = content.replace(/\$\{config:(\w+)\}/g, (_, key) =>
      context.config[key] || ''
    );
    
    // Level 3: Runtime values
    content = content.replace(/\$\{runtime:(\w+)\}/g, (_, key) =>
      context.runtime[key] || ''
    );
    
    return content;
  }
}
```

## Configuration

Agent configuration follows six-layer priority chain:

```typescript
// Six-layer priority (highest to lowest)
const config = mergeConfig([
  runtimeOverrides,      // 1. Runtime flags
  environmentVariables,  // 2. Environment (AGENT_*)
  projectConfig,         // 3. .agentrc.json
  userConfig,            // 4. ~/.config/agent/config.json
  skillDefaults,         // 5. Skill-provided defaults
  systemDefaults,        // 6. Harness defaults
]);

// Feature flag system
interface FeatureFlags {
  // Compile-time flags
  enable_mcp: boolean;
  enable_sub_agents: boolean;
  enable_plan_mode: boolean;
  
  // Runtime flags
  auto_approve: boolean;
  debug_mode: boolean;
  trace_tools: boolean;
}

// Configuration with security boundaries
interface AgentConfig {
  model: ModelConfig;
  tools: ToolConfig;
  permissions: PermissionConfig;
  context: ContextConfig;
  memory: MemoryConfig;
  features: FeatureFlags;
  
  // Security constraints
  security: {
    maxForkDepth: number;
    allowedPaths: string[];
    blockedCommands: string[];
    rateLimits: RateLimitConfig;
  };
}
```

## Performance Optimization

### Startup Optimization

```typescript
// Lazy loading pattern (160ms → 65ms, -59%)
class LazyAgentLoader {
  private loaded = new Set<string>();

  async loadOnDemand(module: string): Promise<void> {
    if (this.loaded.has(module)) return;
    
    switch (module) {
      case 'tools':
        await this.loadTools();
        break;
      case 'mcp':
        await this.loadMCP();
        break;
      case 'skills':
        await this.loadSkills();
        break;
    }
    
    this.loaded.add(module);
  }

  private async loadTools(): Promise<void> {
    // Parallel import with Promise.all
    const toolModules = [
      'filesystem',
      'shell',
      'network',
      'search',
    ];
    
    await Promise.all(
      toolModules.map(m => import(`./tools/${m}`))
    );
  }
}

// Concurrent control
class ConcurrencyController {
  constructor(private maxConcurrent: number = 5) {}

  async executeInBatches<T>(
    tasks: (() => Promise<T>)[],
  ): Promise<T[]> {
    const results: T[] = [];
    
    for (let i = 0; i < tasks.length; i += this.maxConcurrent) {
      const batch = tasks.slice(i, i + this.maxConcurrent);
      const batchResults = await Promise.all(
        batch.map(task => task())
      );
      results.push(...batchResults);
    }
    
    return results;
  }
}
```

## Observability

Four-layer monitoring system:

```typescript
interface Observability {
  // Layer 1: Structured logging
  logger: Logger;
  
  // Layer 2: Metrics
  metrics: MetricsCollector;
  
  // Layer 3: Tracing
  tracer: DistributedTracer;
  
  // Layer 4: Debugging
  debugger: AgentDebugger;
}

class AgentObservability implements Observability {
  logger = new Logger({
    level: process.env.LOG_LEVEL || 'info',
    format: 'json',
    fields: {
      agent_id: this.agentId,
      session_id: this.sessionId,
    },
  });

  metrics = new MetricsCollector({
    counters: [
      'tool_executions',
      'permission_denials',
      'context_compressions',
    ],
    histograms: [
      'turn_duration',
      'tool_latency',
      'context_size',
    ],
  });

  tracer = new DistributedTracer({
    service: 'agent-harness',
    samplingRate: 0.1,
    exporters: [
      new OTLPExporter({ endpoint: process.env.OTEL_ENDPOINT }),
    ],
  });

  async trace<T>(
    operation: string,
    fn: () => Promise<T>
  ): Promise<T> {
    const span = this.tracer.startSpan(operation);
    
    try {
      const result = await fn();
      span.setStatus({ code: 'OK' });
      return result;
    } catch (error) {
      span.setStatus({ code: 'ERROR', message: error.message });
      throw error;
    } finally {
      span.end();
    }
  }
}
```

## Common Patterns

### Pattern 1: Graceful Degradation

```typescript
async function executeWithFallback<T>(
  primary: () => Promise<T>,
  fallback: () => Promise<T>,
  timeout: number = 5000
): Promise<T> {
  try {
    return await Promise.race([
      primary(),
      new Promise<T>((_, reject) =>
        setTimeout(() => reject(new TimeoutError()), timeout)
      ),
    ]);
  } catch (error) {
    console.warn('Primary failed, using fallback:', error);
    return await fallback();
  }
}

// Usage in tool execution
const result = await executeWithFallback(
  () => mcpBridge.invoke('mcp.search.web', query),
  () => localSearch.execute(query),
  3000
);
```

### Pattern 2: Dependency Injection

```typescript
// QueryDeps pattern for testability
interface QueryDeps {
  llm: LLMClient;
  tools: ToolRegistry;
  permissions: PermissionPipeline;
  context: ContextManager;
  memory: AgentMemory;
  ui: UserInterface;
  config: AgentConfig;
}

class Agent {
  constructor(private deps: QueryDeps) {}

  async run(query: string): Promise<void> {
    for await (const event of conversationLoop(this.deps)) {
      await this.deps.ui.render(event);
    }
  }
}

// Testing with mocks
const testAgent = new Agent({
  llm: new MockLLM(),
  tools: new MockToolRegistry(),
  permissions: new AutoApprovePermissions(),
  context: new InMemoryContext(),
  memory: new InMemoryMemory(),
  ui: new TestUI(),
  config: testConfig,
});
```

### Pattern 3: Hook-Based Extension

```typescript
// 26 lifecycle events
type HookEvent =
  | 'agent:init'
  | 'agent:start'
  | 'turn:start'
  | 'llm:request'
  | 'llm:response'
  | 'tool:call'
  | 'tool:result'
  | 'permission:check'
  | 'context:compress'
  | 'memory:save'
  // ... 16 more

class HookSystem {
  private hooks = new Map<HookEvent, Hook[]>();

  register(event: HookEvent, hook: Hook): void {
    const hooks = this.hooks.get(event) || [];
    hooks.push(hook);
    hooks.sort((a, b) => a.priority - b.priority);
    this.hooks.set(event, hooks);
  }

  async trigger(event: HookEvent, data: unknown): Promise<void> {
    const hooks = this.hooks.get(event) || [];
    
    for (const hook of hooks) {
      try {
        await hook.handler(data);
      } catch (error) {
        console.error(`Hook ${hook.name} failed:`, error);
        // Continue with other hooks
      }
    }
  }
}

// Example: Audit logging hook
hookSystem.register('tool:result', {
  name: 'audit_logger',
  priority: 100,
  handler: async (data) => {
    await auditLog.write({
      timestamp: Date.now(),
      tool: data.tool,
      result: data.result,
    });
  },
});
```

## Troubleshooting

### Context Overflow

```typescript
// Symptom: Agent fails with "context budget exceeded"

// Solution 1: Increase reserved budget
const config = {
  context: {
    total: 200000,
    reserved: {
      system: 2000,
      tools: 5000,
      recent: 10000, // Increase this
    },
  },
};

// Solution 2: Enable aggressive compression
const config = {
  features: {
    enable_auto_compact: true,
    compression_threshold: 0.7, // Trigger earlier
  },
};

// Solution 3: Implement custom compression
class CustomCompressor extends ContextManager {
  async compress(messages: Message[]): Promise<Message[]> {
    // Your domain-specific logic
    return messages.filter(m => m.metadata.important);
  }
}
```

### Permission Deadlock

```typescript
// Symptom: Agent hangs waiting for approval

// Solution: Set timeout on permission requests
class TimeoutPermissions extends PermissionPipeline {
  async requestApproval(tool: ToolCall): Promise<boolean> {
    try {
      return await Promise.race([
        super.requestApproval(tool),
        timeout(30000, false), // Auto-deny after 30s
      ]);
    } catch (error) {
      console.error('Permission timeout:', tool.name);
      return false;
    }
  }
}
```

### MCP Connection Failures

```typescript
// Symptom: MCP tools fail with "connection refused"

// Solution: Implement exponential backoff
class ResilientMCPBridge extends MCPBridge {
  async reconnect(
    conn: MCPConnection,
    attempt: number = 0
  ): Promise<void> {
    const delay = Math.min(1000 * Math.pow(2, attempt), 30000);
    
    try {
      await new Promise(r => setTimeout(r, delay));
      await conn.connect();
    } catch (error) {
      if (attempt < 5) {
        await this.reconnect(conn, attempt + 1);
      } else {
        throw new MaxRetriesError();
      }
    }
  }
}
```

### Memory Leak in Long Sessions

```typescript
// Symptom: Memory usage grows unbounded

// Solution: Implement periodic cleanup
class BoundedMemory extends AgentMemory {
  private maxSize = 1000;

  async save(key: string, value: MemoryEntry): Promise<void> {
    await super.save(key, value);
    
    // Evict old entries if over limit
    if (this.store.size > this.maxSize) {
      const sorted = [...this.store.entries()]
        .sort((a, b) => a[1].timestamp - b[1].timestamp);
      
      const toDelete = sorted.slice(0, this.store.size - this.maxSize);
      for (const [key] of toDelete) {
        this.store.delete(key);
      }
    }
  }
}
```

## Advanced Usage

### Building a Custom Harness

Six-step implementation roadmap:

```typescript
// Step 1: Define agent interface
interface CustomAgent {
  run(query: string): Promise<void>;
  addTool(tool: Tool): void;
  fork(config: SubAgentConfig): Promise<CustomAgent>;
}

// Step 2: Implement conversation loop
class CustomAgentImpl implements CustomAgent {
  async run(query: string): Promise<void> {
    const deps = this.buildDeps();
    
    for await (const event of conversationLoop(deps)) {
      await this.handleEvent(event);
    }
  }
}

// Step 3: Build tool registry
class CustomToolRegistry extends ToolRegistry {
  // Your custom tool loading logic
}

// Step 4: Implement permission system
class CustomPermissions extends PermissionPipeline {
  // Your custom approval flow
}

// Step 5: Add observability
const agent = new CustomAgentImpl({
  observability: new AgentObservability(),
});

// Step 6: Test and iterate
await agent.run('Build a web scraper');
```

## References

- **Online Documentation**: https://lintsinghua.github.io
- **Repository**:
