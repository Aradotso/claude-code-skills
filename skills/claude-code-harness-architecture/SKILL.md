---
name: claude-code-harness-architecture
description: Expert guidance on AI Agent Harness architecture based on the comprehensive Claude Code analysis book
triggers:
  - "how do I build an agent harness"
  - "explain agent conversation loop architecture"
  - "show me tool permission pipeline design"
  - "how does context compression work in agents"
  - "implement agent memory system"
  - "design multi-agent coordinator pattern"
  - "set up MCP integration for agents"
  - "build fork-based sub-agent system"
---

# Claude Code Harness Architecture

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

This skill provides deep architectural knowledge from the 420,000-word book "Decoding Agent Harness" (御舆:解码 Agent Harness), a comprehensive analysis of Claude Code's internal architecture. Use this to design, build, and debug production-grade AI Agent systems.

## What This Covers

The book analyzes the complete Agent Harness architecture through 15 chapters:
- **Conversation Loop**: Async generator-based dialog control
- **Tool System**: 45+ tools, concurrent execution, safety protocols
- **Permission Pipeline**: 4-stage access control with speculative classification
- **Context Management**: 4-level progressive compression with circuit breakers
- **Memory System**: 4 types of closed-form memory, fork inheritance
- **Hook System**: 26 lifecycle events, 5 hook types
- **Sub-Agents**: Fork mode with byte-level context inheritance
- **MCP Integration**: 8 transport protocols, bridge architecture
- **Skills & Plugins**: 11 core skills, frontmatter-based loading
- **Streaming Architecture**: Performance optimization patterns

## Core Architecture Patterns

### 1. Conversation Loop (The Heartbeat)

The main loop is an async generator with 5 yield event types:

```typescript
async function* conversationLoop(
  deps: QueryDeps
): AsyncGenerator<ConversationEvent> {
  while (true) {
    // Yield status updates
    yield { type: 'thinking', phase: 'analyzing' };
    
    // Stream LLM response
    const response = await streamLLMCompletion(deps);
    yield { type: 'text', content: response.delta };
    
    // Execute tool calls
    if (response.toolUse) {
      yield { type: 'tool_call', tool: response.toolUse.name };
      const result = await executeTool(response.toolUse, deps);
      yield { type: 'tool_result', data: result };
    }
    
    // Check termination (10 types)
    if (shouldTerminate(response, deps)) {
      yield { type: 'done', reason: response.stopReason };
      break;
    }
  }
}
```

**10 Termination Reasons**:
- `end_turn`: Natural completion
- `max_tokens`: Context limit reached
- `stop_sequence`: Explicit stop marker
- `tool_use`: Waiting for tool approval
- `user_interrupt`: Manual cancellation
- `error`: Execution failure
- `timeout`: Time limit exceeded
- `budget_exceeded`: Token budget exhausted
- `recursion_limit`: Max fork depth reached
- `safety_violation`: Permission denied

### 2. Tool System Architecture

**Tool Protocol** (5 elements):

```typescript
interface Tool<I = unknown, O = unknown, P = unknown> {
  // 1. Schema: Zod v4 for validation
  input: z.ZodType<I>;
  
  // 2. Metadata
  name: string;
  description: string;
  
  // 3. Parameters (optional context)
  parameters?: P;
  
  // 4. Execute: Core logic
  execute: (input: I, params: P, deps: QueryDeps) => Promise<O>;
  
  // 5. Attributes
  readOnly: boolean;
  destructive: boolean;
  concurrencySafe: boolean;
}
```

**Fault-Safe Tool Factory**:

```typescript
function buildTool<I, O, P>(spec: ToolSpec<I, O, P>): Tool<I, O, P> {
  return {
    ...spec,
    execute: async (input, params, deps) => {
      try {
        // Validate input
        const validated = spec.input.parse(input);
        
        // Execute with timeout
        const result = await Promise.race([
          spec.execute(validated, params, deps),
          new Promise((_, reject) => 
            setTimeout(() => reject('timeout'), deps.timeout)
          )
        ]);
        
        return result as O;
      } catch (error) {
        // Return structured error, never throw
        return {
          success: false,
          error: error.message,
          recovery: spec.errorRecovery?.(error)
        } as O;
      }
    }
  };
}
```

**12 Tool Categories** (from Appendix B):
- File Operations: `read_file`, `write_file`, `search_files`
- Shell: `execute_command`, `bash_session`
- Browser: `navigate`, `screenshot`, `extract`
- Git: `commit`, `diff`, `log`
- Search: `web_search`, `codebase_search`
- Memory: `remember`, `recall`, `forget`
- Agent: `fork_agent`, `call_coordinator`
- MCP: `mcp_call_tool`, `mcp_list_resources`
- Plan: `create_plan`, `update_plan_step`
- Config: `get_setting`, `update_setting`
- Debug: `inspect_context`, `trace_tool_call`
- System: `sleep`, `notify`, `request_permission`

### 3. Permission Pipeline (4 Stages)

```typescript
type PermissionMode = 
  | 'auto'           // No approval needed
  | 'notify'         // Show notification, auto-proceed
  | 'confirm'        // Require user approval
  | 'reject'         // Always deny
  | 'interactive';   // Progressive disclosure

async function permissionPipeline(
  toolCall: ToolCall,
  deps: QueryDeps
): Promise<PermissionResult> {
  // Stage 1: Rule Matching (bash-style patterns)
  const mode = matchPermissionRule(toolCall.name, deps.config.permissions);
  
  if (mode === 'auto') return { approved: true };
  if (mode === 'reject') return { approved: false, reason: 'policy' };
  
  // Stage 2: Speculative Classification (2s timeout)
  const classification = await Promise.race([
    classifyToolIntent(toolCall, deps),
    Promise.resolve({ risk: 'unknown', confidence: 0 })
  ]);
  
  if (classification.risk === 'low' && classification.confidence > 0.9) {
    return { approved: true, source: 'classifier' };
  }
  
  // Stage 3: User Prompt
  if (mode === 'confirm' || mode === 'interactive') {
    const response = await deps.ui.promptUser({
      tool: toolCall.name,
      args: toolCall.arguments,
      risk: classification.risk,
      preview: generatePreview(toolCall)
    });
    
    return { approved: response.approved, memorize: response.remember };
  }
  
  // Stage 4: Fallback (default deny)
  return { approved: false, reason: 'no_approval' };
}
```

**Rule Matching Examples**:

```yaml
permissions:
  - pattern: "read_*"
    mode: auto
  - pattern: "write_file:/tmp/**"
    mode: notify
  - pattern: "execute_command:rm *"
    mode: reject
  - pattern: "fork_agent:**"
    mode: confirm
    max_depth: 3
```

### 4. Context Management (Compression Strategies)

**Effective Window Formula**:

```
EffectiveWindow = MaxContext - (SystemPrompt + Tools + Config + Memory + OutputReserve)
```

**4-Level Progressive Compression**:

```typescript
async function manageContext(deps: QueryDeps): Promise<Message[]> {
  const budget = calculateBudget(deps);
  let messages = deps.conversation.messages;
  
  // Level 1: Snip (truncate old content)
  if (getTokenCount(messages) > budget.warning) {
    messages = snipOldMessages(messages, budget.target);
  }
  
  // Level 2: MicroCompact (compress code blocks)
  if (getTokenCount(messages) > budget.warning) {
    messages = await microCompact(messages, {
      maxCodeLength: 500,
      preserveErrors: true
    });
  }
  
  // Level 3: Collapse (summarize message pairs)
  if (getTokenCount(messages) > budget.critical) {
    messages = await collapseMessages(messages, {
      minPairAge: 10,
      maxCollapse: 0.5
    });
  }
  
  // Level 4: AutoCompact (LLM-based summary)
  if (getTokenCount(messages) > budget.critical) {
    messages = await autoCompact(messages, deps);
  }
  
  // Circuit Breaker: Hard truncate if all fails
  if (getTokenCount(messages) > budget.max) {
    messages = messages.slice(-budget.max);
    deps.metrics.recordCircuitBreak('context_overflow');
  }
  
  return messages;
}
```

**Circuit Breaker Pattern**:

```typescript
class ContextCircuitBreaker {
  private failures = 0;
  private state: 'closed' | 'open' | 'half_open' = 'closed';
  
  async execute<T>(fn: () => Promise<T>): Promise<T> {
    if (this.state === 'open') {
      throw new Error('Circuit breaker open');
    }
    
    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }
  
  private onFailure() {
    this.failures++;
    if (this.failures >= 3) {
      this.state = 'open';
      setTimeout(() => this.state = 'half_open', 30000);
    }
  }
  
  private onSuccess() {
    this.failures = 0;
    this.state = 'closed';
  }
}
```

### 5. Memory System (4 Types)

**Closed-Form Memory** (only store non-derivable information):

```typescript
interface Memory {
  type: 'fact' | 'preference' | 'context' | 'goal';
  content: string;
  source: 'user' | 'agent' | 'tool';
  timestamp: number;
  expiresAt?: number;
  confidence: number;
}

class MemorySystem {
  async remember(memory: Memory, deps: QueryDeps) {
    // Deduplicate: Don't store if derivable
    if (await this.isDerivable(memory, deps)) {
      return { stored: false, reason: 'derivable' };
    }
    
    // Store in MEMORY.md
    await deps.fs.appendFile('.claude/MEMORY.md', 
      `\n## ${memory.type} (${new Date(memory.timestamp).toISOString()})\n` +
      `${memory.content}\n` +
      `Source: ${memory.source} | Confidence: ${memory.confidence}\n`
    );
    
    // Update index
    await this.updateIndex(memory, deps);
  }
  
  async recall(query: string, deps: QueryDeps): Promise<Memory[]> {
    // Vector search in index
    const results = await deps.vectorDB.search(query, {
      type: ['fact', 'preference', 'context', 'goal'],
      minConfidence: 0.7,
      limit: 10
    });
    
    // Filter expired
    return results.filter(m => 
      !m.expiresAt || m.expiresAt > Date.now()
    );
  }
}
```

**Fork Memory Inheritance**:

```typescript
async function forkAgent(config: ForkConfig, deps: QueryDeps): Promise<Agent> {
  const childMemory = {
    // Inherit parent facts
    facts: deps.memory.getFacts(),
    
    // Inherit preferences (shallow copy)
    preferences: { ...deps.memory.getPreferences() },
    
    // New context scope
    context: [],
    
    // Inherit goals if specified
    goals: config.inheritGoals ? deps.memory.getGoals() : []
  };
  
  return createAgent({
    ...config,
    memory: childMemory,
    parent: deps.agentId,
    depth: deps.forkDepth + 1
  });
}
```

### 6. Sub-Agent & Coordinator Pattern

**Fork Mode** (byte-level context inheritance):

```typescript
interface ForkConfig {
  type: 'fork' | 'spawn' | 'clone';
  inheritContext: boolean;
  inheritMemory: boolean;
  inheritTools: boolean;
  inheritGoals: boolean;
  maxDepth: number;
}

async function forkAgent(
  config: ForkConfig,
  deps: QueryDeps
): Promise<{ agentId: string; channel: MessageChannel }> {
  // Recursion guard
  if (deps.forkDepth >= config.maxDepth) {
    throw new Error(`Max fork depth ${config.maxDepth} exceeded`);
  }
  
  // Byte-level context copy
  const childContext = config.inheritContext 
    ? structuredClone(deps.conversation)
    : { messages: [] };
  
  // Create child agent
  const child = await createAgent({
    ...config,
    context: childContext,
    memory: config.inheritMemory ? cloneMemory(deps.memory) : {},
    tools: config.inheritTools ? deps.tools : getDefaultTools(),
    parent: deps.agentId,
    depth: deps.forkDepth + 1
  });
  
  return {
    agentId: child.id,
    channel: child.messageChannel
  };
}
```

**Coordinator Pattern** (orchestration-only):

```typescript
class CoordinatorAgent {
  // Constraint: Coordinators never execute tools directly
  private readonly allowedTools = [
    'fork_agent',
    'send_message_to_agent',
    'wait_for_agent',
    'aggregate_results'
  ];
  
  async orchestrate(task: Task, deps: QueryDeps) {
    // Decompose task
    const subtasks = await this.decompose(task, deps);
    
    // Spawn workers
    const workers = await Promise.all(
      subtasks.map(st => forkAgent({
        type: 'spawn',
        inheritContext: false,
        inheritMemory: true,
        inheritTools: true,
        inheritGoals: false,
        maxDepth: deps.forkDepth + 1
      }, deps))
    );
    
    // Distribute work
    await Promise.all(
      workers.map((w, i) => this.sendTask(w, subtasks[i]))
    );
    
    // Aggregate results
    const results = await Promise.all(
      workers.map(w => this.waitForCompletion(w))
    );
    
    return this.synthesize(results, task);
  }
  
  // Double gate: tool filter + instruction constraint
  validateToolCall(toolName: string): boolean {
    return this.allowedTools.includes(toolName);
  }
}
```

**4 Addressing Modes**:

```typescript
type AgentAddress = 
  | { type: 'direct', id: string }           // UUID
  | { type: 'role', role: string }           // 'researcher', 'coder'
  | { type: 'capability', skills: string[] } // ['python', 'data_viz']
  | { type: 'broadcast', scope: 'all' | 'siblings' };

async function routeMessage(
  message: Message,
  address: AgentAddress,
  deps: QueryDeps
): Promise<void> {
  const targets = resolveAddress(address, deps);
  await Promise.all(
    targets.map(agent => agent.channel.postMessage(message))
  );
}
```

### 7. MCP Integration

**8 Transport Protocols** (from Ch12):
- `stdio`: Standard input/output
- `sse`: Server-Sent Events
- `ws`: WebSocket
- `http`: HTTP polling
- `ipc`: Inter-Process Communication
- `tcp`: Raw TCP socket
- `unix`: Unix domain socket
- `embedded`: In-process

**5-State Connection Management**:

```typescript
type MCPConnectionState = 
  | 'disconnected'
  | 'connecting'
  | 'connected'
  | 'reconnecting'
  | 'failed';

class MCPConnection {
  private state: MCPConnectionState = 'disconnected';
  private reconnectAttempts = 0;
  private maxReconnectAttempts = 5;
  
  async connect(config: MCPConfig): Promise<void> {
    this.state = 'connecting';
    
    try {
      const transport = createTransport(config.protocol, config);
      await transport.connect();
      
      // Handshake
      const capabilities = await this.handshake(transport);
      
      this.state = 'connected';
      this.reconnectAttempts = 0;
      
      return { transport, capabilities };
    } catch (error) {
      await this.handleConnectionError(error, config);
    }
  }
  
  private async handleConnectionError(
    error: Error,
    config: MCPConfig
  ): Promise<void> {
    if (this.reconnectAttempts < this.maxReconnectAttempts) {
      this.state = 'reconnecting';
      this.reconnectAttempts++;
      
      const delay = Math.min(1000 * 2 ** this.reconnectAttempts, 30000);
      await sleep(delay);
      
      return this.connect(config);
    } else {
      this.state = 'failed';
      throw error;
    }
  }
}
```

**3-Segment Tool Naming**:

```
mcp://<server>/<category>/<tool>
```

Example:
```typescript
const toolName = 'mcp://github/repo/create_issue';
const [protocol, server, category, tool] = toolName.split('/');
```

**Bridge Pattern** (bidirectional communication):

```typescript
class MCPBridge {
  // Agent → MCP Server
  async callTool(
    toolName: string,
    args: unknown,
    deps: QueryDeps
  ): Promise<unknown> {
    const [server, category, tool] = parseMCPToolName(toolName);
    const connection = deps.mcpConnections.get(server);
    
    const result = await connection.request({
      method: 'tools/call',
      params: { name: `${category}/${tool}`, arguments: args }
    });
    
    return result;
  }
  
  // MCP Server → Agent (notifications)
  async handleNotification(
    notification: MCPNotification,
    deps: QueryDeps
  ): Promise<void> {
    switch (notification.method) {
      case 'notifications/resources/updated':
        await deps.memory.invalidateCache(notification.params.uri);
        break;
      case 'notifications/tools/list_changed':
        await deps.tools.refreshMCPTools(notification.params.server);
        break;
    }
  }
}
```

### 8. Hook System (26 Lifecycle Events)

**5 Hook Types**:

```typescript
type HookType = 
  | 'before'      // Pre-execution interception
  | 'after'       // Post-execution augmentation
  | 'transform'   // Data transformation
  | 'validate'    // Validation checkpoint
  | 'observe';    // Side-effect free monitoring

interface Hook {
  type: HookType;
  event: LifecycleEvent;
  priority: number; // 0-100, higher runs first
  execute: (context: HookContext) => Promise<HookResult>;
  conditions?: HookCondition[];
}
```

**26 Lifecycle Events** (partial list from Ch08):

```typescript
type LifecycleEvent = 
  // Conversation
  | 'conversation:start'
  | 'conversation:message'
  | 'conversation:end'
  
  // Tool execution
  | 'tool:before_call'
  | 'tool:after_call'
  | 'tool:error'
  
  // Context management
  | 'context:compress'
  | 'context:overflow'
  
  // Permission
  | 'permission:request'
  | 'permission:denied'
  
  // Memory
  | 'memory:store'
  | 'memory:recall'
  
  // Agent
  | 'agent:fork'
  | 'agent:terminate'
  
  // MCP
  | 'mcp:connect'
  | 'mcp:disconnect'
  
  // Plan
  | 'plan:create'
  | 'plan:step_complete'
  
  // System
  | 'system:error'
  | 'system:shutdown';
```

**Hook Registration**:

```typescript
async function registerHook(
  hook: Hook,
  deps: QueryDeps
): Promise<void> {
  // Security check: validate source
  if (!deps.config.allowExternalHooks && hook.source !== 'builtin') {
    throw new Error('External hooks disabled');
  }
  
  // Validate hook implementation
  await validateHookSchema(hook);
  
  // Register in priority order
  deps.hooks.register(hook.event, hook.priority, hook.execute);
}

async function executeHooks(
  event: LifecycleEvent,
  context: HookContext,
  deps: QueryDeps
): Promise<HookContext> {
  const hooks = deps.hooks.get(event).sort((a, b) => b.priority - a.priority);
  
  let currentContext = context;
  
  for (const hook of hooks) {
    // Check conditions
    if (hook.conditions && !evaluateConditions(hook.conditions, currentContext)) {
      continue;
    }
    
    // Execute with timeout
    try {
      const result = await Promise.race([
        hook.execute(currentContext),
        timeout(5000, 'Hook timeout')
      ]);
      
      // Transform context
      if (result.context) {
        currentContext = result.context;
      }
      
      // Halt propagation if requested
      if (result.halt) {
        break;
      }
    } catch (error) {
      deps.metrics.recordHookError(hook.event, error);
      // Continue executing other hooks
    }
  }
  
  return currentContext;
}
```

### 9. Skills & Plugins (Frontmatter-Based Loading)

**SKILL.md Frontmatter**:

```yaml
---
name: git-workflow-automation
version: 1.2.0
description: Automated git workflows with conventional commits
triggers:
  - "create a feature branch"
  - "commit with conventional format"
  - "prepare a release"
dependencies:
  tools: [execute_command, read_file, write_file]
  mcpServers: [github]
config:
  commitFormat: conventional
  branchPrefix: feature/
  autoSquash: true
parameters:
  mainBranch: ${param.mainBranch|main}
  remote: ${param.remote|origin}
  signCommits: ${param.signCommits|false}
---
```

**3-Level Parameter Replacement**:

```typescript
function resolveParameters(
  skillConfig: SkillConfig,
  deps: QueryDeps
): Record<string, unknown> {
  const resolved = {};
  
  for (const [key, template] of Object.entries(skillConfig.parameters)) {
    // Level 1: User-provided parameters
    if (deps.userParams[key] !== undefined) {
      resolved[key] = deps.userParams[key];
      continue;
    }
    
    // Level 2: Environment variables
    const envMatch = template.match(/\${env\.([^}|]+)(\|(.+))?}/);
    if (envMatch) {
      const [, envVar, , defaultValue] = envMatch;
      resolved[key] = process.env[envVar] ?? defaultValue;
      continue;
    }
    
    // Level 3: Default value
    const defaultMatch = template.match(/\${param\.[^}|]+\|(.+)}/);
    if (defaultMatch) {
      resolved[key] = defaultMatch[1];
    }
  }
  
  return resolved;
}
```

**Layered Loading**:

```typescript
async function loadSkills(deps: QueryDeps): Promise<Skill[]> {
  const skills: Skill[] = [];
  
  // Layer 1: Built-in skills
  const builtinPath = path.join(__dirname, '../skills');
  skills.push(...await loadSkillsFromDir(builtinPath));
  
  // Layer 2: User skills
  const userPath = path.join(deps.config.skillsDir);
  if (await exists(userPath)) {
    skills.push(...await loadSkillsFromDir(userPath));
  }
  
  // Layer 3: Project skills
  const projectPath = path.join(deps.workingDir, '.claude/skills');
  if (await exists(projectPath)) {
    skills.push(...await loadSkillsFromDir(projectPath));
  }
  
  // Deduplicate by name (later layers override)
  return deduplicateSkills(skills);
}
```

### 10. Performance Optimization

**Startup Optimization** (Ch13: 160ms → 65ms, -59%):

```typescript
// Before: Eager loading
async function initialize(deps: QueryDeps) {
  await loadAllTools();          // 45ms
  await loadAllSkills();         // 30ms
  await connectAllMCP();         // 50ms
  await loadMemory();            // 20ms
  await initializeHooks();       // 15ms
  // Total: 160ms
}

// After: Lazy + parallel loading
async function initialize(deps: QueryDeps) {
  // Parallel critical path
  await Promise.all([
    loadCoreTools(),             // 15ms (only essential tools)
    initializeConversation()     // 10ms
  ]);
  
  // Defer non-critical
  Promise.all([
    lazyLoadSkills(),            // Load on first trigger
    lazyConnectMCP(),            // Connect on first use
    backgroundLoadMemory()       // Index in background
  ]);
  
  // Total: 25ms to first interaction, 65ms fully ready
}
```

**Lazy Tool Loading**:

```typescript
class LazyToolRegistry {
  private loaded = new Set<string>();
  private loaders = new Map<string, () => Promise<Tool>>();
  
  register(name: string, loader: () => Promise<Tool>) {
    this.loaders.set(name, loader);
  }
  
  async get(name: string): Promise<Tool> {
    if (!this.loaded.has(name)) {
      const tool = await this.loaders.get(name)!();
      this.tools.set(name, tool);
      this.loaded.add(name);
    }
    return this.tools.get(name)!;
  }
}
```

## Configuration

**6-Layer Priority Chain** (Ch05):

1. CLI arguments (highest)
2. Environment variables
3. Project config (`.claude/config.json`)
4. User config (`~/.claude/config.json`)
5. Workspace config
6. Default config (lowest)

**Example Config**:

```json
{
  "model": "claude-3-7-sonnet-20250219",
  "maxTokens": 8192,
  "temperature": 0.7,
  
  "tools": {
    "enabled": ["read_file", "write_file", "execute_command"],
    "disabled": ["browser_*"],
    "concurrency": {
      "max": 5,
      "perCategory": 3
    }
  },
  
  "permissions": {
    "defaultMode": "confirm",
    "rules": [
      { "pattern": "read_*", "mode": "auto" },
      { "pattern": "execute_command:rm *", "mode": "reject" }
    ]
  },
  
  "context": {
    "maxTokens": 180000,
    "compressionThreshold": 0.8,
    "strategies": ["snip", "microCompact", "collapse", "autoCompact"]
  },
  
  "memory": {
    "enabled": true,
    "maxEntries": 1000,
    "ttl": 2592000000
  },
  
  "hooks": {
    "allowExternal": false,
    "timeout": 5000
  },
  
  "mcp": {
    "servers": {
      "github": {
        "command": "mcp-server-github",
        "args": ["--token", "${env.GITHUB_TOKEN}"],
        "protocol": "stdio"
      }
    }
  },
  
  "features": {
    "enablePlanMode": true,
    "enableSubAgents": true,
    "maxForkDepth": 3
  }
}
```

## Building Your Own Harness (Ch15)

**6-Step Implementation Roadmap**:

```typescript
// Step 1: Core conversation loop
async function* conversationLoop(deps: QueryDeps) {
  while (true) {
    const response = await streamLLM(deps);
    yield { type: 'text', content: response.delta };
    
    if (response.toolUse) {
      const result = await executeTool(response.toolUse, deps);
      yield { type: 'tool_result', data: result };
    }
    
    if (shouldTerminate(response)) break;
  }
}

// Step 2: Tool registry with validation
const tools = new ToolRegistry();
tools.register(buildTool({
  name: 'read_file',
  input: z.object({ path: z.string() }),
  execute: async (input) => await fs.readFile(input.path, 'utf-8')
}));

// Step 3: Permission pipeline
async function checkPermission(toolCall: ToolCall, deps: QueryDeps) {
  const mode = matchRule(toolCall.name, deps.config.permissions);
  if (mode === 'confirm') {
    return await deps.ui.promptUser(toolCall);
  }
  return mode === 'auto';
}

// Step 4: Context compression
async function compressContext(messages: Message[], budget: number) {
  let compressed = messages;
  if (getTokens(compressed) > budget) {
    compressed = snip(compressed, budget);
  }
  if (getTokens(compressed) > budget) {
    compressed = await autoCompact(compressed);
  }
  return compressed;
}

// Step 5: Memory system
class MemorySystem {
  async remember(fact: Memory) {
    if (!await this.isDerivable(fact)) {
      await this.store(fact);
    }
  }
  
  async recall(query: string) {
    return await this.vectorSearch(query);
  }
}

// Step 6: Observability
class Metrics
