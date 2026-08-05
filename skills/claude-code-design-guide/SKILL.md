---
name: claude-code-design-guide
description: Comprehensive guide to understanding Claude Code's architecture, Agent Runtime, tool systems, and Context Engineering for AI agent development
triggers:
  - how does Claude Code work internally
  - explain Claude Code architecture and design patterns
  - what is Context Engineering in Claude Code
  - how to build AI agents like Claude Code
  - show me Claude Code's tool system design
  - explain Agent Runtime and multi-agent patterns
  - how does Claude Code manage permissions and security
  - teach me about MCP protocol and Claude Code extensions
---

# Claude Code Design Guide

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## Overview

The **Claude Code Design Guide** (claude-code-design-guide) is a comprehensive deep-dive into Claude Code's internal architecture, design patterns, and implementation strategies. This resource bridges the gap between early internet design patterns (Unix philosophy, REPL evolution) and modern AI Agent implementation, providing developers with actionable insights into building production-grade AI coding assistants.

The guide covers:
- **Agent Runtime Systems**: Complete architecture from query engine to multi-agent coordination
- **Tool System Design**: 43 built-in tools, permission models, and extensibility patterns
- **Context Engineering**: System prompts, memory management, auto-compaction strategies
- **Extension Systems**: MCP protocol, Skills, and plugin architectures
- **Security & Performance**: Permission layers, sandboxing, and optimization techniques

## Installation

Clone the repository:

```bash
git clone https://github.com/6551Team/claude-code-design-guide.git
cd claude-code-design-guide
```

The guide is organized as markdown files in a structured directory:

```
claude-code-design-guide/
├── part1/          # Introduction & quickstart (beginner-friendly)
├── part2/          # Unix philosophy to AI agents
├── part3/          # Architecture design
├── part4/          # Tool system design
├── part5/          # Context Engineering
├── part6/          # Agent Runtime & multi-agent
├── part7/          # Extension systems (MCP, Skills, Plugins)
├── part8/          # Security, permissions, performance
├── part9/          # Design philosophy & future
└── architecture/   # Advanced source code analysis
```

## Key Concepts

### 1. Agent Runtime System

Claude Code implements a complete **Agent Runtime** that orchestrates:
- Query engine (conversation heart)
- State management
- Message loops and streaming
- Tool invocation lifecycle

**Core Architecture Pattern:**

```javascript
// Simplified Agent Runtime flow
class AgentRuntime {
  constructor(config) {
    this.queryEngine = new QueryEngine(config.model);
    this.toolRegistry = new ToolRegistry();
    this.stateManager = new StateManager();
    this.contextBuilder = new ContextBuilder();
  }

  async executeQuery(userMessage) {
    // 1. Build context with history + system prompt
    const context = await this.contextBuilder.build({
      history: this.stateManager.getHistory(),
      systemPrompt: this.generateSystemPrompt(),
      memory: this.stateManager.getMemory()
    });

    // 2. Stream response with tool calls
    const stream = await this.queryEngine.query({
      messages: [...context, { role: 'user', content: userMessage }],
      tools: this.toolRegistry.getAvailableTools()
    });

    // 3. Handle tool use loop
    for await (const chunk of stream) {
      if (chunk.type === 'tool_use') {
        const result = await this.executeTool(chunk.name, chunk.input);
        // Feed result back into conversation
        await this.queryEngine.continueWithToolResult(chunk.id, result);
      } else if (chunk.type === 'text') {
        yield chunk.content;
      }
    }
  }

  async executeTool(toolName, input) {
    const tool = this.toolRegistry.get(toolName);
    
    // Permission check
    if (!await this.checkPermission(tool, input)) {
      throw new PermissionDeniedError(toolName);
    }

    return await tool.execute(input);
  }
}
```

### 2. Tool System Design

Claude Code's tool system follows a **declarative schema** pattern:

```javascript
// Tool definition pattern
class FileEditTool {
  static schema = {
    name: 'edit_file',
    description: 'Edit a file with precise line-based operations',
    input_schema: {
      type: 'object',
      properties: {
        path: { type: 'string', description: 'File path' },
        operations: {
          type: 'array',
          items: {
            type: 'object',
            properties: {
              type: { enum: ['insert', 'replace', 'delete'] },
              line: { type: 'number' },
              content: { type: 'string' }
            }
          }
        }
      },
      required: ['path', 'operations']
    }
  };

  async execute(input) {
    // Validate input against schema
    this.validate(input);
    
    // Execute with atomic operations
    const file = await fs.readFile(input.path, 'utf-8');
    const lines = file.split('\n');
    
    for (const op of input.operations) {
      switch (op.type) {
        case 'insert':
          lines.splice(op.line, 0, op.content);
          break;
        case 'replace':
          lines[op.line] = op.content;
          break;
        case 'delete':
          lines.splice(op.line, 1);
          break;
      }
    }
    
    await fs.writeFile(input.path, lines.join('\n'));
    return { success: true, modified_lines: input.operations.length };
  }
}
```

**Tool Registry Pattern:**

```javascript
class ToolRegistry {
  constructor() {
    this.tools = new Map();
    this.permissions = new PermissionManager();
  }

  register(tool) {
    this.tools.set(tool.schema.name, tool);
  }

  getAvailableTools(context) {
    return Array.from(this.tools.values())
      .filter(tool => this.permissions.isAllowed(tool, context))
      .map(tool => tool.schema);
  }
}
```

### 3. Context Engineering

**System Prompt Construction:**

```javascript
class ContextBuilder {
  generateSystemPrompt() {
    return `
You are Claude Code, an AI coding assistant with access to developer tools.

# Capabilities
${this.listAvailableTools()}

# Working Directory
${process.cwd()}

# Project Context
${this.readClaudeMd()}

# Code Style Guidelines
${this.getCodeStyle()}

# Memory (from previous sessions)
${this.getMemorySnippets()}

When editing code:
- Use edit_file for precise line-based changes
- Always show context around changes
- Prefer atomic operations
- Validate before executing

When exploring:
- Use read_file to understand code structure
- Use list_directory to navigate
- Use search_files to find patterns
`.trim();
  }

  readClaudeMd() {
    try {
      return fs.readFileSync('.claude/CLAUDE.md', 'utf-8');
    } catch {
      return 'No project-specific context';
    }
  }

  async build({ history, systemPrompt, memory }) {
    const messages = [{ role: 'system', content: systemPrompt }];
    
    // Add memory snippets as context
    if (memory.length > 0) {
      messages.push({
        role: 'user',
        content: `Relevant context from previous sessions:\n${memory.join('\n')}`
      });
    }

    // Compact old messages if context too large
    const compactedHistory = await this.compactIfNeeded(history);
    
    return [...messages, ...compactedHistory];
  }

  async compactIfNeeded(history) {
    const tokenCount = this.estimateTokens(history);
    
    if (tokenCount > this.maxContextTokens * 0.8) {
      // Summarize old messages, keep recent ones
      const recent = history.slice(-20);
      const old = history.slice(0, -20);
      
      const summary = await this.summarize(old);
      return [
        { role: 'user', content: `Previous context summary: ${summary}` },
        ...recent
      ];
    }
    
    return history;
  }
}
```

### 4. Permission Model

**Layered Permission System:**

```javascript
class PermissionManager {
  constructor() {
    this.layers = {
      tool: new ToolPermissions(),
      path: new PathPermissions(),
      network: new NetworkPermissions(),
      system: new SystemPermissions()
    };
  }

  async checkPermission(tool, input, context) {
    // Layer 1: Tool-level permission
    if (!this.layers.tool.allows(tool.name, context.mode)) {
      return { allowed: false, reason: 'tool_disabled' };
    }

    // Layer 2: Path restrictions
    if (tool.accessesFilesystem) {
      const pathCheck = this.layers.path.validate(input.path);
      if (!pathCheck.allowed) {
        return pathCheck;
      }
    }

    // Layer 3: Network restrictions
    if (tool.accessesNetwork) {
      const networkCheck = this.layers.network.validate(input.url);
      if (!networkCheck.allowed) {
        return networkCheck;
      }
    }

    // Layer 4: System-level restrictions
    if (tool.isPrivileged) {
      return await this.layers.system.requestApproval(tool, input);
    }

    return { allowed: true };
  }
}

class PathPermissions {
  constructor() {
    this.allowedPaths = [process.cwd()];
    this.deniedPatterns = [
      /node_modules/,
      /\.git/,
      /\.env$/,
      /\.ssh/,
      /\/etc\//
    ];
  }

  validate(path) {
    const resolved = path.resolve(path);
    
    // Must be within allowed paths
    if (!this.allowedPaths.some(allowed => resolved.startsWith(allowed))) {
      return { allowed: false, reason: 'path_outside_workspace' };
    }

    // Must not match denied patterns
    if (this.deniedPatterns.some(pattern => pattern.test(resolved))) {
      return { allowed: false, reason: 'path_restricted' };
    }

    return { allowed: true };
  }
}
```

### 5. MCP (Model Context Protocol)

**MCP Server Integration:**

```javascript
// MCP server connection
class MCPClient {
  constructor(serverConfig) {
    this.serverUrl = serverConfig.url;
    this.capabilities = null;
  }

  async connect() {
    const response = await fetch(`${this.serverUrl}/mcp/capabilities`);
    this.capabilities = await response.json();
  }

  async listTools() {
    const response = await fetch(`${this.serverUrl}/mcp/tools`);
    return await response.json();
  }

  async invokeTool(toolName, args) {
    const response = await fetch(`${this.serverUrl}/mcp/tools/${toolName}`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(args)
    });
    
    return await response.json();
  }
}

// Register MCP tools in tool registry
class MCPToolAdapter {
  constructor(mcpClient, mcpTool) {
    this.client = mcpClient;
    this.mcpTool = mcpTool;
  }

  get schema() {
    return {
      name: `mcp_${this.mcpTool.name}`,
      description: this.mcpTool.description,
      input_schema: this.mcpTool.inputSchema
    };
  }

  async execute(input) {
    return await this.client.invokeTool(this.mcpTool.name, input);
  }
}
```

### 6. Multi-Agent Coordination

**Coordinator Pattern:**

```javascript
class AgentCoordinator {
  constructor() {
    this.agents = {
      planner: new PlannerAgent(),
      executor: new ExecutorAgent(),
      reviewer: new ReviewerAgent()
    };
  }

  async executeTask(taskDescription) {
    // Phase 1: Planning
    const plan = await this.agents.planner.createPlan(taskDescription);
    
    console.log('Plan:', plan.steps);

    // Phase 2: Execution
    const results = [];
    for (const step of plan.steps) {
      const result = await this.agents.executor.execute(step);
      results.push(result);
      
      // Adaptive replanning if step fails
      if (!result.success) {
        const revisedPlan = await this.agents.planner.replan(
          plan,
          step,
          result.error
        );
        plan.steps = revisedPlan.steps;
      }
    }

    // Phase 3: Review
    const review = await this.agents.reviewer.review({
      task: taskDescription,
      plan: plan,
      results: results
    });

    return {
      completed: review.success,
      steps: results,
      review: review
    };
  }
}

// Planner agent with task decomposition
class PlannerAgent {
  async createPlan(taskDescription) {
    const prompt = `
Break down this task into atomic steps:
${taskDescription}

Return a JSON plan with steps that can be executed independently.
    `;

    const response = await this.query(prompt);
    return JSON.parse(response);
  }
}
```

## Configuration

### CLAUDE.md (Project Context)

Create `.claude/CLAUDE.md` in your project root:

```markdown
# Project: MyApp

## Tech Stack
- Node.js + TypeScript
- Express.js API
- PostgreSQL database
- React frontend

## Architecture
- `/src/api` - REST endpoints
- `/src/services` - Business logic
- `/src/models` - Database models
- `/src/utils` - Shared utilities

## Code Style
- Use async/await (no callbacks)
- Prefer functional patterns
- All exports named (no default exports)
- Comments for complex logic only

## Testing
- Jest for unit tests
- Supertest for API tests
- Run: `npm test`

## Common Tasks
- Add endpoint: Copy pattern from `/src/api/users.ts`
- Database migration: `npm run migrate:create`
```

### Environment Variables

```bash
# API keys (never hardcode)
export ANTHROPIC_API_KEY=your_key_here
export OPENAI_API_KEY=your_key_here

# Claude Code config
export CLAUDE_CODE_MODE=developer  # or 'safe', 'autonomous'
export CLAUDE_CODE_MAX_TOKENS=100000
export CLAUDE_CODE_COMPACT_THRESHOLD=80000
```

## Common Patterns

### Pattern 1: Task-Based Development

```javascript
// Using Claude Code's task system
const task = {
  id: 'add-user-endpoint',
  description: 'Add POST /api/users endpoint with validation',
  subtasks: [
    'Create route handler in api/users.ts',
    'Add Zod schema for validation',
    'Write unit tests',
    'Update API documentation'
  ]
};

// Agent breaks down and executes each subtask
await agentRuntime.executeTask(task);
```

### Pattern 2: Tool Composition

```javascript
// Composing multiple tools for complex operations
class RefactoringTool {
  constructor(toolRegistry) {
    this.search = toolRegistry.get('search_files');
    this.read = toolRegistry.get('read_file');
    this.edit = toolRegistry.get('edit_file');
  }

  async renameFunction(oldName, newName) {
    // 1. Find all occurrences
    const matches = await this.search.execute({
      pattern: oldName,
      filePattern: '**/*.ts'
    });

    // 2. Edit each file
    for (const match of matches) {
      const content = await this.read.execute({ path: match.file });
      const operations = this.buildRenameOperations(content, oldName, newName);
      
      await this.edit.execute({
        path: match.file,
        operations
      });
    }

    return { renamed: matches.length, files: matches.map(m => m.file) };
  }
}
```

### Pattern 3: Memory Management

```javascript
// Storing and retrieving context across sessions
class MemoryManager {
  constructor(dbPath) {
    this.db = new Database(dbPath);
  }

  async store(key, content, metadata = {}) {
    const embedding = await this.embed(content);
    
    await this.db.insert({
      key,
      content,
      embedding,
      metadata,
      timestamp: Date.now()
    });
  }

  async recall(query, limit = 5) {
    const queryEmbedding = await this.embed(query);
    
    const results = await this.db.vectorSearch({
      embedding: queryEmbedding,
      limit,
      threshold: 0.7
    });

    return results.map(r => r.content);
  }
}

// Usage in agent
const memory = new MemoryManager('.claude/memory.db');
await memory.store('api-pattern', 'Use Express Router pattern for all endpoints');
const relevantMemories = await memory.recall('how to add new endpoint');
```

## Troubleshooting

### Issue: Context Window Exceeded

**Problem:** Agent stops responding or returns truncated responses.

**Solution:** Enable auto-compaction:

```javascript
// In context builder configuration
contextBuilder.config.autoCompact = true;
contextBuilder.config.compactThreshold = 0.8; // Compact at 80% capacity

// Manual compaction
const compacted = await contextBuilder.compactHistory(history, {
  keepRecent: 20,  // Keep last 20 messages
  summarizeOld: true
});
```

### Issue: Tool Permission Denied

**Problem:** Tool execution fails with permission errors.

**Solution:** Check permission layers:

```javascript
// Debug permission checks
const debugPermission = await permissionManager.checkPermission(
  tool,
  input,
  { debug: true }
);

console.log('Permission check:', debugPermission);
// { allowed: false, reason: 'path_outside_workspace', layer: 'path' }

// Adjust allowed paths
permissionManager.layers.path.allowedPaths.push('/path/to/allowed/dir');
```

### Issue: MCP Server Connection Failed

**Problem:** External MCP server tools not available.

**Solution:** Verify server configuration:

```javascript
// Test MCP server connectivity
const mcpClient = new MCPClient({ url: process.env.MCP_SERVER_URL });

try {
  await mcpClient.connect();
  console.log('Capabilities:', mcpClient.capabilities);
} catch (error) {
  console.error('MCP connection failed:', error.message);
  // Fallback to built-in tools only
}
```

### Issue: Slow Response Times

**Problem:** Agent takes too long to respond.

**Solution:** Optimize context and enable streaming:

```javascript
// Enable streaming for faster perceived performance
const stream = agentRuntime.executeQueryStream(userMessage);

for await (const chunk of stream) {
  process.stdout.write(chunk);  // Output as it arrives
}

// Reduce context size
contextBuilder.config.maxHistoryMessages = 30;  // Instead of 100
contextBuilder.config.includeMemory = false;   // Disable if not needed
```

## Learning Path

1. **Start Here:** Read [part1/01-introduction.md](./part1/01-introduction.md) for overview
2. **Core Concepts:** Study [part3/06-query-engine.md](./part3/06-query-engine.md) (query engine)
3. **Tool System:** Deep dive [part4/09-tool-design.md](./part4/09-tool-design.md)
4. **Context Engineering:** Master [part5/12-context-what.md](./part5/12-context-what.md)
5. **Advanced:** Explore [part6/17-multi-agent.md](./part6/17-multi-agent.md) (multi-agent)
6. **Extensions:** Learn [part7/19-mcp.md](./part7/19-mcp.md) (MCP protocol)

## Key Takeaways

- **Agent Runtime = Query Engine + Tools + Context + State**
- **Tool design follows declarative schemas with strict permission layers**
- **Context Engineering is the art of prompt construction + memory + compaction**
- **MCP enables tool interoperability across different AI systems**
- **Multi-agent coordination uses planner → executor → reviewer pattern**
- **Security is layered: tool → path → network → system**

## References

- GitHub: https://github.com/6551Team/claude-code-design-guide
- Architecture Deep-Dive: [architecture/README.md](./architecture/README.md)
- Advanced Source Analysis: [architecture/README_EN.md](./architecture/README_EN.md)
