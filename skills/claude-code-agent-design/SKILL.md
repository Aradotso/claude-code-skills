---
name: claude-code-agent-design
description: Deep dive guide for building AI agents with Claude Code - architecture, tools, context engineering, and runtime patterns
triggers:
  - how to build an AI agent with Claude Code
  - explain Claude Code agent architecture
  - show me Claude Code tool system design
  - how does context engineering work in Claude Code
  - implement MCP protocol integration
  - design multi-agent system with Claude Code
  - Claude Code permission model and security
  - build custom tools for Claude Code
---

# Claude Code Agent Design Guide

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

Expert guidance for understanding and implementing AI agent systems based on Claude Code's architecture. This comprehensive guide covers agent runtime design, tool systems, context engineering, multi-agent coordination, and extension mechanisms.

## What This Guide Covers

Claude Code is Anthropic's official AI programming assistant CLI tool - not just a chatbot, but a complete **Agent Runtime System** including:

- Tool calling and execution framework
- Context engineering patterns
- Multi-agent collaboration
- Permission management
- Extension system (MCP protocol, Skills, Plugins)
- State management and message loops

## Core Architecture Concepts

### Query Engine - The Heart of Conversations

The query engine manages the agent's interaction loop:

```javascript
// Core query engine pattern
class QueryEngine {
  async processQuery(userInput, context) {
    // 1. Context preparation
    const systemPrompt = this.buildSystemPrompt(context);
    const messages = this.prepareMessages(userInput, context.history);
    
    // 2. LLM invocation with tool use
    const response = await this.llm.complete({
      system: systemPrompt,
      messages: messages,
      tools: this.availableTools,
      stream: true
    });
    
    // 3. Tool execution loop
    while (response.hasToolCalls()) {
      const toolResults = await this.executeTool(response.toolCalls);
      response = await this.llm.continue(toolResults);
    }
    
    return response.finalMessage;
  }
}
```

### Tool System Design Philosophy

Tools are the agent's hands - allowing it to interact with the world:

```javascript
// Tool definition pattern
const toolDefinition = {
  name: "execute_command",
  description: "Execute a shell command in the project directory",
  input_schema: {
    type: "object",
    properties: {
      command: {
        type: "string",
        description: "The command to execute"
      },
      workingDirectory: {
        type: "string",
        description: "Optional working directory"
      }
    },
    required: ["command"]
  },
  
  // Permission level
  permission: "auto", // auto, ask, deny
  
  // Execution handler
  async execute({ command, workingDirectory }, context) {
    const result = await context.shell.exec(command, {
      cwd: workingDirectory || context.projectRoot
    });
    return {
      stdout: result.stdout,
      stderr: result.stderr,
      exitCode: result.code
    };
  }
};
```

### Built-in Tools Categories

Claude Code includes 43 built-in tools organized by function:

```javascript
// File system operations
const fileTools = [
  'read_file',           // Read file contents
  'write_file',          // Write to file
  'edit_file',           // Edit specific lines
  'list_directory',      // List directory contents
  'create_directory',    // Create directories
  'move_file',           // Move/rename files
  'delete_file'          // Delete files
];

// Code analysis
const analysisTools = [
  'search_files',        // Grep-like search
  'analyze_code',        // AST analysis
  'find_references',     // Symbol references
  'get_diagnostics'      // Linter/type errors
];

// Execution
const execTools = [
  'execute_command',     // Run shell commands
  'run_tests',          // Execute test suites
  'start_dev_server',   // Start dev environment
  'install_packages'    // Package manager operations
];

// Memory and context
const memoryTools = [
  'update_memory',       // Store information
  'read_memory',        // Retrieve stored data
  'add_to_claudemd'     // Update project memory
];
```

## Context Engineering

### System Prompt Construction

Building effective system prompts is critical:

```javascript
function buildSystemPrompt(context) {
  const sections = [
    // 1. Identity and capabilities
    `You are Claude Code, an AI programming assistant.
You have access to ${context.tools.length} tools for file operations, 
code analysis, command execution, and more.`,
    
    // 2. Project context
    context.claudeMd ? `
## Project Context (from CLAUDE.md)
${context.claudeMd}
` : '',
    
    // 3. Active task context
    context.activeTask ? `
## Current Task
${context.activeTask.description}
Progress: ${context.activeTask.progress}
` : '',
    
    // 4. Memory snippets (relevant past context)
    context.memory.length > 0 ? `
## Relevant Memory
${context.memory.map(m => `- ${m.content}`).join('\n')}
` : '',
    
    // 5. Tool usage guidelines
    `
## Tool Usage Guidelines
- Use file operations tools to read and modify code
- Execute commands to run tests and checks
- Update memory for important discoveries
- Ask for permission before destructive operations
`,
    
    // 6. Output formatting
    `
## Response Format
- Explain your reasoning before taking actions
- Show command output and analysis
- Suggest next steps when tasks complete
`
  ];
  
  return sections.filter(Boolean).join('\n\n');
}
```

### CLAUDE.md - Project Memory

The CLAUDE.md file serves as persistent project context:

```markdown
<!-- Example CLAUDE.md structure -->

# Project Overview
This is a React + TypeScript web application for task management.

## Tech Stack
- React 18 with TypeScript
- Vite for bundling
- TanStack Query for data fetching
- Tailwind CSS for styling

## Key Architecture Decisions
- All API calls go through `src/api/client.ts`
- State management uses React Context + TanStack Query
- Component structure: `src/components/{feature}/{Component}.tsx`

## Important Patterns
- Use custom hooks for business logic (src/hooks/)
- API types are generated from OpenAPI spec (npm run generate-types)
- All forms use react-hook-form with zod validation

## Environment Variables
- `VITE_API_URL` - Backend API endpoint
- `VITE_AUTH_DOMAIN` - Auth0 domain
- `VITE_SENTRY_DSN` - Error tracking

## Common Tasks
- `npm run dev` - Start dev server
- `npm run test` - Run unit tests
- `npm run type-check` - TypeScript validation
- `npm run generate-types` - Update API types from OpenAPI
```

### Context Compression (Auto-Compact)

Managing token limits through intelligent compression:

```javascript
class ContextCompactor {
  async compactHistory(messages, maxTokens) {
    const tokenCount = this.countTokens(messages);
    
    if (tokenCount <= maxTokens) {
      return messages;
    }
    
    // Strategy 1: Summarize old messages
    const cutoffIndex = this.findCutoffPoint(messages, maxTokens);
    const oldMessages = messages.slice(0, cutoffIndex);
    const recentMessages = messages.slice(cutoffIndex);
    
    const summary = await this.summarizeMessages(oldMessages);
    
    return [
      {
        role: 'system',
        content: `## Previous Conversation Summary\n${summary}`
      },
      ...recentMessages
    ];
  }
  
  async summarizeMessages(messages) {
    const prompt = `Summarize this conversation concisely, preserving:
- Key decisions made
- Files modified
- Important discoveries
- Current task status

Conversation:
${messages.map(m => `${m.role}: ${m.content}`).join('\n\n')}`;
    
    const summary = await this.llm.complete({
      messages: [{ role: 'user', content: prompt }],
      max_tokens: 500
    });
    
    return summary.content;
  }
}
```

## Task System and Multi-Agent Patterns

### Task Definition and Execution

```javascript
// Task structure
class Task {
  constructor(config) {
    this.id = config.id;
    this.description = config.description;
    this.status = 'pending'; // pending, in_progress, completed, failed
    this.subtasks = [];
    this.dependencies = [];
    this.assignedAgent = null;
  }
  
  async execute(context) {
    this.status = 'in_progress';
    
    try {
      // Break down into subtasks if complex
      if (this.shouldDecompose()) {
        this.subtasks = await this.decompose();
        
        for (const subtask of this.subtasks) {
          await subtask.execute(context);
        }
      } else {
        // Direct execution
        await this.run(context);
      }
      
      this.status = 'completed';
    } catch (error) {
      this.status = 'failed';
      this.error = error;
      throw error;
    }
  }
  
  shouldDecompose() {
    // Heuristics for task complexity
    return this.estimatedSteps > 5 || 
           this.description.includes(' and ') ||
           this.requiresMultipleTools();
  }
}
```

### Multi-Agent Coordination

```javascript
// Coordinator pattern for multiple specialized agents
class AgentCoordinator {
  constructor() {
    this.agents = {
      coder: new CodingAgent(),
      tester: new TestingAgent(),
      reviewer: new ReviewAgent(),
      debugger: new DebuggingAgent()
    };
  }
  
  async handleComplexTask(task, context) {
    // 1. Planning phase
    const plan = await this.createExecutionPlan(task);
    
    // 2. Delegate to specialized agents
    for (const step of plan.steps) {
      const agent = this.selectAgent(step.type);
      const result = await agent.execute(step, context);
      
      // 3. Collect and integrate results
      plan.results.push(result);
      
      // 4. Update shared context
      context.memory.add({
        step: step.description,
        agent: agent.name,
        result: result.summary
      });
    }
    
    // 5. Final integration
    return this.integrateResults(plan.results);
  }
  
  selectAgent(taskType) {
    const mapping = {
      'implement_feature': this.agents.coder,
      'write_tests': this.agents.tester,
      'code_review': this.agents.reviewer,
      'fix_bug': this.agents.debugger
    };
    
    return mapping[taskType] || this.agents.coder;
  }
}
```

## MCP Protocol - Extension System

The Model Context Protocol enables tool interoperability:

```javascript
// MCP Server implementation
class MCPServer {
  constructor() {
    this.tools = new Map();
    this.resources = new Map();
  }
  
  // Register a tool provider
  registerTool(toolDef) {
    this.tools.set(toolDef.name, {
      definition: {
        name: toolDef.name,
        description: toolDef.description,
        inputSchema: toolDef.inputSchema
      },
      handler: toolDef.handler
    });
  }
  
  // MCP protocol handlers
  async handleToolsList() {
    return {
      tools: Array.from(this.tools.values()).map(t => t.definition)
    };
  }
  
  async handleToolCall(name, args) {
    const tool = this.tools.get(name);
    if (!tool) {
      throw new Error(`Tool not found: ${name}`);
    }
    
    return await tool.handler(args);
  }
  
  // Resource management for context
  async handleResourcesList() {
    return {
      resources: Array.from(this.resources.values())
    };
  }
}

// MCP Client - connecting to external tools
class MCPClient {
  async connectToServer(serverUrl) {
    const response = await fetch(`${serverUrl}/mcp/initialize`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        protocolVersion: '1.0',
        capabilities: ['tools', 'resources']
      })
    });
    
    const { tools, resources } = await response.json();
    
    // Make external tools available to agent
    tools.forEach(tool => this.registerExternalTool(tool));
  }
}
```

## Permission Model

Three-tier permission system for tools:

```javascript
const permissionLevels = {
  // Auto-execute without asking
  AUTO: 'auto',
  
  // Ask user before executing
  ASK: 'ask',
  
  // Never execute, always deny
  DENY: 'deny'
};

class PermissionManager {
  constructor() {
    this.rules = new Map();
    this.userPreferences = {};
  }
  
  async checkPermission(tool, args, context) {
    // 1. Check explicit deny rules
    if (this.isDenied(tool, args)) {
      throw new Error(`Permission denied for tool: ${tool.name}`);
    }
    
    // 2. Check auto-approve rules
    if (this.isAutoApproved(tool, args, context)) {
      return { allowed: true, needsConfirmation: false };
    }
    
    // 3. Require user confirmation
    const allowed = await this.requestUserApproval(tool, args);
    
    // 4. Remember user preference if "always" chosen
    if (allowed.remember) {
      this.updatePreferences(tool, args, allowed.decision);
    }
    
    return { allowed: allowed.decision, needsConfirmation: true };
  }
  
  isAutoApproved(tool, args, context) {
    // Safe read operations
    if (['read_file', 'list_directory', 'search_files'].includes(tool.name)) {
      return true;
    }
    
    // Writes within project directory
    if (tool.name === 'write_file' && 
        this.isWithinProject(args.path, context.projectRoot)) {
      return true;
    }
    
    // User-defined auto-approvals
    return this.userPreferences[tool.name] === 'auto';
  }
}
```

## Skills System

Creating reusable agent behaviors:

```javascript
// Skill definition
const debuggingSkill = {
  name: 'advanced-debugging',
  description: 'Systematic debugging using multiple tools',
  
  async execute(context) {
    // 1. Reproduce the issue
    const reproSteps = await this.reproduceIssue(context);
    
    // 2. Gather diagnostics
    const diagnostics = await context.tools.get_diagnostics();
    
    // 3. Check error logs
    const logs = await context.tools.execute_command({
      command: 'tail -n 100 error.log'
    });
    
    // 4. Analyze stack trace
    const analysis = await this.analyzeStackTrace(logs.stdout);
    
    // 5. Test hypotheses
    for (const hypothesis of analysis.hypotheses) {
      const result = await this.testHypothesis(hypothesis, context);
      if (result.confirmed) {
        return this.proposeFixtrue(hypothesis, context);
      }
    }
    
    return {
      status: 'needs_more_info',
      suggestions: analysis.nextSteps
    };
  }
};
```

## Common Patterns and Best Practices

### Pattern: Incremental Code Changes

```javascript
// Instead of rewriting entire files, use targeted edits
async function incrementalUpdate(file, changes) {
  // Read current content
  const content = await readFile(file);
  
  // Apply line-level edits
  for (const change of changes) {
    await editFile({
      path: file,
      start_line: change.startLine,
      end_line: change.endLine,
      new_content: change.newContent
    });
  }
  
  // Verify changes
  const updated = await readFile(file);
  const diff = createDiff(content, updated);
  
  return { diff, success: true };
}
```

### Pattern: Verification Loop

```javascript
// Always verify actions with tests or checks
async function implementWithVerification(task, context) {
  // 1. Implement
  await task.execute(context);
  
  // 2. Run tests
  const testResult = await context.tools.run_tests({
    pattern: task.testPattern
  });
  
  // 3. Check types
  const typeCheck = await context.tools.execute_command({
    command: 'npm run type-check'
  });
  
  // 4. If failures, attempt fixes
  if (!testResult.success || !typeCheck.success) {
    const fixes = await analyzeFailures(testResult, typeCheck);
    await applyFixes(fixes, context);
    
    // Recursive verification
    return implementWithVerification(task, context);
  }
  
  return { success: true };
}
```

### Pattern: Context-Aware Suggestions

```javascript
// Use project context for better suggestions
async function suggestNextSteps(currentState, context) {
  const suggestions = [];
  
  // Based on project type
  if (context.project.type === 'web-app') {
    if (!currentState.hasTests) {
      suggestions.push('Add unit tests for the new component');
    }
    if (!currentState.hasStorybook) {
      suggestions.push('Create Storybook story for component');
    }
  }
  
  // Based on recent changes
  const recentFiles = currentState.modifiedFiles;
  if (recentFiles.some(f => f.endsWith('.tsx'))) {
    suggestions.push('Run type checking: npm run type-check');
  }
  
  // Based on memory
  const previousIssues = context.memory.query('errors');
  if (previousIssues.length > 0) {
    suggestions.push('Verify previous error is fixed');
  }
  
  return suggestions;
}
```

## Troubleshooting

### Managing Context Window

```javascript
// Monitor and manage token usage
function monitorTokens(context) {
  const usage = context.getTokenUsage();
  
  if (usage.total > usage.limit * 0.8) {
    console.warn('Approaching token limit, compacting context...');
    context.compact();
  }
  
  // Selectively clear old context
  if (usage.total > usage.limit * 0.9) {
    context.clearOldMessages({ keepRecent: 20 });
  }
}
```

### Tool Execution Failures

```javascript
// Robust error handling for tools
async function safeToolExecution(tool, args, context) {
  try {
    return await tool.execute(args, context);
  } catch (error) {
    // Log for debugging
    context.logger.error(`Tool ${tool.name} failed:`, error);
    
    // Try alternative approach
    if (tool.fallback) {
      return await tool.fallback(args, context);
    }
    
    // Return graceful error
    return {
      success: false,
      error: error.message,
      suggestion: `Consider trying ${tool.alternatives.join(' or ')}`
    };
  }
}
```

### Memory Management

```javascript
// Prevent memory from growing unbounded
class ManagedMemory {
  constructor(maxSize = 1000) {
    this.items = [];
    this.maxSize = maxSize;
  }
  
  add(item) {
    this.items.push({
      ...item,
      timestamp: Date.now(),
      importance: this.calculateImportance(item)
    });
    
    // Prune if needed
    if (this.items.length > this.maxSize) {
      this.prune();
    }
  }
  
  prune() {
    // Keep most important and most recent
    this.items.sort((a, b) => {
      const scoreA = a.importance + (Date.now() - a.timestamp) / 1000000;
      const scoreB = b.importance + (Date.now() - b.timestamp) / 1000000;
      return scoreB - scoreA;
    });
    
    this.items = this.items.slice(0, this.maxSize);
  }
}
```

## Key Takeaways

1. **Tool-First Design**: Build agents around concrete tools, not abstract capabilities
2. **Context is King**: Effective system prompts and memory management determine agent quality
3. **Verification Loops**: Always verify actions with tests, checks, or user confirmation
4. **Incremental Actions**: Prefer small, verifiable changes over large rewrites
5. **Layered Permissions**: Balance autonomy with safety through tiered permission model
6. **Extensibility**: Use MCP protocol for tool interoperability and ecosystem growth

This architecture demonstrates how to build production-grade AI agent systems that are reliable, safe, and extensible.
