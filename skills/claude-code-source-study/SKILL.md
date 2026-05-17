---
name: claude-code-source-study
description: Deep dive into Claude Code's source code to learn AI agent implementation patterns, system prompt engineering, and production-grade AI coding assistant architecture
triggers:
  - how does claude code work internally
  - analyze claude code source architecture
  - learn ai agent implementation from claude code
  - understand claude code's prompt engineering
  - study claude code tool system design
  - explore claude code multi-agent orchestration
  - show me claude code design patterns
  - explain claude code's context management
---

# Claude Code Source Study

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

This skill provides expertise in understanding and applying architectural patterns from Claude Code's source code — Anthropic's production-grade AI coding assistant. The project contains ~1,900 files covering system prompt engineering, multi-agent orchestration, tool systems, security, and terminal UI.

## What This Project Is

Claude Code Source Study (`luyao618/Claude-Code-Source-Study`) is a comprehensive 25-article source code analysis of Claude Code, covering:

- **Global Architecture**: Startup optimization, state management, module structure
- **AI Core**: System prompt engineering, conversation loops, context management, prompt caching
- **Tool & Agent Systems**: Tool builder patterns, bash execution safety, multi-agent coordination
- **Security & Engineering**: Permission systems, settings architecture, feature flags, error recovery
- **Terminal UI**: Custom Ink framework, design system, memory architecture

## Repository Structure

```
claude-code-source-study/
├── docs/
│   ├── 00-目录与阅读指引.md          # Reading guide
│   ├── 01-项目全景.md                # Project overview
│   ├── 02-启动优化.md                # Startup optimization
│   ├── 03-状态管理.md                # State management
│   ├── 04-System-Prompt-工程.md      # System prompt engineering
│   ├── 05-对话循环.md                # Conversation loop
│   ├── 06-上下文管理.md              # Context management
│   ├── 07-Prompt-Cache.md           # Prompt caching
│   ├── 08-Thinking-与推理控制.md     # Thinking & reasoning control
│   ├── 09-工具系统设计.md            # Tool system design
│   ├── 10-BashTool-深度剖析.md       # BashTool deep dive
│   ├── 11-命令系统.md                # Command system
│   ├── 12-Agent-系统.md              # Agent system
│   ├── 13-内置Agent设计模式.md        # Built-in agent patterns
│   ├── 14-任务系统.md                # Task system
│   ├── 15-MCP-协议实现.md            # MCP protocol implementation
│   ├── 16-权限系统.md                # Permission system
│   ├── 17-Settings-系统.md           # Settings system
│   ├── 18-Hooks系统.md               # Hooks system
│   ├── 19-Feature-Flag与编译期优化.md # Feature flags & compile-time optimization
│   ├── 20-API调用与错误恢复.md        # API calls & error recovery
│   ├── 21-Ink框架深度定制.md          # Ink framework customization
│   ├── 22-设计系统.md                # Design system
│   ├── 23-Memory系统.md              # Memory system
│   ├── 24-Skill-Plugin开发实战.md     # Skill/plugin development
│   └── 25-架构模式总结.md             # Architecture patterns summary
└── README.md
```

## Key Learning Paths

### Quick Start Path (7 articles)
For rapid global understanding:
1. Project Overview (01)
2. Startup Optimization (02)
3. State Management (03)
4. Conversation Loop (05)
5. Tool System Design (09)
6. Agent System (12)
7. Architecture Patterns Summary (25)

### AI Engineering Path (9 articles)
For deep AI architecture understanding:
1. Project Overview (01)
2. State Management (03)
3. System Prompt Engineering (04)
4. Conversation Loop (05)
5. Context Management (06)
6. Thinking & Reasoning Control (08)
7. Tool System Design (09)
8. Agent System (12)
9. Built-in Agent Patterns (13)

### Complete Path (25 articles)
Read sequentially for comprehensive understanding.

## Core Design Patterns

### 1. System Prompt Engineering

**Pattern**: Segmented construction with cache boundaries

```typescript
// System prompt built in segments for optimal caching
const systemPrompt = [
  // Static instructions (cacheable)
  baseInstructions,
  
  // Tool schemas (cacheable if tools unchanged)
  toolSchemas,
  
  // Dynamic context (not cached)
  currentProjectContext,
  userPreferences
].join('\n\n');

// Cache control headers
const cacheBreakpoints = {
  ephemeral: 'static_instructions_end',
  persistent: 'tool_schemas_end'
};
```

**Key Learnings**:
- Separate static from dynamic content
- Place frequently-changing content last
- Use explicit cache boundary markers
- Balance cache hit rate vs. context freshness

### 2. Conversation State Machine

**Pattern**: AsyncGenerator-driven conversation loop

```typescript
async function* conversationLoop(
  messages: Message[],
  config: ConversationConfig
): AsyncGenerator<ConversationState> {
  let state: ConversationState = { phase: 'thinking' };
  
  while (true) {
    yield state;
    
    switch (state.phase) {
      case 'thinking':
        const thinking = await generateThinking(messages);
        state = { phase: 'responding', thinking };
        break;
        
      case 'responding':
        const response = await generateResponse(messages, state.thinking);
        if (response.toolCalls) {
          state = { phase: 'tool_execution', toolCalls: response.toolCalls };
        } else {
          state = { phase: 'complete', response };
        }
        break;
        
      case 'tool_execution':
        const results = await executeTools(state.toolCalls);
        messages.push(...results);
        state = { phase: 'thinking' };
        break;
        
      case 'complete':
        return;
    }
  }
}
```

**Key Learnings**:
- AsyncGenerator provides natural state streaming
- Each yield enables UI updates
- Tool execution results feed back into conversation
- Clear state transitions prevent deadlocks

### 3. Tool Builder Pattern

**Pattern**: Fluent API for tool registration with conditional activation

```typescript
interface ToolBuilder {
  buildTool(name: string): {
    description: (desc: string) => ToolBuilder;
    parameters: (schema: JSONSchema) => ToolBuilder;
    handler: (fn: ToolHandler) => ToolBuilder;
    condition: (predicate: () => boolean) => ToolBuilder;
    permission: (level: PermissionLevel) => ToolBuilder;
    build: () => Tool;
  };
}

// Usage
const readFileTool = buildTool('read_file')
  .description('Read contents of a file')
  .parameters({
    type: 'object',
    properties: {
      path: { type: 'string', description: 'File path' }
    },
    required: ['path']
  })
  .handler(async ({ path }) => {
    return await fs.readFile(path, 'utf-8');
  })
  .condition(() => !isInRestrictedMode())
  .permission('read')
  .build();
```

**Key Learnings**:
- Three-layer registration: builder → registry → runtime
- Conditions evaluated at registration time
- Permissions checked at execution time
- Type-safe parameter schemas

### 4. Context Budget Management

**Pattern**: Token-aware context window with auto-compaction

```typescript
interface ContextManager {
  budget: {
    total: number;      // e.g., 200k tokens
    system: number;     // ~5k for system prompt
    tools: number;      // ~10k for tool schemas
    history: number;    // Remaining for conversation
  };
  
  async addMessage(msg: Message): Promise<void> {
    const tokens = await this.countTokens(msg);
    
    if (this.currentUsage + tokens > this.budget.history) {
      await this.compact();
    }
    
    this.messages.push(msg);
    this.currentUsage += tokens;
  }
  
  async compact(): Promise<void> {
    // Strategy 1: Remove old tool results (keep latest)
    this.messages = this.messages.filter((m, i) => {
      if (m.role === 'tool') {
        return i >= this.messages.length - 10;
      }
      return true;
    });
    
    // Strategy 2: Summarize middle conversation
    const [start, middle, end] = this.partition(this.messages);
    const summary = await this.summarize(middle);
    this.messages = [...start, summary, ...end];
    
    this.currentUsage = await this.countTokens(this.messages);
  }
}
```

**Key Learnings**:
- Pre-allocate token budget by category
- Prioritize recent context over old
- Multi-strategy compaction (remove, summarize, truncate)
- Always preserve system prompt and tool schemas

### 5. Multi-Agent Orchestration

**Pattern**: Context-isolated agent delegation

```typescript
interface Agent {
  name: string;
  systemPrompt: string;
  availableTools: Tool[];
  conversationLoop: ConversationLoop;
}

class AgentOrchestrator {
  private agents: Map<string, Agent> = new Map();
  
  async delegateTask(
    taskType: string,
    context: TaskContext,
    parentConversation: Message[]
  ): Promise<AgentResult> {
    const agent = this.selectAgent(taskType);
    
    // Create isolated context
    const agentContext = {
      goal: context.goal,
      relevantFiles: context.files,
      constraints: context.constraints,
      // Do NOT pass full parent conversation
    };
    
    const agentMessages = [
      { role: 'user', content: this.formatTaskPrompt(agentContext) }
    ];
    
    const result = await agent.conversationLoop(agentMessages);
    
    // Merge result back to parent
    return this.mergeResult(result, parentConversation);
  }
  
  selectAgent(taskType: string): Agent {
    const agentMap = {
      'explore': this.agents.get('explorer'),
      'plan': this.agents.get('planner'),
      'verify': this.agents.get('verifier'),
      'execute': this.agents.get('executor')
    };
    return agentMap[taskType] || this.agents.get('default');
  }
}
```

**Key Learnings**:
- Each agent has isolated conversation context
- Parent conversation not leaked to child agents
- Task-specific agent selection
- Results merged back, not entire conversation

### 6. Permission System

**Pattern**: Seven-mode permission model with decision pipeline

```typescript
type PermissionMode = 
  | 'auto'          // No prompts, auto-approve
  | 'confirm'       // Prompt for each action
  | 'reject'        // Auto-reject
  | 'readonly'      // Only allow read operations
  | 'custom'        // User-defined rules
  | 'trust_verified'// Auto-approve verified operations
  | 'sandbox';      // Execute in isolated environment

interface PermissionDecision {
  allowed: boolean;
  reason?: string;
  modified?: ToolCall; // Modified version with restricted params
}

class PermissionManager {
  async checkPermission(
    toolCall: ToolCall,
    context: ExecutionContext
  ): Promise<PermissionDecision> {
    // 7-step decision pipeline
    
    // 1. Global mode check
    if (this.mode === 'auto') return { allowed: true };
    if (this.mode === 'reject') return { allowed: false };
    
    // 2. Tool-level permission
    const toolPerm = this.getToolPermission(toolCall.name);
    if (toolPerm === 'blocked') {
      return { allowed: false, reason: 'Tool blocked by policy' };
    }
    
    // 3. Readonly mode check
    if (this.mode === 'readonly' && !toolPerm.isReadOnly) {
      return { allowed: false, reason: 'Write operation in readonly mode' };
    }
    
    // 4. Path restriction check
    if (!this.isPathAllowed(toolCall.arguments.path)) {
      return { allowed: false, reason: 'Path outside allowed directories' };
    }
    
    // 5. Resource limit check
    if (await this.exceedsResourceLimit(toolCall)) {
      return { allowed: false, reason: 'Resource limit exceeded' };
    }
    
    // 6. Custom rule evaluation
    for (const rule of this.customRules) {
      const decision = await rule.evaluate(toolCall, context);
      if (!decision.allowed) return decision;
    }
    
    // 7. User confirmation (if mode === 'confirm')
    if (this.mode === 'confirm') {
      const approved = await this.promptUser(toolCall);
      return { allowed: approved };
    }
    
    return { allowed: true };
  }
}
```

**Key Learnings**:
- Layered decision-making (global → tool → resource → custom)
- Fail-closed by default
- Each layer can short-circuit
- Support for parameter modification (e.g., restrict file paths)

### 7. Feature Flag with Dead Code Elimination

**Pattern**: Compile-time feature toggling

```typescript
// feature.ts
export function feature(flag: string): boolean {
  const COMPILE_TIME_FLAGS = {
    'mcp_support': process.env.BUILD_TARGET !== 'minimal',
    'analytics': process.env.BUILD_TARGET === 'enterprise',
    'cloud_sync': process.env.ENABLE_CLOUD === 'true'
  };
  
  return COMPILE_TIME_FLAGS[flag] ?? false;
}

// Usage (DCE-optimized)
if (feature('mcp_support')) {
  // This entire block removed if BUILD_TARGET=minimal
  import('./mcp-client').then(mcp => {
    mcp.initialize();
  });
}

// Build script
{
  "scripts": {
    "build:minimal": "BUILD_TARGET=minimal bun build",
    "build:full": "BUILD_TARGET=full bun build",
    "build:enterprise": "BUILD_TARGET=enterprise ENABLE_CLOUD=true bun build"
  }
}
```

**Key Learnings**:
- Single codebase, multiple build outputs
- Dead code eliminated at bundle time
- Feature flags as environment variables
- Conditional imports for chunk splitting

## Common Patterns & Idioms

### AsyncGenerator for Streaming State

```typescript
// Producer
async function* processWithProgress() {
  yield { status: 'starting' };
  
  const result = await heavyComputation();
  yield { status: 'processing', progress: 0.5 };
  
  await saveResult(result);
  yield { status: 'complete', data: result };
}

// Consumer
for await (const state of processWithProgress()) {
  updateUI(state);
}
```

### Store Pattern for React + Non-React

```typescript
class Store<T> {
  private state: T;
  private listeners = new Set<(state: T) => void>();
  
  getState(): T {
    return this.state;
  }
  
  setState(partial: Partial<T>): void {
    this.state = { ...this.state, ...partial };
    this.listeners.forEach(fn => fn(this.state));
  }
  
  subscribe(fn: (state: T) => void): () => void {
    this.listeners.add(fn);
    return () => this.listeners.delete(fn);
  }
}

// React hook
function useStore<T>(store: Store<T>): T {
  const [state, setState] = useState(store.getState());
  
  useEffect(() => {
    return store.subscribe(setState);
  }, [store]);
  
  return state;
}
```

### Retry with Exponential Backoff

```typescript
async function withRetry<T>(
  fn: () => Promise<T>,
  options: RetryOptions = {}
): Promise<T> {
  const { maxAttempts = 3, backoff = 'exponential', baseDelay = 1000 } = options;
  
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      return await fn();
    } catch (error) {
      if (attempt === maxAttempts) throw error;
      
      const delay = backoff === 'exponential' 
        ? baseDelay * Math.pow(2, attempt - 1)
        : baseDelay;
      
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
  
  throw new Error('Retry failed');
}
```

## Configuration Examples

### System Prompt Configuration

```typescript
// System prompt with cache boundaries
const systemPromptConfig = {
  segments: [
    {
      name: 'base_instructions',
      cacheable: true,
      cacheType: 'persistent',
      content: `You are Claude Code, an AI coding assistant...`
    },
    {
      name: 'tool_schemas',
      cacheable: true,
      cacheType: 'ephemeral',
      content: JSON.stringify(toolSchemas)
    },
    {
      name: 'project_context',
      cacheable: false,
      content: () => getCurrentProjectContext()
    }
  ]
};
```

### Agent Configuration

```typescript
// Multi-agent setup
const agentConfig = {
  agents: [
    {
      name: 'explorer',
      systemPrompt: 'You explore codebases and understand structure...',
      tools: ['read_file', 'list_directory', 'search_files'],
      maxTokens: 50000
    },
    {
      name: 'planner',
      systemPrompt: 'You create detailed implementation plans...',
      tools: ['read_file', 'analyze_dependencies'],
      maxTokens: 30000
    },
    {
      name: 'executor',
      systemPrompt: 'You write and modify code...',
      tools: ['read_file', 'write_file', 'bash'],
      maxTokens: 100000
    }
  ],
  orchestration: {
    delegation_threshold: 'complex_task',
    context_isolation: true,
    result_merge_strategy: 'summary'
  }
};
```

### Permission Configuration

```typescript
// Permission rules
const permissionConfig = {
  mode: 'confirm',
  rules: {
    read_file: {
      allowed_paths: ['./src/**', './docs/**'],
      denied_paths: ['.env', '**/*.key', '**/*.pem'],
      auto_approve: true
    },
    write_file: {
      allowed_paths: ['./src/**'],
      denied_paths: ['./src/config/**'],
      requires_confirmation: true
    },
    bash: {
      allowed_commands: ['npm', 'git', 'ls', 'cat'],
      denied_patterns: ['rm -rf', 'sudo', '> /dev/'],
      sandbox: true
    }
  },
  resource_limits: {
    max_file_size: '10MB',
    max_bash_runtime: 30000, // ms
    max_concurrent_operations: 5
  }
};
```

## Troubleshooting

### Context Window Overflow

**Problem**: "Context window exceeded" errors

**Solution**: Implement aggressive compaction
```typescript
// Increase compaction aggressiveness
contextManager.setCompactionStrategy({
  trigger_threshold: 0.7, // Compact at 70% capacity
  keep_recent_messages: 20,
  summarize_threshold: 50, // Summarize if >50 messages
  remove_tool_results: 'keep_latest_10'
});
```

### Slow Agent Responses

**Problem**: Multi-agent delegation causes delays

**Solution**: Use parallel execution where possible
```typescript
// Execute independent agents in parallel
const [exploreResult, analyzeResult] = await Promise.all([
  orchestrator.delegateTask('explore', context),
  orchestrator.delegateTask('analyze', context)
]);
```

### Cache Miss Rate High

**Problem**: Poor prompt cache hit rate

**Solution**: Reorder prompt segments
```typescript
// Place stable content first
const optimizedPrompt = [
  staticInstructions,    // Rarely changes
  toolSchemas,           // Changes when tools added
  projectContext,        // Changes per project
  conversationHistory    // Always changes
];
```

### Permission Deadlocks

**Problem**: Agent stuck waiting for user confirmation

**Solution**: Implement timeout-based defaults
```typescript
permissionManager.setConfirmationTimeout({
  timeout: 30000, // 30 seconds
  default_action: 'deny', // Or 'allow' for trusted environments
  show_notification: true
});
```

## References

- **Main Repository**: https://github.com/luyao618/Claude-Code-Source-Study
- **Reading Guide**: See `docs/00-目录与阅读指引.md`
- **Architecture Summary**: See `docs/25-架构模式总结.md`
- **Key Learnings**: Each article ends with 2-3 transferable design patterns

## Learning Recommendations

1. **For AI Engineers**: Focus on articles 04-08 (AI core) and 12-13 (agents)
2. **For Tool Builders**: Study articles 09-11 (tools & commands)
3. **For Security-Focused**: Read articles 16-18 (permissions, settings, hooks)
4. **For Full-Stack**: Complete all 25 articles sequentially

This is a study resource, not executable code. The patterns and architectures documented here are extracted from Claude Code's source and can be applied to your own AI agent projects.
