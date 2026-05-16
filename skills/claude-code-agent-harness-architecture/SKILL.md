---
name: claude-code-agent-harness-architecture
description: Deep architectural knowledge of AI Agent Harness design patterns, event loops, tool systems, permission pipelines, and context management for building production-grade AI agents
triggers:
  - how do I design an agent harness architecture
  - explain agent event loop patterns
  - help me build a tool permission system
  - how does context compression work in agents
  - show me agent memory system design
  - implement a fork-based sub-agent pattern
  - design an MCP protocol bridge
  - build an agent configuration hierarchy
---

# Claude Code Agent Harness Architecture

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## Overview

This skill provides deep architectural knowledge from the 420,000-word analysis of Claude Code's Agent Harness design. It covers the fundamental patterns, design decisions, and implementation strategies for building production-grade AI agents, including:

- **Event-driven conversation loops** (async generator patterns)
- **Tool systems** with concurrent execution and safety guarantees
- **4-stage permission pipelines** (declare → validate → execute → verify)
- **Context compression** strategies for token budget management
- **Memory systems** with fork-based inheritance
- **Hook-based lifecycle extensibility**
- **Sub-agent orchestration** patterns
- **MCP protocol integration**

While based on Claude Code's architecture, these patterns are **framework-agnostic** and applicable to any agent system (LangChain, AutoGen, CrewAI, custom builds).

## Core Architecture Principles

### 1. The Five Design Pillars

```typescript
/**
 * Agent Harness Design Principles (from Ch01)
 */
interface AgentHarnessPrinciples {
  // 1. Async-first: Everything yields, nothing blocks
  eventLoop: AsyncGenerator<AgentEvent, void, void>;
  
  // 2. Dependency injection: No global state
  deps: QueryDeps;
  
  // 3. Fail-safe by default: Explicit opt-in for destructive ops
  toolSafety: 'readOnly' | 'requiresApproval' | 'destructive';
  
  // 4. Observable: Every decision point emits events
  hooks: HookSystem;
  
  // 5. Composable: Agents can fork and orchestrate sub-agents
  fork: (context: ConversationContext) => Agent;
}
```

### 2. Event Loop Architecture (Ch02)

The core conversation loop is an **async generator** that yields five event types:

```typescript
/**
 * Main conversation loop - the "heartbeat" of the agent
 */
async function* conversationLoop(
  deps: QueryDeps
): AsyncGenerator<ConversationEvent, void, void> {
  while (true) {
    // 1. Get model response
    const response = await deps.model.generate(deps.context);
    
    // 2. Yield thinking event
    if (response.thinking) {
      yield { type: 'thinking', content: response.thinking };
    }
    
    // 3. Yield content event
    if (response.content) {
      yield { type: 'content', delta: response.content };
    }
    
    // 4. Handle tool calls
    if (response.toolCalls) {
      for (const call of response.toolCalls) {
        yield { type: 'tool_call', tool: call };
        
        const result = await executeToolWithPermission(call, deps);
        yield { type: 'tool_result', result };
        
        deps.context.addToolResult(result);
      }
      continue; // Loop back for next model turn
    }
    
    // 5. Check termination
    const termination = checkTermination(response, deps);
    if (termination) {
      yield { type: 'done', reason: termination };
      break;
    }
  }
}

// 10 termination reasons (from Ch02)
type TerminationReason =
  | 'stop'              // Model returned stop token
  | 'max_turns'         // Hit conversation limit
  | 'token_budget'      // Exceeded context window
  | 'user_interrupt'    // Manual cancellation
  | 'tool_error'        // Unrecoverable tool failure
  | 'permission_denied' // User rejected critical tool
  | 'timeout'           // Wall-clock time limit
  | 'cost_limit'        // API cost exceeded
  | 'safety_violation'  // Content policy triggered
  | 'goal_achieved';    // Plan completed
```

### 3. Tool System (Ch03)

Tools follow a strict **5-element protocol**:

```typescript
/**
 * Tool protocol with safety guarantees
 */
interface Tool<Input, Output, Params = void> {
  // 1. Identity
  name: string;
  description: string;
  
  // 2. Contract (Zod schemas)
  inputSchema: z.ZodSchema<Input>;
  outputSchema: z.ZodSchema<Output>;
  
  // 3. Safety properties
  readOnly: boolean;           // Can it mutate state?
  destructive: boolean;        // Is mutation irreversible?
  concurrencySafe: boolean;    // Safe to run in parallel?
  
  // 4. Execution
  execute: (input: Input, params: Params) => Promise<Output>;
  
  // 5. Permissions
  requiresApproval?: (input: Input) => boolean;
  permissionScope?: string[];
}

/**
 * Fail-safe tool factory (Ch03)
 */
function buildTool<I, O, P = void>(
  spec: ToolSpec<I, O, P>
): Tool<I, O, P> {
  return {
    ...spec,
    execute: async (input, params) => {
      try {
        // 1. Validate input
        const validated = spec.inputSchema.parse(input);
        
        // 2. Execute with timeout
        const result = await withTimeout(
          spec.execute(validated, params),
          spec.timeout ?? 30000
        );
        
        // 3. Validate output
        return spec.outputSchema.parse(result);
      } catch (error) {
        // 4. Return structured error (never throw)
        return {
          success: false,
          error: error.message,
          code: error.code ?? 'TOOL_ERROR'
        } as O;
      }
    }
  };
}

/**
 * Example: File read tool (read-only, safe)
 */
const readFileTool = buildTool({
  name: 'read_file',
  description: 'Read file contents',
  inputSchema: z.object({
    path: z.string(),
    encoding: z.enum(['utf8', 'base64']).default('utf8')
  }),
  outputSchema: z.object({
    content: z.string(),
    size: z.number()
  }),
  readOnly: true,
  destructive: false,
  concurrencySafe: true,
  execute: async ({ path, encoding }) => {
    const content = await fs.readFile(path, encoding);
    return { content, size: content.length };
  }
});

/**
 * Example: Bash execution (destructive, needs approval)
 */
const bashTool = buildTool({
  name: 'bash',
  description: 'Execute shell command',
  inputSchema: z.object({
    command: z.string(),
    cwd: z.string().optional()
  }),
  outputSchema: z.object({
    stdout: z.string(),
    stderr: z.string(),
    exitCode: z.number()
  }),
  readOnly: false,
  destructive: true,
  concurrencySafe: false,
  requiresApproval: (input) => {
    // Auto-approve safe commands
    const safePatterns = [/^ls/, /^pwd/, /^echo/];
    return !safePatterns.some(p => p.test(input.command));
  },
  execute: async ({ command, cwd }) => {
    const result = await exec(command, { cwd });
    return result;
  }
});
```

### 4. Permission Pipeline (Ch04)

Four-stage pipeline for tool safety:

```typescript
/**
 * 4-stage permission pipeline
 */
async function executeToolWithPermission<I, O>(
  call: ToolCall<I>,
  deps: QueryDeps
): Promise<ToolResult<O>> {
  const tool = deps.toolRegistry.get(call.name);
  
  // STAGE 1: Declaration check
  if (!tool) {
    return { error: 'Tool not found', code: 'UNKNOWN_TOOL' };
  }
  
  // STAGE 2: Validation
  const validation = tool.inputSchema.safeParse(call.input);
  if (!validation.success) {
    return { 
      error: 'Invalid input', 
      code: 'VALIDATION_ERROR',
      details: validation.error 
    };
  }
  
  // STAGE 3: Permission check
  const needsApproval = 
    tool.destructive || 
    tool.requiresApproval?.(call.input);
    
  if (needsApproval) {
    const approved = await deps.permissionManager.requestApproval({
      tool: tool.name,
      input: call.input,
      reason: classifyIntent(call),
      timeout: 120000 // 2 minutes
    });
    
    if (!approved) {
      return { error: 'Permission denied', code: 'DENIED' };
    }
  }
  
  // STAGE 4: Execution + verification
  const result = await tool.execute(validation.data, deps.params);
  
  // Post-execution verification (if configured)
  if (tool.verify) {
    const verified = await tool.verify(result, call.input);
    if (!verified.success) {
      await tool.rollback?.(result);
      return { error: verified.reason, code: 'VERIFICATION_FAILED' };
    }
  }
  
  return result;
}

/**
 * 5 permission modes (Ch04)
 */
type PermissionMode =
  | 'auto'        // Auto-approve safe tools
  | 'prompt'      // Prompt for destructive tools
  | 'strict'      // Prompt for all tools
  | 'deny'        // Deny all destructive tools
  | 'passthrough'; // No checks (dev mode only)

/**
 * Speculative classifier (Ch04)
 * Runs in parallel with user approval for faster UX
 */
async function classifyIntent(
  call: ToolCall,
  timeout = 2000
): Promise<IntentClassification> {
  return Promise.race([
    // Fast heuristic classifier
    quickClassify(call),
    
    // LLM-based classifier (may be slow)
    llmClassify(call),
    
    // Timeout fallback
    delay(timeout).then(() => ({ confidence: 'low', safe: false }))
  ]);
}
```

### 5. Context Management (Ch07)

Token budget management with 4-level compression:

```typescript
/**
 * Effective context window formula
 */
interface ContextBudget {
  total: number;           // Model's max context (200k for Claude 3.5)
  reserved: {
    systemPrompt: number;  // ~2k tokens
    tools: number;         // ~500 tokens per tool
    response: number;      // ~4k tokens reserved for output
  };
  effective: number;       // total - sum(reserved)
}

/**
 * 4-level progressive compression (Ch07)
 */
class ContextManager {
  private budget: ContextBudget;
  
  async compress(context: Message[]): Promise<Message[]> {
    const usage = this.estimateTokens(context);
    
    // Level 1: Snip - Remove old tool results
    if (usage > this.budget.effective * 0.8) {
      context = this.snipOldToolResults(context);
    }
    
    // Level 2: MicroCompact - Collapse repeated patterns
    if (usage > this.budget.effective * 0.9) {
      context = this.microCompact(context);
    }
    
    // Level 3: Collapse - Summarize old conversations
    if (usage > this.budget.effective * 0.95) {
      context = await this.collapseOldMessages(context);
    }
    
    // Level 4: AutoCompact - Emergency fallback
    if (usage > this.budget.effective) {
      context = await this.autoCompact(context);
    }
    
    return context;
  }
  
  /**
   * Circuit breaker pattern (Ch07)
   * Prevents infinite compression loops
   */
  private autoCompact(context: Message[]): Message[] {
    const attempts = 0;
    const maxAttempts = 3;
    
    while (this.estimateTokens(context) > this.budget.effective) {
      if (attempts >= maxAttempts) {
        // Circuit breaker triggered
        throw new Error('CONTEXT_OVERFLOW');
      }
      
      // Remove least important 20% of messages
      context = this.removeByImportance(context, 0.2);
      attempts++;
    }
    
    return context;
  }
}
```

### 6. Memory System (Ch06)

4 types of closed-form memory:

```typescript
/**
 * Memory types (Ch06)
 */
interface MemorySystem {
  // 1. Conversation memory (session-scoped)
  conversation: {
    messages: Message[];
    toolResults: ToolResult[];
    userPreferences: Record<string, unknown>;
  };
  
  // 2. Project memory (workspace-scoped)
  project: {
    structure: FileTree;
    dependencies: PackageJson;
    conventions: CodingStyle;
  };
  
  // 3. User memory (global)
  user: {
    preferences: UserConfig;
    learnedPatterns: Pattern[];
  };
  
  // 4. Skill memory (external knowledge)
  skills: {
    installed: Skill[];
    index: SkillIndex;
  };
}

/**
 * Memory persistence principle (Ch06):
 * "Only save information that cannot be derived"
 */
async function shouldPersist(info: Information): Promise<boolean> {
  // Don't save: file contents (can be re-read)
  if (info.type === 'file_content') return false;
  
  // Don't save: computed values (can be re-computed)
  if (info.type === 'derived_value') return false;
  
  // DO save: user preferences (can't be inferred reliably)
  if (info.type === 'user_preference') return true;
  
  // DO save: learned patterns (expensive to re-learn)
  if (info.type === 'pattern' && info.confidence > 0.8) return true;
  
  return false;
}

/**
 * Fork-based memory inheritance (Ch09)
 */
class Agent {
  async fork(taskDescription: string): Promise<Agent> {
    const childAgent = new Agent({
      // Byte-level context copy
      context: this.context.clone(),
      
      // Shared memory (copy-on-write)
      memory: this.memory.createSnapshot(),
      
      // Isolated tool permissions
      permissions: this.permissions.createChild(),
      
      // Custom system prompt
      systemPrompt: `You are a sub-agent working on: ${taskDescription}
      
      Parent context:
      ${this.context.summarize()}
      
      When done, return control to parent.`
    });
    
    return childAgent;
  }
}
```

### 7. Hook System (Ch08)

Lifecycle extensibility without modifying core:

```typescript
/**
 * Hook system (Ch08)
 */
interface Hook {
  name: string;
  type: 'before' | 'after' | 'around' | 'replace' | 'stream';
  event: HookEvent;
  priority: number; // 0-100, higher = earlier
  execute: (data: HookData) => Promise<HookResult>;
}

/**
 * 26 lifecycle events (subset shown)
 */
type HookEvent =
  | 'query:start'
  | 'query:end'
  | 'tool:before_execute'
  | 'tool:after_execute'
  | 'context:before_compress'
  | 'context:after_compress'
  | 'permission:request'
  | 'permission:granted'
  | 'memory:save'
  | 'agent:fork';

/**
 * Example: Custom approval hook
 */
const customApprovalHook: Hook = {
  name: 'custom-approval',
  type: 'around',
  event: 'permission:request',
  priority: 50,
  execute: async (data) => {
    const { tool, input } = data;
    
    // Custom logic: auto-approve npm installs for known packages
    if (tool === 'bash' && input.command.startsWith('npm install')) {
      const pkg = input.command.split(' ')[2];
      if (await isKnownPackage(pkg)) {
        return { approved: true, reason: 'Known package' };
      }
    }
    
    // Fall through to default approval flow
    return { continue: true };
  }
};

/**
 * Hook registration
 */
class HookSystem {
  private hooks = new Map<HookEvent, Hook[]>();
  
  register(hook: Hook): void {
    const existing = this.hooks.get(hook.event) ?? [];
    existing.push(hook);
    // Sort by priority (descending)
    existing.sort((a, b) => b.priority - a.priority);
    this.hooks.set(hook.event, existing);
  }
  
  async trigger(event: HookEvent, data: HookData): Promise<HookResult> {
    const hooks = this.hooks.get(event) ?? [];
    
    for (const hook of hooks) {
      const result = await hook.execute(data);
      
      // If hook says to stop, short-circuit
      if (result.stop) return result;
      
      // If hook modified data, pass it forward
      if (result.data) {
        data = { ...data, ...result.data };
      }
    }
    
    return { continue: true, data };
  }
}
```

### 8. Sub-Agent Patterns (Ch09-10)

Coordinator-Worker orchestration:

```typescript
/**
 * Coordinator pattern (Ch10)
 * "Plan but don't execute"
 */
class CoordinatorAgent {
  private workers = new Map<string, Agent>();
  
  async orchestrate(task: Task): Promise<Result> {
    // 1. Decompose into subtasks
    const plan = await this.decompose(task);
    
    // 2. Assign to workers
    for (const subtask of plan.subtasks) {
      const worker = await this.getOrCreateWorker(subtask.capability);
      
      // Fork with isolated context
      const workerAgent = await worker.fork(subtask.description);
      
      this.workers.set(subtask.id, workerAgent);
    }
    
    // 3. Monitor progress (don't execute tools yourself)
    const results = await Promise.all(
      Array.from(this.workers.entries()).map(([id, agent]) =>
        agent.run().then(result => ({ id, result }))
      )
    );
    
    // 4. Synthesize results
    return this.synthesize(results);
  }
  
  /**
   * Coordinator constraints (Ch10)
   */
  private enforceConstraints(tool: Tool): boolean {
    // Coordinators can only use planning/communication tools
    const allowedTools = [
      'create_subtask',
      'assign_to_worker',
      'wait_for_worker',
      'get_worker_status'
    ];
    
    return allowedTools.includes(tool.name);
  }
}

/**
 * 4 addressing modes (Ch10)
 */
type WorkerAddress =
  | { type: 'id'; value: string }                    // By ID
  | { type: 'capability'; value: string[] }          // By skill tags
  | { type: 'context'; value: ConversationContext }  // By context match
  | { type: 'cost'; value: number };                 // By cost budget
```

### 9. MCP Integration (Ch12)

Bridge pattern for external protocols:

```typescript
/**
 * MCP transport types (Ch12)
 */
type MCPTransport =
  | { type: 'stdio'; command: string; args: string[] }
  | { type: 'sse'; url: string }
  | { type: 'websocket'; url: string }
  | { type: 'unix'; path: string }
  | { type: 'tcp'; host: string; port: number }
  | { type: 'http'; endpoint: string }
  | { type: 'grpc'; target: string }
  | { type: 'custom'; handler: CustomTransport };

/**
 * MCP server connection lifecycle
 */
class MCPServer {
  private state: 'disconnected' | 'connecting' | 'connected' | 'error' | 'reconnecting';
  
  async connect(config: MCPConfig): Promise<void> {
    this.state = 'connecting';
    
    try {
      const transport = this.createTransport(config.transport);
      await transport.connect();
      
      // Initialize protocol
      const initResponse = await transport.send({
        jsonrpc: '2.0',
        method: 'initialize',
        params: {
          protocolVersion: '2024-11-05',
          capabilities: {
            tools: {},
            resources: {},
            prompts: {}
          },
          clientInfo: {
            name: 'claude-code',
            version: '1.0.0'
          }
        }
      });
      
      this.state = 'connected';
      this.registerTools(initResponse.result.capabilities.tools);
    } catch (error) {
      this.state = 'error';
      await this.reconnect();
    }
  }
  
  /**
   * 3-segment tool naming (Ch12)
   */
  private registerTools(tools: MCPTool[]): void {
    for (const tool of tools) {
      const name = `mcp:${this.serverName}:${tool.name}`;
      
      this.toolRegistry.register({
        name,
        description: tool.description,
        inputSchema: z.object(tool.inputSchema),
        execute: async (input) => {
          return this.transport.send({
            method: 'tools/call',
            params: { name: tool.name, arguments: input }
          });
        }
      });
    }
  }
}
```

## Configuration Hierarchy (Ch05)

6-layer configuration priority chain:

```typescript
/**
 * Configuration layers (highest to lowest priority)
 */
interface ConfigHierarchy {
  // 1. Runtime flags (CLI args)
  runtime: {
    '--model': string;
    '--temperature': number;
  };
  
  // 2. Environment variables
  env: {
    ANTHROPIC_API_KEY: string;
    CLAUDE_CODE_CONFIG_PATH: string;
  };
  
  // 3. Workspace .claude/config.json
  workspace: {
    model: string;
    tools: ToolConfig[];
  };
  
  // 4. User ~/.config/claude-code/config.json
  user: {
    defaultModel: string;
    keybindings: Record<string, string>;
  };
  
  // 5. Project package.json (claudeCode field)
  project: {
    skills: string[];
    mcpServers: MCPConfig[];
  };
  
  // 6. Built-in defaults
  defaults: {
    model: 'claude-3-5-sonnet-20241022';
    temperature: 0.7;
  };
}

/**
 * Config merge strategy (Ch05)
 */
function mergeConfigs(...configs: Partial<Config>[]): Config {
  return configs.reduce((merged, config) => {
    // Arrays: concatenate + dedupe
    if (Array.isArray(config)) {
      return [...new Set([...merged, ...config])];
    }
    
    // Objects: deep merge
    if (typeof config === 'object') {
      return deepMerge(merged, config);
    }
    
    // Primitives: last one wins
    return config;
  }, {} as Config);
}
```

## Performance Patterns (Ch13)

```typescript
/**
 * Startup optimization (Ch13)
 * Before: 160ms, After: 65ms (-59%)
 */
class QueryEngine {
  // Lazy-load heavy dependencies
  private _toolRegistry?: ToolRegistry;
  
  get toolRegistry(): ToolRegistry {
    if (!this._toolRegistry) {
      this._toolRegistry = new ToolRegistry();
      this.loadCoreTools(); // Only load when first accessed
    }
    return this._toolRegistry;
  }
  
  /**
   * Parallel initialization
   */
  async initialize(): Promise<void> {
    await Promise.all([
      this.loadConfig(),      // ~20ms
      this.connectMCP(),      // ~30ms
      this.initializeMemory() // ~15ms
    ]);
    // Total: max(20, 30, 15) = 30ms (vs 65ms sequential)
  }
  
  /**
   * Concurrent tool execution with partitioning
   */
  async executeTools(calls: ToolCall[]): Promise<ToolResult[]> {
    // Partition by concurrency safety
    const { safe, unsafe } = partition(calls, c => 
      this.toolRegistry.get(c.name).concurrencySafe
    );
    
    // Execute safe tools in parallel
    const safeResults = await Promise.all(
      safe.map(call => this.executeTool(call))
    );
    
    // Execute unsafe tools sequentially
    const unsafeResults = [];
    for (const call of unsafe) {
      unsafeResults.push(await this.executeTool(call));
    }
    
    return [...safeResults, ...unsafeResults];
  }
}
```

## Building Your Own Harness (Ch15)

6-step implementation roadmap:

```typescript
/**
 * Step 1: Core event loop
 */
async function* createAgentLoop(deps: QueryDeps) {
  while (true) {
    const response = await deps.model.generate(deps.context);
    yield* processResponse(response, deps);
    
    if (shouldTerminate(response, deps)) break;
  }
}

/**
 * Step 2: Dependency injection
 */
interface QueryDeps {
  model: ModelClient;
  context: ContextManager;
  tools: ToolRegistry;
  permissions: PermissionManager;
  memory: MemorySystem;
  hooks: HookSystem;
  config: Config;
}

/**
 * Step 3: Tool registry
 */
class ToolRegistry {
  private tools = new Map<string, Tool>();
  
  register(tool: Tool): void {
    this.tools.set(tool.name, tool);
  }
  
  async execute(call: ToolCall, deps: QueryDeps): Promise<ToolResult> {
    const tool = this.tools.get(call.name);
    if (!tool) throw new Error('Unknown tool');
    
    // Run through permission pipeline
    return executeToolWithPermission(tool, call, deps);
  }
}

/**
 * Step 4: Observable events
 */
class EventBus {
  private listeners = new Map<string, Function[]>();
  
  emit(event: string, data: unknown): void {
    const handlers = this.listeners.get(event) ?? [];
    handlers.forEach(fn => fn(data));
  }
  
  on(event: string, handler: Function): void {
    const existing = this.listeners.get(event) ?? [];
    this.listeners.set(event, [...existing, handler]);
  }
}

/**
 * Step 5: Circular dependency resolution
 */
class DependencyContainer {
  private instances = new Map<string, unknown>();
  private factories = new Map<string, () => unknown>();
  
  register<T>(name: string, factory: () => T): void {
    this.factories.set(name, factory);
  }
  
  resolve<T>(name: string): T {
    // Return cached instance if exists
    if (this.instances.has(name)) {
      return this.instances.get(name) as T;
    }
    
    // Create proxy to break circular deps
    const proxy = new Proxy({}, {
      get: (_, prop) => {
        const instance = this.instances.get(name);
        return instance?.[prop];
      }
    });
    
    this.instances.set(name, proxy);
    
    // Execute factory
    const factory = this.factories.get(name);
    const instance = factory();
    
    // Replace proxy with real instance
    this.instances.set(name, instance);
    
    return instance as T;
  }
}

/**
 * Step 6: Full agent implementation
 */
class Agent {
  constructor(private deps: QueryDeps) {}
  
  async run(userMessage: string): Promise<string> {
    // Add user message to context
    this.deps.context.add({ role: 'user', content: userMessage });
    
    // Start conversation loop
    const loop = createAgentLoop(this.deps);
    
    let finalResponse = '';
    
    for await (const event of loop) {
      // Emit observable events
      this.deps.hooks.trigger(`query:${event.type}`, event);
      
      if (event.type === 'content') {
        finalResponse += event.delta;
      }
      
      if (event.type === 'done') {
        break;
      }
    }
    
    return finalResponse;
  }
}
```

## Common Patterns

### Pattern 1: Fail-Safe Defaults

```typescript
/**
 * Always default to safe mode
 */
const tool = buildTool({
  name: 'write_file',
  readOnly: false,         // Explicit: this mutates
  destructive: true,       // Explicit: mutation is irreversible
  requiresApproval: () => true, // Default: always ask
  execute: async (input) => {
    // Implementation
  }
});
```

### Pattern 2: Progressive Enhancement

```typescript
/**
 * Start simple, add capabilities incrementally
 */
class Agent {
  private capabilities = new Set<string>();
  
  enableCapability(name: string): void {
    this.capabilities.add(name);
  }
  
  async run(message: string): Promise<string> {
    // Core loop works without any capabilities
    const loop = this.createLoop();
    
    // Add tools if enabled
    if (this.capabilities.has('tools')) {
      loop.use(toolMiddleware);
    }
    
    // Add memory if enabled
    if (this.capabilities.has('memory')) {
      loop.use(memoryMiddleware);
    }
    
    return loop.run(message);
  }
}
```

### Pattern 3: Event Sourcing for Debugging

```typescript
/**
 * Log every decision as a replayable event
 */
class DebugLogger {
  private events: AgentEvent[] = [];
  
  log(event: AgentEvent
