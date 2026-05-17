---
name: claude-code-architecture-patterns
description: Architectural patterns and design principles from Anthropic's Claude Code agent, reverse-engineered for building production AI coding agents
triggers:
  - "how does Claude Code architecture work"
  - "show me AI agent patterns from Claude Code"
  - "implement async generator agent loop"
  - "build multi-agent orchestration system"
  - "show me Claude Code state management"
  - "implement tool execution pipeline like Claude Code"
  - "how to build production AI coding agent"
  - "explain Claude Code internals"
---

# Claude Code Architecture Patterns

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

This skill provides expertise in the architectural patterns, design principles, and implementation strategies extracted from Anthropic's Claude Code agent. Use these patterns to build production-grade AI coding agents with robust state management, efficient tool execution, multi-agent orchestration, and context management.

## What This Provides

**Claude Code from Source** is a comprehensive technical analysis of Claude Code's architecture, distilled into 18 chapters covering:

- **Agent Loop Architecture**: AsyncGenerator-based control flow
- **Tool Execution**: Concurrent-safe batching and speculative execution
- **Multi-Agent Orchestration**: Fork agents, task coordination, swarms
- **State Management**: Two-tier architecture with bootstrap singleton and AppState
- **Context Management**: 4-layer compression, prompt cache optimization
- **Memory Systems**: File-based memory with LLM recall
- **Performance**: Token budgets, cache sharing, rendering optimization

## Installation

```bash
# Clone the repository
git clone https://github.com/alejandrobalderas/claude-code-from-source.git
cd claude-code-from-source

# Install dependencies
npm install

# Build the book site (optional)
npm run build

# Start dev server to read online
npm run dev
```

## Key Architectural Patterns

### 1. AsyncGenerator Agent Loop

The core pattern for agent execution - yields messages during execution, returns terminal state.

```typescript
async function* agentLoop(
  query: string,
  context: ExecutionContext
): AsyncGenerator<Message, TerminalState> {
  let conversationHistory: Message[] = [];
  let tokenBudget = context.maxTokens;

  while (tokenBudget > 0) {
    // Compress context if needed
    if (shouldCompress(conversationHistory)) {
      conversationHistory = await compressContext(conversationHistory);
    }

    // Stream response from LLM
    const stream = await streamCompletion({
      messages: conversationHistory,
      tools: context.availableTools,
    });

    let currentMessage = { role: 'assistant', content: '' };
    let toolCalls: ToolCall[] = [];

    for await (const chunk of stream) {
      if (chunk.type === 'text') {
        currentMessage.content += chunk.text;
        yield { ...currentMessage, streaming: true };
      } else if (chunk.type === 'tool_use') {
        toolCalls.push(chunk.toolCall);
      }

      tokenBudget -= chunk.tokensUsed;
    }

    yield { ...currentMessage, streaming: false };
    conversationHistory.push(currentMessage);

    // Execute tools if present
    if (toolCalls.length > 0) {
      const results = await executeToolsConcurrent(toolCalls, context);
      
      for (const result of results) {
        const toolMessage = {
          role: 'user',
          content: formatToolResult(result),
        };
        yield toolMessage;
        conversationHistory.push(toolMessage);
      }
    } else {
      // No tools - agent is done
      return {
        status: 'complete',
        finalMessage: currentMessage,
        tokensUsed: context.maxTokens - tokenBudget,
      };
    }
  }

  return { status: 'budget_exceeded', tokensUsed: context.maxTokens };
}
```

### 2. Concurrent-Safe Tool Execution

Partition tools by safety guarantees, execute reads in parallel, serialize writes.

```typescript
interface ToolCall {
  id: string;
  name: string;
  args: Record<string, unknown>;
}

interface ToolDefinition {
  name: string;
  execute: (args: Record<string, unknown>) => Promise<unknown>;
  readonly: boolean;
  requiresConfirmation: boolean;
}

async function executeToolsConcurrent(
  calls: ToolCall[],
  context: ExecutionContext
): Promise<ToolResult[]> {
  // Partition by safety
  const readOnly = calls.filter(c => 
    context.tools[c.name]?.readonly === true
  );
  const writeOps = calls.filter(c => 
    context.tools[c.name]?.readonly !== true
  );

  const results: ToolResult[] = [];

  // Execute all read-only tools in parallel
  if (readOnly.length > 0) {
    const readResults = await Promise.all(
      readOnly.map(call => executeTool(call, context))
    );
    results.push(...readResults);
  }

  // Execute write operations serially
  for (const call of writeOps) {
    // Check if confirmation needed
    if (context.tools[call.name]?.requiresConfirmation) {
      const approved = await requestUserConfirmation(call);
      if (!approved) {
        results.push({
          id: call.id,
          status: 'rejected',
          error: 'User declined permission',
        });
        continue;
      }
    }

    const result = await executeTool(call, context);
    results.push(result);
  }

  return results;
}

async function executeTool(
  call: ToolCall,
  context: ExecutionContext
): Promise<ToolResult> {
  const tool = context.tools[call.name];
  if (!tool) {
    return { id: call.id, status: 'error', error: 'Unknown tool' };
  }

  try {
    const output = await tool.execute(call.args);
    return { id: call.id, status: 'success', output };
  } catch (error) {
    return { 
      id: call.id, 
      status: 'error', 
      error: error instanceof Error ? error.message : 'Unknown error' 
    };
  }
}
```

### 3. Speculative Tool Execution

Start read-only tools during streaming, before response completes.

```typescript
async function* agentLoopWithSpeculation(
  query: string,
  context: ExecutionContext
): AsyncGenerator<Message, TerminalState> {
  const stream = await streamCompletion({
    messages: context.history,
    tools: context.availableTools,
  });

  const toolCalls: ToolCall[] = [];
  const speculativeResults = new Map<string, Promise<ToolResult>>();

  for await (const chunk of stream) {
    if (chunk.type === 'tool_use') {
      const call = chunk.toolCall;
      toolCalls.push(call);

      // Start read-only tools immediately
      const tool = context.tools[call.name];
      if (tool?.readonly) {
        speculativeResults.set(
          call.id,
          executeTool(call, context)
        );
        yield {
          role: 'system',
          content: `⚡ Started ${call.name} speculatively`,
        };
      }
    }
  }

  // Wait for speculative results
  const results: ToolResult[] = [];
  for (const call of toolCalls) {
    if (speculativeResults.has(call.id)) {
      results.push(await speculativeResults.get(call.id)!);
    } else {
      results.push(await executeTool(call, context));
    }
  }

  return { status: 'complete', results };
}
```

### 4. Fork Agents for Cache Sharing

Spawn parallel agents with byte-identical prompt prefixes to share prompt cache.

```typescript
interface ForkAgentConfig {
  parentContext: ConversationHistory;
  tasks: string[];
  sharedPrefix: Message[];
}

async function forkAgents(
  config: ForkAgentConfig
): Promise<AgentResult[]> {
  // Build byte-identical prefix
  const sharedPrefix = config.sharedPrefix.map(msg => ({
    role: msg.role,
    content: msg.content,
    // Ensure exact JSON serialization
    _cacheKey: JSON.stringify({ role: msg.role, content: msg.content }),
  }));

  // Spawn parallel agents
  const agents = config.tasks.map(async (task) => {
    const childMessages = [
      ...sharedPrefix,
      { role: 'user', content: task },
    ];

    const result: AgentResult = {
      task,
      messages: [],
      status: 'running',
    };

    // Run agent loop
    const loop = agentLoop(task, {
      ...config.parentContext,
      history: childMessages,
    });

    for await (const msg of loop) {
      result.messages.push(msg);
    }

    const terminal = await loop.next();
    result.status = terminal.value.status;

    return result;
  });

  return Promise.all(agents);
}

// Usage
const results = await forkAgents({
  parentContext: mainContext,
  sharedPrefix: conversationHistory.slice(0, -1),
  tasks: [
    'Analyze the authentication module',
    'Review error handling patterns',
    'Document the API endpoints',
  ],
});
```

### 5. Four-Layer Context Compression

Progressive compression strategies to fit within token budget.

```typescript
type CompressionLevel = 'snip' | 'microcompact' | 'collapse' | 'autocompact';

interface CompressionStrategy {
  level: CompressionLevel;
  apply: (messages: Message[]) => Promise<Message[]>;
  estimatedReduction: number; // 0.0 to 1.0
}

const compressionStrategies: CompressionStrategy[] = [
  {
    level: 'snip',
    estimatedReduction: 0.3,
    apply: async (messages) => {
      // Remove middle portions of long tool outputs
      return messages.map(msg => {
        if (msg.role === 'user' && msg.toolResult) {
          const output = msg.toolResult.output as string;
          if (output.length > 4000) {
            const head = output.slice(0, 1500);
            const tail = output.slice(-1500);
            return {
              ...msg,
              toolResult: {
                ...msg.toolResult,
                output: `${head}\n\n[... ${output.length - 3000} chars omitted ...]\n\n${tail}`,
              },
            };
          }
        }
        return msg;
      });
    },
  },
  {
    level: 'microcompact',
    estimatedReduction: 0.5,
    apply: async (messages) => {
      // Use LLM to summarize old messages
      const cutoff = messages.length - 10;
      const toCompress = messages.slice(0, cutoff);
      const toKeep = messages.slice(cutoff);

      if (toCompress.length === 0) return messages;

      const summary = await summarizeWithLLM(toCompress);
      return [
        { role: 'system', content: `[Prior context]: ${summary}` },
        ...toKeep,
      ];
    },
  },
  {
    level: 'collapse',
    estimatedReduction: 0.7,
    apply: async (messages) => {
      // Merge consecutive messages from same role
      const collapsed: Message[] = [];
      for (const msg of messages) {
        const last = collapsed[collapsed.length - 1];
        if (last && last.role === msg.role) {
          last.content += '\n\n' + msg.content;
        } else {
          collapsed.push({ ...msg });
        }
      }
      return collapsed;
    },
  },
  {
    level: 'autocompact',
    estimatedReduction: 0.85,
    apply: async (messages) => {
      // Aggressive: keep only last 5 messages + system prompt
      const system = messages.find(m => m.role === 'system');
      const recent = messages.slice(-5);
      return system ? [system, ...recent] : recent;
    },
  },
];

async function compressContext(
  messages: Message[],
  targetTokens: number
): Promise<Message[]> {
  let current = messages;
  let currentTokens = estimateTokens(current);

  for (const strategy of compressionStrategies) {
    if (currentTokens <= targetTokens) break;

    console.log(`Applying ${strategy.level} compression...`);
    current = await strategy.apply(current);
    currentTokens = estimateTokens(current);
  }

  return current;
}
```

### 6. Two-Tier State Management

Bootstrap singleton for process-level config, AppState for runtime state.

```typescript
// Bootstrap singleton - loaded once at startup
class Bootstrap {
  private static instance: Bootstrap;
  
  readonly config: {
    apiKey: string;
    model: string;
    maxTokens: number;
    enableCache: boolean;
  };
  
  readonly tools: Map<string, ToolDefinition>;
  readonly skills: Map<string, SkillDefinition>;

  private constructor() {
    // Load from env/config files
    this.config = {
      apiKey: process.env.ANTHROPIC_API_KEY!,
      model: process.env.MODEL || 'claude-3-5-sonnet-20241022',
      maxTokens: parseInt(process.env.MAX_TOKENS || '100000', 10),
      enableCache: process.env.ENABLE_CACHE !== 'false',
    };

    this.tools = this.loadTools();
    this.skills = this.loadSkills();
  }

  static getInstance(): Bootstrap {
    if (!Bootstrap.instance) {
      Bootstrap.instance = new Bootstrap();
    }
    return Bootstrap.instance;
  }

  private loadTools(): Map<string, ToolDefinition> {
    // Load tool definitions from disk
    return new Map();
  }

  private loadSkills(): Map<string, SkillDefinition> {
    // Load skills with frontmatter only
    return new Map();
  }
}

// AppState - runtime mutable state
class AppState {
  conversationHistory: Message[] = [];
  tokenBudget: number;
  costTracking: {
    inputTokens: number;
    outputTokens: number;
    cacheReads: number;
    cacheWrites: number;
  } = {
    inputTokens: 0,
    outputTokens: 0,
    cacheReads: 0,
    cacheWrites: 0,
  };
  
  // Sticky latches - once set, never unset
  betaHeaders = new Set<string>();

  constructor() {
    const bootstrap = Bootstrap.getInstance();
    this.tokenBudget = bootstrap.config.maxTokens;
  }

  trackUsage(usage: TokenUsage): void {
    this.costTracking.inputTokens += usage.input;
    this.costTracking.outputTokens += usage.output;
    this.costTracking.cacheReads += usage.cacheRead || 0;
    this.costTracking.cacheWrites += usage.cacheWrite || 0;
    this.tokenBudget -= (usage.input + usage.output);
  }

  enableBetaFeature(feature: string): void {
    // Sticky latch - never remove once added
    this.betaHeaders.add(feature);
  }

  getCost(): number {
    // Calculate total cost based on pricing
    const inputCost = this.costTracking.inputTokens * 0.000003;
    const outputCost = this.costTracking.outputTokens * 0.000015;
    const cacheCost = this.costTracking.cacheReads * 0.0000003;
    return inputCost + outputCost + cacheCost;
  }
}
```

### 7. File-Based Memory with LLM Recall

Store memories as markdown files, use LLM to select relevant ones.

```typescript
import { promises as fs } from 'fs';
import path from 'path';

interface Memory {
  id: string;
  type: 'fact' | 'preference' | 'context' | 'decision';
  content: string;
  timestamp: string;
  tags: string[];
}

class MemorySystem {
  private memoryDir: string;

  constructor(projectRoot: string) {
    this.memoryDir = path.join(projectRoot, '.claude', 'memory');
  }

  async initialize(): Promise<void> {
    await fs.mkdir(this.memoryDir, { recursive: true });
  }

  async store(memory: Memory): Promise<void> {
    const filename = `${memory.id}.md`;
    const filepath = path.join(this.memoryDir, filename);
    
    const content = [
      `# ${memory.type.toUpperCase()}: ${memory.id}`,
      '',
      `**Timestamp**: ${memory.timestamp}`,
      `**Tags**: ${memory.tags.join(', ')}`,
      '',
      memory.content,
    ].join('\n');

    await fs.writeFile(filepath, content, 'utf-8');
  }

  async recall(query: string): Promise<Memory[]> {
    // Load all memory files
    const files = await fs.readdir(this.memoryDir);
    const memories: Memory[] = [];

    for (const file of files) {
      if (!file.endsWith('.md')) continue;
      
      const content = await fs.readFile(
        path.join(this.memoryDir, file),
        'utf-8'
      );
      
      const memory = this.parseMemory(content, file);
      memories.push(memory);
    }

    // Use LLM to select relevant memories
    const relevant = await this.selectRelevantMemories(query, memories);
    return relevant;
  }

  private parseMemory(content: string, filename: string): Memory {
    const lines = content.split('\n');
    const header = lines[0];
    const type = header.match(/# (\w+):/)?.[1].toLowerCase() as Memory['type'];
    const id = filename.replace('.md', '');
    
    const timestampLine = lines.find(l => l.startsWith('**Timestamp**:'));
    const timestamp = timestampLine?.split(': ')[1] || new Date().toISOString();
    
    const tagsLine = lines.find(l => l.startsWith('**Tags**:'));
    const tags = tagsLine?.split(': ')[1].split(', ') || [];
    
    const contentStart = lines.findIndex(l => l.startsWith('**Tags**:')) + 2;
    const memoryContent = lines.slice(contentStart).join('\n');

    return { id, type, content: memoryContent, timestamp, tags };
  }

  private async selectRelevantMemories(
    query: string,
    allMemories: Memory[]
  ): Promise<Memory[]> {
    if (allMemories.length === 0) return [];

    // Use small model for memory selection
    const prompt = `Given this query: "${query}"
    
Select the 3 most relevant memories from this list:

${allMemories.map((m, i) => `${i + 1}. [${m.type}] ${m.tags.join(', ')}: ${m.content.slice(0, 100)}...`).join('\n')}

Respond with only the numbers of relevant memories (e.g., "1, 3, 7"):`;

    const response = await callLLM({
      model: 'claude-3-haiku-20240307',
      messages: [{ role: 'user', content: prompt }],
      maxTokens: 50,
    });

    const indices = response.content
      .match(/\d+/g)
      ?.map(n => parseInt(n, 10) - 1) || [];

    return indices
      .filter(i => i >= 0 && i < allMemories.length)
      .map(i => allMemories[i]);
  }
}

// Usage
const memory = new MemorySystem(process.cwd());
await memory.initialize();

await memory.store({
  id: 'pref-001',
  type: 'preference',
  content: 'User prefers TypeScript with strict mode enabled',
  timestamp: new Date().toISOString(),
  tags: ['typescript', 'preferences', 'code-style'],
});

const relevant = await memory.recall('generate a new TypeScript file');
// Returns memories about TypeScript preferences
```

## Common Implementation Patterns

### Multi-Agent Coordination

```typescript
interface Task {
  id: string;
  description: string;
  status: 'pending' | 'running' | 'complete' | 'failed';
  assignedAgent?: string;
  result?: unknown;
}

class TaskCoordinator {
  private tasks: Map<string, Task> = new Map();
  private agents: Map<string, AgentInstance> = new Map();

  async coordinate(taskDescriptions: string[]): Promise<Map<string, unknown>> {
    // Create tasks
    for (const desc of taskDescriptions) {
      const task: Task = {
        id: crypto.randomUUID(),
        description: desc,
        status: 'pending',
      };
      this.tasks.set(task.id, task);
    }

    // Spawn agents
    const agentPromises = Array.from(this.tasks.values()).map(task =>
      this.runTaskAgent(task)
    );

    await Promise.all(agentPromises);

    // Collect results
    const results = new Map<string, unknown>();
    for (const [id, task] of this.tasks) {
      if (task.status === 'complete') {
        results.set(id, task.result);
      }
    }

    return results;
  }

  private async runTaskAgent(task: Task): Promise<void> {
    task.status = 'running';
    
    const agent = agentLoop(task.description, {
      availableTools: Bootstrap.getInstance().tools,
      maxTokens: 10000,
      history: [],
    });

    const messages: Message[] = [];
    for await (const msg of agent) {
      messages.push(msg);
    }

    const terminal = await agent.next();
    
    if (terminal.value.status === 'complete') {
      task.status = 'complete';
      task.result = terminal.value.finalMessage;
    } else {
      task.status = 'failed';
    }
  }
}
```

### Token Budget Management

```typescript
class TokenBudgetManager {
  private budget: number;
  private reserved: number = 8192; // Default output reservation

  constructor(maxTokens: number) {
    this.budget = maxTokens;
  }

  checkAndReserve(estimatedTokens: number): boolean {
    const available = this.budget - this.reserved;
    return estimatedTokens <= available;
  }

  consume(actualTokens: number): void {
    this.budget -= actualTokens;
  }

  escalateReservation(newReservation: number): void {
    // Increase output reservation (e.g., when hitting limit)
    if (newReservation > this.reserved) {
      this.reserved = newReservation;
    }
  }

  getRemaining(): number {
    return this.budget - this.reserved;
  }

  getAvailableForOutput(): number {
    return Math.min(this.reserved, this.budget);
  }
}
```

## Reading the Book

### Online

Visit [claude-code-from-source.com](https://claude-code-from-source.com) to read all 18 chapters with rendered diagrams.

### Locally

```bash
# Start dev server
npm run dev

# Open http://localhost:3000
```

### Chapter Organization

Each chapter follows this structure:

1. **Narrative flow** - Technical leaders can read straight through
2. **Deep-dive sections** - Implementers get code-level detail
3. **Diagrams** - Mermaid diagrams for visual learners
4. **Apply This** - Transferable patterns you can steal

### Key Chapters by Use Case

**Building an agent from scratch?**
- Ch 1: The Architecture of an AI Agent
- Ch 5: The Agent Loop
- Ch 6: Tools — From Definition to Execution

**Optimizing token usage?**
- Ch 4: Talking to Claude — The API Layer
- Ch 9: Fork Agents and the Prompt Cache
- Ch 17: Performance

**Multi-agent systems?**
- Ch 8: Spawning Sub-Agents
- Ch 10: Tasks, Coordination, and Swarms

**Adding memory/learning?**
- Ch 11: Memory — Learning Across Conversations
- Ch 12: Extensibility — Skills and Hooks

## Configuration

The book itself is a static site built with Next.js. Configuration is in `web/`:

```typescript
// web/next.config.js
module.exports = {
  output: 'export',
  images: { unoptimized: true },
  basePath: process.env.BASE_PATH || '',
};
```

To customize the reading experience:

```bash
# Change theme, fonts, etc in web/styles/
cd web/styles

# Modify chapter navigation in web/components/
cd web/components
```

## Troubleshooting

### "Module not found" when building

```bash
# Clear cache and reinstall
rm -rf node_modules package-lock.json
npm install
```

### Diagrams not rendering

Mermaid diagrams render in:
- GitHub (natively)
- The web build (using mermaid.js)
- VSCode (with Markdown Preview Mermaid Support extension)

If diagrams don't show, ensure you're viewing the built site or GitHub.

### Port 3000 already in use

```bash
# Use different port
PORT=3001 npm run dev
```

## Environment Variables

The book itself needs no environment variables. If you're implementing the patterns:

```bash
# For agent implementation
ANTHROPIC_API_KEY=your_key_here
MODEL=claude-3-5-sonnet-20241022
MAX_TOKENS=100000
ENABLE_CACHE=true

# For memory system
MEMORY_DIR=./.claude/memory

# For multi-agent
MAX_CONCURRENT_AGENTS=10
AGENT_TIMEOUT_MS=300000
```

## Best Practices

1. **Start with the agent loop** - AsyncGenerator is the foundation
2. **Add tools incrementally** - Begin with read-only, add writes carefully
3. **Use the compression ladder** - Don't jump straight to aggressive compression
4. **Cache-optimize for parallel agents** - Fork agents share prefix = 95% savings
5. **Track token usage** - Always know your budget and consumption
6. **Memory is opt-in** - Don't store everything; be selective
7. **Test with real workloads** - Synthetic examples hide edge cases

## Further Reading

- **Ch 17: Performance** - Deep dive on every optimization technique
- **Ch 18: Epilogue** - The 5 architectural bets that make Claude Code work
- **MCP Integration (Ch 15)** - Connect to external tools and services
- **Remote Execution (Ch 16)** - Cloud-based agent orchestration

## Resources

- [Book Website](https://claude-code-from-source.com)
- [GitHub Repository](https://github.com/alejandrobalderas/claude-code-from-source)
- [Anthropic API Documentation](https://docs.anthropic.com/)
- [Model Context Protocol](https://modelcontextprotocol.io/)

---

**Remember**: This book contains no proprietary code. Every example is original pseudocode written to teach the patterns. Use these patterns to build your own production AI agents.
