---
name: how-claude-code-works-analysis
description: Expert guidance on understanding Claude Code internals, architecture patterns, agent loops, context engineering, and AI coding agent design principles
triggers:
  - "how does Claude Code work internally"
  - "explain Claude Code architecture"
  - "understand AI agent design patterns"
  - "implement AI coding agent features"
  - "Claude Code source code analysis"
  - "build coding agent like Claude"
  - "context compression strategies"
  - "AI agent security and permissions"
---

# How Claude Code Works Analysis

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

Expert knowledge of Claude Code's internal architecture, based on deep analysis of its 512,000+ line TypeScript codebase. This skill helps you understand production-grade AI coding agent design, implement similar features, and leverage Claude Code's advanced capabilities.

## What This Project Provides

**how-claude-code-works** is a comprehensive documentation project analyzing Claude Code's source code:

- **15 specialized chapters** covering architecture, agent loops, context engineering, tools, permissions, multi-agent systems, and UX
- **Interactive documentation site** with diagrams and code references
- **Companion implementation project** (claude-code-from-scratch) with working TypeScript/Python examples
- **Production-grade patterns** validated by millions of daily users

## Key Architecture Concepts

### Core Agent Loop

The dual-layer architecture separates concerns:

```typescript
// High-level QueryEngine manages sessions
class QueryEngine {
  async query(userMessage: string) {
    // 1. Add message to history
    // 2. Build context with compression
    // 3. Call Claude API
    // 4. Parse response stream
    // 5. Execute tools in parallel
    // 6. Handle 7 continuation sites
  }
}

// Low-level StreamingToolExecutor handles concurrency
class StreamingToolExecutor {
  // Executes read-only tools in parallel
  // Serializes write operations
  // Pre-executes tools while model is still generating
}
```

**7 Continuation Sites** for fault recovery:

1. **ContextLengthExceeded** → compress and retry
2. **MaxTokensReached** → upgrade from 4K to 64K limit
3. **ToolError** → inject error, continue loop
4. **UserInterrupt** → save state, allow resume
5. **RateLimitHit** → exponential backoff
6. **ValidationFailure** → rollback, re-plan
7. **CompactionNeeded** → trigger compression pipeline

### 4-Level Context Compression Pipeline

When approaching context limits, execute progressively:

```typescript
// Level 1: Trim (fast, 0 API calls)
async function trimCompaction(messages: Message[]): Promise<Message[]> {
  return messages.map(msg => {
    if (msg.role === 'assistant' && msg.toolResults) {
      // Truncate old tool outputs to 1000 chars
      return truncateToolResults(msg, 1000);
    }
    return msg;
  });
}

// Level 2: Dedupe (near-zero cost)
async function dedupeCompaction(messages: Message[]): Promise<Message[]> {
  const seen = new Set<string>();
  return messages.filter(msg => {
    const hash = hashContent(msg);
    if (seen.has(hash)) return false;
    seen.add(hash);
    return true;
  });
}

// Level 3: Fold (preserves original, can unfold)
async function foldCompaction(messages: Message[]): Promise<Message[]> {
  // Collapse inactive conversation segments
  // Keep metadata for potential expansion
}

// Level 4: Summarize (1 API call)
async function summarizeCompaction(messages: Message[]): Promise<Message[]> {
  const summary = await subAgent.summarize(messages);
  // Auto-restore last 5 edited files after summarization
  return [summary, ...restoreRecentEdits()];
}
```

**Critical**: After compression, automatically restore:
- Last 5 edited files (full content)
- Active skills that were in use
- Current working context

### Tool System Architecture

All 66+ tools follow unified interface:

```typescript
interface Tool {
  name: string;
  description: string;
  parameters: ZodSchema;
  execute(params: z.infer<parameters>): Promise<ToolResult>;
  
  // Metadata for execution engine
  readonly: boolean;        // If true, can run in parallel
  requiresApproval: boolean; // Triggers permission check
  tokenBudget?: number;      // Max output size
}

// Example: File read tool
const readFileTool: Tool = {
  name: 'read_file',
  readonly: true,
  requiresApproval: false,
  
  async execute({ path }: { path: string }) {
    const content = await fs.readFile(path, 'utf-8');
    
    // Auto-compact if over 100K chars
    if (content.length > 100_000) {
      await fs.writeFile(`/tmp/${hash(path)}`, content);
      return {
        summary: content.slice(0, 1000),
        fullContentPath: `/tmp/${hash(path)}`,
        instructions: 'Full content saved to file. Use read_file to access.'
      };
    }
    
    return { content };
  }
};
```

### 5-Layer Security Model

Defense in depth for dangerous operations:

```typescript
// Layer 1: Permission modes
enum PermissionMode {
  Full,      // All tools allowed
  Limited,   // Restricted set
  ReadOnly,  // No write/exec
  Ask        // Confirm everything
}

// Layer 2: Rule-based filtering
const rules = {
  allowPatterns: [/^git status$/, /^npm test$/],
  denyPatterns: [/rm -rf \//, /sudo/, /curl .* \| bash/]
};

// Layer 3: AST-based shell analysis (23 checks)
async function analyzeBashCommand(cmd: string): Promise<SecurityRisk[]> {
  const ast = parseShellWithTreeSitter(cmd);
  
  return [
    checkCommandInjection(ast),
    checkEnvVarLeakage(ast),
    checkFileSystemRisk(ast),
    checkNetworkAccess(ast),
    checkPrivilegeEscalation(ast),
    // ... 18 more checks
  ].filter(risk => risk.severity >= 'medium');
}

// Layer 4: User confirmation with debounce
async function confirmDangerousOperation(
  operation: string,
  risks: SecurityRisk[]
): Promise<boolean> {
  // 200ms debounce prevents accidental key repeat
  await sleep(200);
  return await showConfirmDialog(operation, risks);
}

// Layer 5: Hook validation (custom user rules)
async function runPermissionHooks(
  request: PermissionRequest
): Promise<PermissionResult> {
  // Users can inject custom logic via CLAUDE.md hooks
  const customRules = await loadHooks('permission');
  
  for (const hook of customRules) {
    const result = await hook.validate(request);
    if (result.deny) return result;
    if (result.modify) {
      // Can even modify tool inputs (e.g., add --dry-run)
      request.params = result.modifiedParams;
    }
  }
  
  return { allow: true };
}
```

### Multi-Agent Coordination

Three patterns supported:

```typescript
// Pattern 1: Sub-agent (parent waits for child)
async function delegateToSubAgent(task: string) {
  const subAgent = await QueryEngine.createSubAgent({
    task,
    worktree: await git.createWorktree(), // Isolated workspace
    inheritContext: true
  });
  
  const result = await subAgent.run();
  await git.mergeWorktree(subAgent.worktree);
  return result;
}

// Pattern 2: Coordinator (cannot use tools directly)
class CoordinatorAgent extends QueryEngine {
  // Override: coordinator can ONLY delegate, not execute
  async execute(tool: Tool) {
    if (!tool.name.startsWith('delegate_')) {
      throw new Error('Coordinators must delegate all work');
    }
    return super.execute(tool);
  }
}

// Pattern 3: Swarm (peer-to-peer messaging)
class SwarmAgent extends QueryEngine {
  async sendMessage(toAgent: string, message: string) {
    await messageBus.publish(toAgent, {
      from: this.id,
      content: message,
      timestamp: Date.now()
    });
  }
  
  async receiveMessages(): Promise<Message[]> {
    return await messageBus.poll(this.id);
  }
}
```

## Configuration via CLAUDE.md

Place in project root to customize behavior:

```markdown
# Project Context

This is a TypeScript monorepo using Bun and React.

## Custom Tools

### run_integration_tests
Runs full test suite with real API calls (takes ~5min).
Requires: API_KEY environment variable set.

## Permission Rules

- ALLOW: npm test, npm run build
- DENY: npm publish (manual only)
- REQUIRE_CONFIRM: database migrations

## Skills Priority

1. Use Zod for all validation
2. Prefer Bun APIs over Node.js when available
3. Always run formatter before commit

## Memory

REMEMBER: API rate limit is 100 req/min. Batch operations when possible.
```

## Common Patterns

### Streaming with Tool Pre-execution

```typescript
async function streamingQuery(userMessage: string) {
  const stream = await claude.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    messages: [{ role: 'user', content: userMessage }],
    tools: allTools,
    stream: true
  });
  
  const toolExecutor = new StreamingToolExecutor();
  
  for await (const chunk of stream) {
    // Display to user immediately
    process.stdout.write(chunk.delta.text || '');
    
    // Parse tool calls AS THEY ARRIVE (not after completion)
    if (chunk.delta.type === 'tool_use') {
      // Pre-execute while model still generating
      toolExecutor.queue(chunk.delta);
    }
  }
  
  // Wait for all queued tools
  const results = await toolExecutor.waitAll();
  return results;
}
```

### Context Budget Management

```typescript
interface ContextBudget {
  total: 200_000; // Model limit
  
  allocation: {
    systemPrompt: 5000,      // Fixed
    claudeMd: 3000,          // Project config
    gitStatus: 1000,         // Dynamic
    skills: 15000,           // Lazy-loaded
    conversation: 176000     // Remaining for messages
  };
}

async function buildContext(budget: ContextBudget): Promise<Message[]> {
  const messages: Message[] = [];
  
  // System prompt (always included)
  messages.push(await loadSystemPrompt()); // ~5K tokens
  
  // CLAUDE.md (if exists, cache-eligible)
  const claudeMd = await loadClaudeMd();
  if (claudeMd) messages.push(claudeMd); // ~3K tokens
  
  // Skills (lazy load based on triggers)
  const activeSkills = await matchSkills(conversationHistory);
  messages.push(...activeSkills.slice(0, budget.allocation.skills));
  
  // Conversation (compress if needed)
  let conversation = conversationHistory;
  while (tokenCount(conversation) > budget.allocation.conversation) {
    conversation = await compress(conversation); // 4-level pipeline
  }
  messages.push(...conversation);
  
  return messages;
}
```

### Edit-Before-Read Enforcement

```typescript
// Critical pattern: Always read before editing
class EditFileTool implements Tool {
  async execute({ path, edits }: EditParams) {
    // FORCE read current content first
    const current = await fs.readFile(path, 'utf-8');
    
    // Validate search strings exist (anti-hallucination)
    for (const edit of edits) {
      const occurrences = countOccurrences(current, edit.search);
      
      if (occurrences === 0) {
        throw new Error(`Search string not found: ${edit.search}`);
      }
      
      if (occurrences > 1) {
        throw new Error(
          `Search string not unique (found ${occurrences} times): ${edit.search}`
        );
      }
    }
    
    // Apply edits
    let result = current;
    for (const edit of edits) {
      result = result.replace(edit.search, edit.replace);
    }
    
    await fs.writeFile(path, result);
    return { success: true, linesChanged: edits.length };
  }
}
```

## Advanced Usage

### Custom Hook Implementation

```typescript
// .claude/hooks/prevent-friday-deploys.ts
export default {
  name: 'prevent-friday-deploys',
  event: 'permission:shell',
  
  async handle(request: PermissionRequest): Promise<HookResult> {
    const isFriday = new Date().getDay() === 5;
    const isDeploy = request.command.includes('deploy');
    
    if (isFriday && isDeploy) {
      return {
        deny: true,
        reason: 'No deploys on Friday (team policy)',
        suggestion: 'Schedule for Monday or run with --force-friday flag'
      };
    }
    
    return { allow: true };
  }
};
```

### Prompt Caching Strategy

```typescript
// Mark cacheable content (static across conversation)
const systemPrompt = {
  role: 'system',
  content: [
    {
      type: 'text',
      text: await loadSystemPrompt(),
      cache_control: { type: 'ephemeral' } // Cache this
    }
  ]
};

const claudeMd = {
  role: 'system', 
  content: [
    {
      type: 'text',
      text: await fs.readFile('CLAUDE.md'),
      cache_control: { type: 'ephemeral' } // Cache this too
    }
  ]
};

// Detect cache breaks (invalidation)
function detectCacheBreak(oldMessages: Message[], newMessages: Message[]) {
  // If cacheable content changed, cache is invalidated
  const oldCached = oldMessages.filter(m => m.cache_control);
  const newCached = newMessages.filter(m => m.cache_control);
  
  return !isEqual(oldCached, newCached);
}
```

## Troubleshooting

### Context Length Exceeded

```typescript
// Symptom: "maximum context length exceeded" error
// Solution: Compression pipeline didn't run or failed

// Debug: Check which level failed
const debug = {
  async diagnoseCompression() {
    console.log('Level 1 (trim):', await estimateTokenSavings(trimCompaction));
    console.log('Level 2 (dedupe):', await estimateTokenSavings(dedupeCompaction));
    console.log('Level 3 (fold):', await estimateTokenSavings(foldCompaction));
    console.log('Level 4 (summarize):', await estimateTokenSavings(summarizeCompaction));
  }
};

// Fix: Force aggressive compression
await forceCompaction(messages, { target: 150_000 });
```

### Tools Not Executing

```typescript
// Symptom: Model generates tool calls but nothing happens
// Cause: Permission denied or tool not registered

// Debug:
const registeredTools = queryEngine.getTools();
console.log('Registered:', registeredTools.map(t => t.name));

const permissionMode = await getPermissionMode();
console.log('Mode:', permissionMode);

// Fix: Check permission mode
if (permissionMode === PermissionMode.ReadOnly) {
  await setPermissionMode(PermissionMode.Ask); // Prompt for confirmation
}
```

### Memory Not Persisting

```typescript
// Symptom: Agent forgets previous conversations
// Cause: Memory extraction failed or not triggered

// Debug: Check memory system
const memories = await memorySystem.search('project structure');
console.log('Found memories:', memories.length);

// Fix: Manually trigger extraction
await memorySystem.extract({
  type: 'project',
  content: 'This is a React app using Vite',
  importance: 0.9
});
```

### Slow Startup

```typescript
// Symptom: Agent takes >5 seconds to start
// Cause: Serial initialization instead of parallel

// Fix: Ensure parallel initialization
await Promise.all([
  loadSystemPrompt(),
  loadClaudeMd(),
  initializeGit(),
  connectMCPServers(),
  loadSkills(),
  // ... all independent tasks
]);
```

## Resources

- **Documentation Site**: https://windy3f3f3f3f.github.io/how-claude-code-works/
- **Source Analysis**: https://github.com/Windy3f3f3f3f/how-claude-code-works
- **Implementation Guide**: https://github.com/Windy3f3f3f3f/claude-code-from-scratch
- **15 Deep-Dive Chapters**: Architecture, Agent Loop, Context Engineering, Tools, Skills, Memory, Hooks, Multi-Agent, Plan Mode, Editing, Tasks, Permissions, Prompts, UX, Minimal Components

## Environment Variables

```bash
# Required for API access
ANTHROPIC_API_KEY=sk-ant-...

# Optional: Custom model
CLAUDE_MODEL=claude-3-5-sonnet-20241022

# Optional: Debug mode
DEBUG_AGENT_LOOP=true
DEBUG_COMPRESSION=true
DEBUG_TOOLS=true
```

This skill enables deep understanding and implementation of production-grade AI coding agent patterns used by millions of developers.
