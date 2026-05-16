---
name: claude-code-agent-harness-architecture
description: Deep architectural knowledge of AI Agent Harness design patterns, systems, and implementation based on Claude Code analysis
triggers:
  - how do I build an agent harness
  - explain agent conversation loop architecture
  - show me agent tool system design
  - how does agent permission pipeline work
  - implement agent context management
  - design agent memory system
  - build agent with MCP integration
  - create agent coordinator pattern
---

# Claude Code Agent Harness Architecture

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## Overview

This skill provides comprehensive knowledge of Agent Harness architecture—the runtime framework that powers AI agents. Based on deep analysis of Claude Code's production implementation, it covers:

- **Conversation loop** patterns (async generators, yield events, termination)
- **Tool system** protocols (`Tool<I,O,P>`, factory patterns, concurrent execution)
- **Permission pipeline** (4-stage authorization, safety classifiers)
- **Context management** (token budgets, compression strategies, circuit breakers)
- **Memory systems** (4 memory types, fork inheritance)
- **Hooks & lifecycle** (26 events, 5 hook types)
- **Sub-agent orchestration** (Fork patterns, Coordinator mode)
- **MCP integration** (protocol bridges, transport layers)

**Not a user guide**—this is architectural knowledge for building production-grade agent systems.

## Core Architecture Principles

### 1. Five Design Pillars

```typescript
// Agent Harness design philosophy
const DESIGN_PRINCIPLES = {
  transparency: "All decisions visible to user",
  composability: "Tools, hooks, agents as building blocks",
  safety: "Multi-layer permission gates + circuit breakers",
  streaming: "AsyncGenerator-based reactive architecture",
  dependency_injection: "QueryDeps container for testability"
} as const;
```

### 2. Technology Stack

- **Runtime**: Bun (fast startup, native TypeScript)
- **UI**: React + Ink (TUI in terminal)
- **Validation**: Zod v4 (schema-first tool definitions)
- **Concurrency**: AsyncIterator + Promise.race patterns

## Conversation Loop (Agent's Heartbeat)

### Basic Loop Structure

```typescript
async function* conversationLoop(
  deps: QueryDeps,
  initialMessages: Message[]
): AsyncGenerator<QueryEvent, void, void> {
  
  let messages = [...initialMessages];
  let turnCount = 0;
  const MAX_TURNS = deps.config.maxTurns ?? 100;
  
  while (turnCount < MAX_TURNS) {
    // 1. Check context budget
    const budget = calculateTokenBudget(messages, deps.context);
    if (budget.remaining < deps.config.minTokenReserve) {
      yield { type: 'context_full', budget };
      await compressContext(messages, deps);
    }
    
    // 2. Call LLM with tool schemas
    const response = await deps.llm.generate({
      messages,
      tools: deps.toolRegistry.getSchemas(),
      systemPrompt: deps.config.systemPrompt
    });
    
    yield { type: 'assistant_message', content: response };
    
    // 3. Handle tool calls
    if (response.toolCalls?.length > 0) {
      const results = await executeToolsWithPermissions(
        response.toolCalls,
        deps
      );
      
      for (const result of results) {
        yield { type: 'tool_result', result };
        messages.push({ role: 'tool', ...result });
      }
    }
    
    // 4. Check termination
    if (response.stopReason === 'end_turn' || !response.toolCalls) {
      yield { type: 'query_complete', reason: 'natural_stop' };
      break;
    }
    
    turnCount++;
  }
  
  if (turnCount >= MAX_TURNS) {
    yield { type: 'query_complete', reason: 'max_turns_exceeded' };
  }
}
```

### Five Core Yield Events

```typescript
type QueryEvent = 
  | { type: 'assistant_message'; content: string; thinking?: string }
  | { type: 'tool_call'; tool: string; input: unknown; approved: boolean }
  | { type: 'tool_result'; success: boolean; output: unknown }
  | { type: 'context_full'; budget: TokenBudget }
  | { type: 'query_complete'; reason: TerminationReason };

type TerminationReason =
  | 'natural_stop'        // Agent said "done"
  | 'max_turns_exceeded'  // Hit turn limit
  | 'user_interrupt'      // Ctrl+C
  | 'error_threshold'     // Too many tool failures
  | 'token_exhausted'     // Context cannot compress further
  | 'permission_denied'   // Critical tool blocked
  | 'safety_triggered'    // Harmful action detected
  | 'timeout'             // Wall-clock limit
  | 'explicit_stop'       // stopQuery() called
  | 'parent_terminated';  // Sub-agent's parent stopped
```

## Tool System (Agent's Hands)

### Tool Protocol

```typescript
interface Tool<Input = unknown, Output = unknown, Params = void> {
  name: string;                    // e.g., "bash_execute"
  description: string;             // For LLM tool selection
  inputSchema: z.ZodType<Input>;   // Zod validation
  outputSchema: z.ZodType<Output>; // Response shape
  
  // Execution
  execute: (input: Input, params: Params) => Promise<Output>;
  
  // Safety metadata
  readOnly: boolean;               // No side effects
  destructive: boolean;            // Can delete/modify data
  requiresApproval: boolean;       // Force permission gate
  concurrencySafe: boolean;        // Can run in parallel
  
  // Categorization
  category: ToolCategory;          // file | shell | memory | ...
  tags?: string[];
}

type ToolCategory =
  | 'file'      // read_file, write_file, list_directory
  | 'shell'     // bash_execute, run_command
  | 'memory'    // store_memory, query_memory
  | 'web'       // fetch_url, search
  | 'mcp'       // MCP server tools
  | 'meta'      // fork_agent, use_skill
  | 'planning'  // create_plan, update_plan
  | 'config'    // set_flag, load_skill
  | 'search'    // grep, find_in_codebase
  | 'git'       // commit, diff, branch
  | 'chat'      // ask_user, send_notification
  | 'system';   // get_env, check_platform
```

### Fault-Safe Tool Factory

```typescript
function buildTool<I, O, P = void>(
  spec: ToolSpec<I, O, P>
): Tool<I, O, P> {
  
  return {
    ...spec,
    
    // Wrapped executor with safety layers
    execute: async (input: I, params: P): Promise<O> => {
      // 1. Input validation
      const validatedInput = spec.inputSchema.parse(input);
      
      // 2. Pre-execution hook
      await spec.hooks?.beforeExecute?.(validatedInput, params);
      
      // 3. Timeout wrapper
      const timeoutMs = spec.timeout ?? 30_000;
      const result = await Promise.race([
        spec.execute(validatedInput, params),
        new Promise<never>((_, reject) =>
          setTimeout(() => reject(new Error('Tool timeout')), timeoutMs)
        )
      ]);
      
      // 4. Output validation
      const validatedOutput = spec.outputSchema.parse(result);
      
      // 5. Post-execution hook
      await spec.hooks?.afterExecute?.(validatedOutput);
      
      return validatedOutput;
    }
  };
}
```

### Example: File System Tool

```typescript
const readFileTool = buildTool({
  name: 'read_file',
  description: 'Read contents of a file',
  category: 'file',
  readOnly: true,
  destructive: false,
  concurrencySafe: true,
  
  inputSchema: z.object({
    path: z.string(),
    encoding: z.enum(['utf8', 'binary']).default('utf8')
  }),
  
  outputSchema: z.object({
    content: z.string(),
    size: z.number(),
    mtime: z.date()
  }),
  
  execute: async ({ path, encoding }) => {
    const content = await fs.readFile(path, encoding);
    const stats = await fs.stat(path);
    
    return {
      content,
      size: stats.size,
      mtime: stats.mtime
    };
  }
});
```

### Concurrent Tool Execution

```typescript
async function executeToolsWithConcurrency(
  toolCalls: ToolCall[],
  deps: QueryDeps
): Promise<ToolResult[]> {
  
  // Partition by concurrency safety
  const partitions = partitionTools(toolCalls, deps.toolRegistry);
  
  const results: ToolResult[] = [];
  
  for (const partition of partitions) {
    // Safe tools run in parallel
    if (partition.concurrencySafe) {
      const partitionResults = await Promise.all(
        partition.calls.map(call => 
          executeSingleTool(call, deps)
        )
      );
      results.push(...partitionResults);
    } 
    // Unsafe tools run sequentially
    else {
      for (const call of partition.calls) {
        const result = await executeSingleTool(call, deps);
        results.push(result);
      }
    }
  }
  
  return results;
}

function partitionTools(
  toolCalls: ToolCall[],
  registry: ToolRegistry
): ToolPartition[] {
  const partitions: ToolPartition[] = [];
  let currentPartition: ToolPartition | null = null;
  
  for (const call of toolCalls) {
    const tool = registry.get(call.name);
    
    // Start new partition if concurrency mode changes
    if (!currentPartition || 
        currentPartition.concurrencySafe !== tool.concurrencySafe) {
      currentPartition = {
        concurrencySafe: tool.concurrencySafe,
        calls: []
      };
      partitions.push(currentPartition);
    }
    
    currentPartition.calls.push(call);
  }
  
  return partitions;
}
```

## Permission Pipeline (Agent's Guardrails)

### 4-Stage Authorization Flow

```typescript
async function checkToolPermission(
  toolCall: ToolCall,
  deps: QueryDeps
): Promise<PermissionResult> {
  
  // Stage 1: Policy Check (instant)
  const policyResult = await deps.permissionManager.checkPolicy(toolCall);
  if (policyResult === 'ALWAYS_DENY') {
    return { approved: false, reason: 'policy_block' };
  }
  if (policyResult === 'ALWAYS_ALLOW') {
    return { approved: true, reason: 'policy_auto_approve' };
  }
  
  // Stage 2: Rule Matching (< 1ms)
  const ruleMatch = deps.permissionManager.matchRules(toolCall);
  if (ruleMatch) {
    return {
      approved: ruleMatch.action === 'allow',
      reason: 'rule_match',
      rule: ruleMatch.pattern
    };
  }
  
  // Stage 3: Speculative Classifier (< 2s)
  const classification = await Promise.race([
    deps.classifier.predictSafety(toolCall),
    new Promise<'unknown'>((resolve) => 
      setTimeout(() => resolve('unknown'), 2000)
    )
  ]);
  
  if (classification === 'safe') {
    return { approved: true, reason: 'classifier_safe' };
  }
  if (classification === 'harmful') {
    return { approved: false, reason: 'classifier_block' };
  }
  
  // Stage 4: User Prompt (blocking)
  const userDecision = await deps.ui.promptUser({
    tool: toolCall.name,
    input: toolCall.input,
    context: deps.context.recent()
  });
  
  // Store decision as rule for future
  if (userDecision.remember) {
    deps.permissionManager.addRule({
      pattern: userDecision.pattern,
      action: userDecision.approved ? 'allow' : 'deny'
    });
  }
  
  return {
    approved: userDecision.approved,
    reason: 'user_decision'
  };
}
```

### Permission Modes Spectrum

```typescript
type PermissionMode =
  | 'always_ask'      // Every tool requires approval
  | 'ask_dangerous'   // Only destructive tools prompt
  | 'auto_safe'       // Auto-approve readOnly tools
  | 'trust_classifier'// Use ML classifier, rarely prompt
  | 'autonomous';     // No prompts (production agents)

const permissionConfig = {
  mode: 'ask_dangerous' as PermissionMode,
  
  // Custom rules (Bash-style patterns)
  rules: [
    { pattern: 'bash_execute rm -rf *', action: 'deny' },
    { pattern: 'read_file ~/.ssh/*', action: 'allow' },
    { pattern: 'write_file *.test.ts', action: 'allow' }
  ],
  
  // Override per tool
  toolOverrides: {
    'bash_execute': 'always_ask',
    'read_file': 'auto_safe'
  }
};
```

## Context Management (Agent's Working Memory)

### Token Budget Calculation

```typescript
interface TokenBudget {
  total: number;           // Model context window (e.g., 200k)
  reserved: number;        // System prompt + tool schemas
  used: number;            // Current message history
  remaining: number;       // Available for new turns
  compressionThreshold: number; // Trigger compression at %
}

function calculateBudget(
  messages: Message[],
  config: ContextConfig
): TokenBudget {
  const total = config.modelContextWindow;
  const reserved = 
    estimateTokens(config.systemPrompt) +
    estimateTokens(JSON.stringify(config.toolSchemas)) +
    config.safetyMargin;
  
  const used = messages.reduce(
    (sum, msg) => sum + estimateTokens(msg.content),
    0
  );
  
  const remaining = total - reserved - used;
  const compressionThreshold = total * config.compressionTrigger;
  
  return {
    total,
    reserved,
    used,
    remaining,
    compressionThreshold
  };
}

function estimateTokens(text: string): number {
  // Rough approximation: 1 token ≈ 4 characters
  return Math.ceil(text.length / 4);
}
```

### 4-Level Progressive Compression

```typescript
async function compressContext(
  messages: Message[],
  deps: QueryDeps
): Promise<Message[]> {
  
  let compressed = [...messages];
  const budget = calculateBudget(compressed, deps.config);
  
  // Level 1: Snip - Remove middle chunks of long messages
  if (budget.remaining < budget.total * 0.2) {
    compressed = await snipLongMessages(compressed, {
      maxMessageLength: 2000,
      keepFirstChars: 500,
      keepLastChars: 500
    });
  }
  
  // Level 2: MicroCompact - Summarize old messages
  if (budget.remaining < budget.total * 0.15) {
    compressed = await microCompact(compressed, {
      targetReduction: 0.3,
      preserveRecent: 10,
      summaryModel: 'fast-summary-model'
    });
  }
  
  // Level 3: Collapse - Merge sequential tool results
  if (budget.remaining < budget.total * 0.1) {
    compressed = collapseToolResults(compressed, {
      maxSequentialTools: 3,
      keepSummary: true
    });
  }
  
  // Level 4: AutoCompact - Aggressive LLM summarization
  if (budget.remaining < budget.total * 0.05) {
    compressed = await autoCompact(compressed, {
      targetTokens: budget.total * 0.3,
      systemPrompt: deps.config.compactionPrompt
    });
  }
  
  return compressed;
}

function snipLongMessages(
  messages: Message[],
  opts: SnipOptions
): Message[] {
  return messages.map(msg => {
    if (msg.content.length <= opts.maxMessageLength) {
      return msg;
    }
    
    const first = msg.content.slice(0, opts.keepFirstChars);
    const last = msg.content.slice(-opts.keepLastChars);
    const omitted = msg.content.length - first.length - last.length;
    
    return {
      ...msg,
      content: `${first}\n\n[... ${omitted} chars omitted ...]\n\n${last}`
    };
  });
}
```

### Circuit Breaker Pattern

```typescript
class ContextCircuitBreaker {
  private state: 'closed' | 'open' | 'half_open' = 'closed';
  private failureCount = 0;
  private lastFailureTime = 0;
  
  constructor(
    private threshold = 3,
    private resetTimeout = 60_000
  ) {}
  
  async executeWithBreaker<T>(
    operation: () => Promise<T>
  ): Promise<T> {
    
    // Open state: reject fast
    if (this.state === 'open') {
      if (Date.now() - this.lastFailureTime > this.resetTimeout) {
        this.state = 'half_open';
      } else {
        throw new Error('Circuit breaker OPEN: context system unavailable');
      }
    }
    
    try {
      const result = await operation();
      
      // Success: reset
      if (this.state === 'half_open') {
        this.state = 'closed';
        this.failureCount = 0;
      }
      
      return result;
      
    } catch (error) {
      this.failureCount++;
      this.lastFailureTime = Date.now();
      
      if (this.failureCount >= this.threshold) {
        this.state = 'open';
      }
      
      throw error;
    }
  }
}

// Usage in context manager
const breaker = new ContextCircuitBreaker();

await breaker.executeWithBreaker(async () => {
  await compressContext(messages, deps);
});
```

## Memory System (Agent's Long-Term Memory)

### 4 Memory Types

```typescript
type Memory =
  | UserPreferenceMemory   // "User prefers tabs over spaces"
  | TaskContextMemory      // "Working on auth refactor"
  | WorldKnowledgeMemory   // "This repo uses Bun"
  | InteractionHistoryMemory; // "Failed bash_execute 3 times"

interface MemoryEntry {
  id: string;
  type: Memory['type'];
  content: string;
  importance: 1 | 2 | 3 | 4 | 5; // 5 = critical
  created: Date;
  lastAccessed: Date;
  accessCount: number;
  tags: string[];
  source: 'user' | 'agent' | 'system';
}

class MemoryManager {
  private memories = new Map<string, MemoryEntry>();
  
  async store(memory: Omit<MemoryEntry, 'id' | 'created'>): Promise<string> {
    // Only store non-derivable information
    if (await this.isDerivable(memory.content)) {
      return 'skipped:derivable';
    }
    
    const id = crypto.randomUUID();
    const entry: MemoryEntry = {
      ...memory,
      id,
      created: new Date(),
      lastAccessed: new Date(),
      accessCount: 0
    };
    
    this.memories.set(id, entry);
    await this.persistToFile(entry);
    
    return id;
  }
  
  async query(query: string, limit = 5): Promise<MemoryEntry[]> {
    // Simple vector search (production uses embeddings)
    const scored = Array.from(this.memories.values())
      .map(mem => ({
        memory: mem,
        score: this.relevanceScore(query, mem)
      }))
      .sort((a, b) => b.score - a.score)
      .slice(0, limit);
    
    // Update access metadata
    for (const { memory } of scored) {
      memory.lastAccessed = new Date();
      memory.accessCount++;
    }
    
    return scored.map(s => s.memory);
  }
  
  private async isDerivable(content: string): Promise<boolean> {
    // Don't store: file contents, public API docs, timestamps
    const derivablePatterns = [
      /^File.*contains:/,
      /^The documentation says/,
      /^Current time is/
    ];
    
    return derivablePatterns.some(pattern => pattern.test(content));
  }
  
  private relevanceScore(query: string, memory: MemoryEntry): number {
    const textSimilarity = this.cosineSimilarity(query, memory.content);
    const importanceBoost = memory.importance / 5;
    const recencyBoost = this.recencyScore(memory.lastAccessed);
    
    return textSimilarity * 0.6 + importanceBoost * 0.3 + recencyBoost * 0.1;
  }
}
```

### MEMORY.md Index Pattern

```markdown
<!-- MEMORY.md - Auto-generated memory index -->

# Agent Memory Index

## User Preferences (5 entries)
- Uses TypeScript strict mode [importance: 4]
- Prefers functional over OOP [importance: 3]
- Always wants tests for new features [importance: 5]

## Task Context (3 entries)
- Refactoring auth system to use JWT [importance: 5, active]
- Planning migration to Bun runtime [importance: 4]

## World Knowledge (12 entries)
- This codebase: Bun + React + Zod architecture [importance: 4]
- Database: PostgreSQL 15 with Drizzle ORM [importance: 3]

## Interaction History (recent 20)
- bash_execute failed 3x due to missing env vars [importance: 2]
- Successfully created 15 test files [importance: 1]
```

### Fork Memory Inheritance

```typescript
async function forkAgentWithMemory(
  parentAgent: Agent,
  task: string
): Promise<Agent> {
  
  const childAgent = await createAgent({
    systemPrompt: `Subtask: ${task}\n\nParent context: You are a forked agent.`,
  });
  
  // Inherit parent's memory
  const inheritedMemories = parentAgent.memory.query('', Infinity)
    .filter(mem => 
      // Only inherit high-importance and task-relevant
      mem.importance >= 3 || mem.type === 'task_context'
    );
  
  for (const mem of inheritedMemories) {
    await childAgent.memory.store({
      ...mem,
      source: 'parent_fork',
      tags: [...mem.tags, 'inherited']
    });
  }
  
  // Child's new memories merge back to parent on completion
  childAgent.onComplete(async () => {
    const childMemories = childAgent.memory.query('', Infinity)
      .filter(mem => mem.source === 'agent'); // Only agent-generated
    
    for (const mem of childMemories) {
      await parentAgent.memory.store(mem);
    }
  });
  
  return childAgent;
}
```

## Hook System (Agent's Lifecycle Extensions)

### 5 Hook Types

```typescript
type Hook =
  | BeforeToolHook      // Intercept before tool execution
  | AfterToolHook       // Post-process tool results
  | BeforeMessageHook   // Modify messages before LLM
  | AfterMessageHook    // Process LLM responses
  | ContextUpdateHook;  // React to context changes

interface HookRegistration {
  type: Hook['type'];
  priority: number;     // 1-1000, higher runs first
  handler: HookHandler;
  enabled: boolean;
  conditions?: HookCondition[];
}

type HookHandler = (
  event: HookEvent,
  context: HookContext
) => Promise<HookResult>;

type HookResult = 
  | { action: 'continue' }
  | { action: 'modify'; data: unknown }
  | { action: 'block'; reason: string };
```

### Example: Logging Hook

```typescript
const loggingHook: HookRegistration = {
  type: 'after_tool',
  priority: 100,
  enabled: true,
  
  handler: async (event, context) => {
    const { tool, input, output, duration } = event;
    
    await fs.appendFile('.claude/tool-audit.log', JSON.stringify({
      timestamp: new Date().toISOString(),
      tool: tool.name,
      input,
      output,
      duration,
      success: !output.error,
      user: context.user
    }) + '\n');
    
    return { action: 'continue' };
  }
};

deps.hooks.register(loggingHook);
```

### Example: Safety Hook

```typescript
const sanitizeSecretsHook: HookRegistration = {
  type: 'before_message',
  priority: 900, // High priority
  enabled: true,
  
  conditions: [{
    field: 'message.role',
    operator: 'equals',
    value: 'assistant'
  }],
  
  handler: async (event, context) => {
    const { message } = event;
    
    // Redact secrets before sending to LLM
    const secretPatterns = [
      /sk-[a-zA-Z0-9]{48}/g,         // OpenAI keys
      /ghp_[a-zA-Z0-9]{36}/g,        // GitHub tokens
      /Bearer [a-zA-Z0-9_-]+/g       // Bearer tokens
    ];
    
    let sanitized = message.content;
    for (const pattern of secretPatterns) {
      sanitized = sanitized.replace(pattern, '[REDACTED]');
    }
    
    if (sanitized !== message.content) {
      return {
        action: 'modify',
        data: { ...message, content: sanitized }
      };
    }
    
    return { action: 'continue' };
  }
};
```

## Sub-Agent & Fork Patterns

### Fork with Context Inheritance

```typescript
async function forkAgent(
  parentDeps: QueryDeps,
  task: string,
  options: ForkOptions = {}
): Promise<Agent> {
  
  // 1. Clone context (byte-level copy)
  const childContext = {
    messages: [...parentDeps.context.messages],
    budget: { ...parentDeps.context.budget },
    metadata: { ...parentDeps.context.metadata }
  };
  
  // 2. Inherit tools (unless overridden)
  const childTools = options.tools ?? parentDeps.toolRegistry.clone();
  
  // 3. Inherit memory (filtered by importance)
  const childMemory = await parentDeps.memory.fork({
    importanceThreshold: options.memoryImportanceThreshold ?? 3
  });
  
  // 4. Create child deps
  const childDeps: QueryDeps = {
    ...parentDeps,
    context: childContext,
    toolRegistry: childTools,
    memory: childMemory,
    config: {
      ...parentDeps.config,
      systemPrompt: `${parentDeps.config.systemPrompt}\n\n---\n\nSubtask: ${task}`,
      maxTurns: options.maxTurns ?? 20
    }
  };
  
  // 5. Start child conversation loop
  const childAgent = conversationLoop(
    childDeps,
    childContext.messages
  );
  
  // 6. Setup parent-child communication
  const messageBridge = createParentChildBridge(parentDeps, childDeps);
  
  return {
    generator: childAgent,
    send: (msg: string) => messageBridge.sendToChild(msg),
    terminate: () => messageBridge.terminateChild(),
    waitForCompletion: () => messageBridge.waitForChild()
  };
}

// Prevent infinite fork recursion
const MAX_FORK_DEPTH = 5;

function checkForkDepth(deps: QueryDeps): void {
  const depth = deps.context.metadata.forkDepth ?? 0;
  if (depth >= MAX_FORK_DEPTH) {
    throw new Error(`Fork depth limit exceeded (${MAX_FORK_DEPTH})`);
  }
  deps.context.metadata.forkDepth = depth + 1;
}
```

### Coordinator Pattern (Multi-Agent Orchestration)

```typescript
interface CoordinatorConfig {
  workers: WorkerAgent[];
  strategy: 'sequential' | 'parallel' | 'priority';
  timeout?: number;
}

class Coordinator {
  private workers: Map<string, WorkerAgent>;
  
  constructor(private config: CoordinatorConfig) {
    this.workers = new Map(
      config.workers.map(w => [w.id, w])
    );
  }
  
  async delegate(
    task: Task,
    deps: QueryDeps
  ): Promise<TaskResult> {
    
    // Coordinator only plans, never executes tools directly
    const plan = await this.createPlan(task, deps);
    
    if (this.config.strategy === 'sequential') {
      return await this.executeSequential(plan, deps);
    } else if (this.config.strategy === 'parallel') {
      return await this.executeParallel(plan, deps);
    } else {
      return await this.executePriority(plan, deps);
    }
  }
  
  private async executeSequential(
    plan: ExecutionPlan,
    deps: QueryDeps
  ): Promise<TaskResult> {
    
    const results: StepResult[] = [];
    
    for (const step of plan.steps) {
      const worker = this.selectWorker(step);
      
      const result = await worker.execute({
        task: step.task,
        context: {
          previousResults: results,
          parentDeps: deps
        }
      });
      
      results.push(result);
      
      // Early exit if step failed and not optional
      if (!result.success && !step.optional) {
        return {
          success: false,
          completedSteps: results,
          failedAt: step.id
        };
      }
    }
    
    return {
      success: true,
      completedSt
