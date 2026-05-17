---
name: dive-into-claude-code-analysis
description: Analyze and apply architectural patterns from Claude Code's system design for building AI agent systems
triggers:
  - how does Claude Code architecture work
  - show me Claude Code design patterns
  - apply Claude Code safety principles
  - implement agent loop like Claude Code
  - use Claude Code permission system design
  - analyze AI agent architecture patterns
  - design agent system with Claude Code principles
  - implement defense in depth for AI agents
---

# Dive into Claude Code Analysis

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

This skill provides expertise in understanding and applying the architectural patterns, design principles, and implementation strategies documented in the VILA-Lab "Dive into Claude Code" analysis — a comprehensive study of Claude Code v2.1.88 (~512K lines of TypeScript across 1,884 files).

## What This Project Provides

The Dive into Claude Code project is a systematic analysis that reveals:

- **Architectural blueprint**: How Claude Code structures its 98.4% deterministic infrastructure around 1.6% AI decision logic
- **Design principles**: 5 core values → 13 design principles → concrete implementation patterns
- **Safety patterns**: 7-layer defense-in-depth permission system with graduated trust spectrum
- **Context management**: 5-stage compaction pipeline for token budget management
- **Agent loop implementation**: ReAct-pattern while-loop with streaming execution and graceful recovery
- **Extensibility mechanisms**: 4 graduated extension points (hooks, skills, plugins, MCP)

## Installation

```bash
# Clone the repository
git clone https://github.com/VILA-Lab/Dive-into-Claude-Code.git
cd Dive-into-Claude-Code

# Read the paper (PDF in repo or arXiv)
open paper/Dive_into_Claude_Code.pdf
# or visit https://arxiv.org/abs/2604.14228
```

## Core Architecture Patterns

### The 98/2 Split

Claude Code demonstrates that production AI agents are primarily infrastructure:

- **1.6%**: AI decision logic (LLM reasoning, tool selection)
- **98.4%**: Deterministic harness (permissions, context management, recovery, routing)

**Key insight**: The agent loop is a simple while-loop. The engineering complexity lives in the systems around it.

### Five Architectural Layers

```
┌─────────────────────────────────────────┐
│ User & Interfaces (CLI, SDK, IDE)      │
├─────────────────────────────────────────┤
│ Agent Loop (queryLoop)                  │
│  • Context Assembly                     │
│  • Model Invocation                     │
│  • Tool Dispatch                        │
├─────────────────────────────────────────┤
│ Permission System (7 modes, deny-first) │
├─────────────────────────────────────────┤
│ Tools & Extensions (54 base + MCP)     │
├─────────────────────────────────────────┤
│ State & Execution Environment          │
└─────────────────────────────────────────┘
```

## Design Principles Framework

### The 5 Core Values

```typescript
// Values that drive all architectural decisions
enum CoreValue {
  HumanDecisionAuthority = "human-control",
  SafetySecurityPrivacy = "safe-by-default", 
  ReliableExecution = "does-what-meant",
  CapabilityAmplification = "unix-utility-philosophy",
  ContextualAdaptability = "evolves-with-trust"
}
```

### The 13 Design Principles

Apply these when designing your AI agent:

1. **Deny-first with human escalation**: Unrecognized actions → escalate to human
2. **Graduated trust spectrum**: 7 permission modes users traverse over time
3. **Defense in depth**: Multiple overlapping safety layers
4. **Externalized programmable policy**: Configs with lifecycle hooks
5. **Context as scarce resource**: 5-stage graduated compaction pipeline
6. **Append-only durable state**: Immutable history logs
7. **Minimal scaffolding, maximal harness**: Invest in operational infrastructure
8. **Values over rules**: Contextual judgment + deterministic guardrails
9. **Composable multi-mechanism extensibility**: 4 extension layers at different costs
10. **Reversibility-weighted risk assessment**: Lighter oversight for reversible actions
11. **Transparent file-based config**: User-visible files over opaque DBs
12. **Isolated subagent boundaries**: Separate context/permissions per subagent
13. **Graceful recovery and resilience**: Silent recovery over hard failures

## Implementing the Agent Loop Pattern

### Basic ReAct Loop Structure

```typescript
async function* agentLoop(
  initialPrompt: string,
  context: AgentContext,
  permissions: PermissionSystem
): AsyncGenerator<AgentEvent> {
  
  let turnCount = 0;
  const MAX_TURNS = 25;
  const conversationHistory: Message[] = [];
  
  while (turnCount < MAX_TURNS) {
    // 1. Context assembly
    const assembledContext = await assembleContext(
      conversationHistory,
      context.workspaceState,
      context.availableTools
    );
    
    // 2. Five-stage compaction (if needed)
    const compactedContext = await compactIfNeeded(
      assembledContext,
      context.tokenBudget
    );
    
    // 3. Model invocation
    const modelResponse = await callModel({
      messages: compactedContext.messages,
      tools: compactedContext.availableTools,
      streaming: true
    });
    
    // 4. Tool dispatch
    const toolCalls = extractToolCalls(modelResponse);
    
    if (toolCalls.length === 0) {
      yield { type: 'complete', message: modelResponse.content };
      break;
    }
    
    // 5. Permission gate
    const authorizedCalls = await permissions.authorize(toolCalls);
    
    // 6. Execute with streaming
    for await (const toolResult of executeTools(authorizedCalls)) {
      yield { type: 'tool_result', data: toolResult };
      conversationHistory.push({
        role: 'tool',
        content: toolResult.output
      });
    }
    
    turnCount++;
  }
}
```

### Five-Stage Compaction Pipeline

```typescript
// Run sequentially, cheapest-first
async function compactIfNeeded(
  context: AssembledContext,
  tokenBudget: number
): Promise<CompactedContext> {
  
  let current = context;
  const currentTokens = estimateTokens(current);
  
  if (currentTokens <= tokenBudget) {
    return current;
  }
  
  // Stage 1: Budget Reduction (remove low-priority items)
  current = await budgetReduction(current, tokenBudget);
  if (estimateTokens(current) <= tokenBudget) return current;
  
  // Stage 2: Snip (truncate long individual messages)
  current = await snipLongMessages(current, tokenBudget);
  if (estimateTokens(current) <= tokenBudget) return current;
  
  // Stage 3: Microcompact (remove whitespace, comments)
  current = await microcompact(current);
  if (estimateTokens(current) <= tokenBudget) return current;
  
  // Stage 4: Context Collapse (merge related messages)
  current = await contextCollapse(current);
  if (estimateTokens(current) <= tokenBudget) return current;
  
  // Stage 5: Auto-Compact (LLM-based summarization)
  current = await autoCompact(current, tokenBudget);
  
  return current;
}
```

## Permission System Implementation

### Seven Permission Modes (Graduated Trust)

```typescript
enum PermissionMode {
  Plan = "plan",              // Show what would happen, no execution
  Default = "default",        // Ask for every action
  AcceptEdits = "acceptEdits", // Auto-approve file edits only
  Auto = "auto",              // ML classifier decides
  DontAsk = "dontAsk",        // Auto-approve in current directory
  BypassPermissions = "bypassPermissions", // Trust completely
  Bubble = "bubble"           // Internal: defer to parent
}
```

### Deny-First Authorization Pipeline

```typescript
interface PermissionRule {
  scope: 'global' | 'directory' | 'file';
  pattern: string;
  decision: 'allow' | 'deny' | 'ask';
  priority: number;
}

class DenyFirstPermissionSystem {
  
  async authorize(toolCalls: ToolCall[]): Promise<ToolCall[]> {
    const authorized: ToolCall[] = [];
    
    for (const call of toolCalls) {
      // 1. Pre-filtering: strip denied tools entirely
      if (this.isDeniedTool(call.tool)) {
        continue;
      }
      
      // 2. Evaluate rules (deny-first: broadest deny wins)
      const decision = this.evaluateRules(call);
      
      if (decision === 'deny') {
        continue;
      }
      
      if (decision === 'ask') {
        const userApproved = await this.promptUser(call);
        if (!userApproved) continue;
      }
      
      // 3. Execute hooks (can still block)
      const hookResult = await this.runPreToolUseHooks(call);
      if (hookResult.blocked) {
        continue;
      }
      
      authorized.push(call);
    }
    
    return authorized;
  }
  
  private evaluateRules(call: ToolCall): 'allow' | 'deny' | 'ask' {
    const matchingRules = this.rules
      .filter(rule => this.matches(rule, call))
      .sort((a, b) => b.priority - a.priority);
    
    // Deny-first: any deny rule wins
    const denyRule = matchingRules.find(r => r.decision === 'deny');
    if (denyRule) return 'deny';
    
    const allowRule = matchingRules.find(r => r.decision === 'allow');
    if (allowRule) return 'allow';
    
    return 'ask'; // Default: escalate to human
  }
}
```

### Seven Independent Safety Layers

```typescript
class DefenseInDepth {
  
  async executeTool(call: ToolCall, context: ExecutionContext): Promise<ToolResult> {
    
    // Layer 1: Tool pre-filtering (denied tools removed from tool pool)
    // (already applied during context assembly)
    
    // Layer 2: PreToolUse hooks
    await this.runHooks('PreToolUse', call);
    
    // Layer 3: Deny-first rule evaluation
    const permissionDecision = await this.permissions.authorize([call]);
    if (permissionDecision.length === 0) {
      throw new Error('Permission denied');
    }
    
    // Layer 4: Auto-mode classifier (if in auto mode)
    if (context.mode === PermissionMode.Auto) {
      const classifierResult = await this.classifier.evaluate(call);
      if (!classifierResult.safe) {
        const userOverride = await this.promptUser(call);
        if (!userOverride) throw new Error('Classifier rejected');
      }
    }
    
    // Layer 5: Reversibility assessment
    const risk = this.assessReversibility(call);
    if (risk === 'irreversible' && !call.userApproved) {
      throw new Error('Irreversible action requires explicit approval');
    }
    
    // Layer 6: Resource limits (timeout, memory, file size)
    const result = await this.executeWithLimits(call, {
      timeout: 30000,
      maxMemory: 512 * 1024 * 1024,
      maxFileSize: 10 * 1024 * 1024
    });
    
    // Layer 7: PostToolUse hooks (can still intervene)
    await this.runHooks('PostToolUse', call, result);
    
    return result;
  }
}
```

## Extensibility Patterns

### Four Graduated Extension Mechanisms

```typescript
// Cost spectrum: Hooks < Skills < Plugins < MCP

// 1. HOOKS (zero context cost)
interface Hook {
  event: HookEvent; // 27 available events
  handler: string | Function;
  blocking?: boolean;
}

// Example: PreToolUse hook
const securityHook: Hook = {
  event: 'PreToolUse',
  handler: async (call: ToolCall) => {
    if (call.tool === 'execute_command' && call.args.command.includes('rm -rf')) {
      return { allow: false, reason: 'Dangerous command blocked' };
    }
  },
  blocking: true
};

// 2. SKILLS (low context cost - injected only when triggered)
interface Skill {
  name: string;
  description: string;
  triggers: string[];
  content: string; // Markdown with examples
}

// 3. PLUGINS (medium cost - loaded at startup)
interface PluginManifest {
  name: string;
  components: {
    commands?: CommandDefinition[];
    agents?: AgentDefinition[];
    skills?: Skill[];
    hooks?: Hook[];
    mcpServers?: MCPServerConfig[];
    settings?: Setting[];
  };
}

// 4. MCP (high cost - persistent tool pool expansion)
interface MCPServerConfig {
  name: string;
  command: string;
  args?: string[];
  env?: Record<string, string>;
}
```

### Tool Pool Assembly

```typescript
async function assembleToolPool(context: AgentContext): Promise<Tool[]> {
  
  // Step 1: Base enumeration (up to 54 tools)
  let tools = [...BASE_TOOLS];
  
  // Step 2: Mode filtering (e.g., plan mode removes write tools)
  tools = filterByMode(tools, context.mode);
  
  // Step 3: Deny pre-filtering (remove denied tools)
  tools = tools.filter(tool => !isDenied(tool, context.permissions));
  
  // Step 4: MCP integration (add external tools)
  const mcpTools = await loadMCPTools(context.mcpServers);
  tools.push(...mcpTools);
  
  // Step 5: Deduplication (by tool name)
  tools = deduplicateTools(tools);
  
  return tools;
}
```

## Session Persistence Pattern

### Append-Only State Management

```typescript
interface SessionState {
  id: string;
  createdAt: number;
  events: SessionEvent[]; // Append-only
  metadata: {
    workspaceRoot: string;
    permissionMode: PermissionMode;
    trustLevel: number;
  };
}

class SessionPersistence {
  
  // Never mutate history - only append
  async appendEvent(sessionId: string, event: SessionEvent): Promise<void> {
    const session = await this.loadSession(sessionId);
    session.events.push({
      ...event,
      timestamp: Date.now(),
      sequenceNumber: session.events.length
    });
    await this.saveSession(session);
  }
  
  // Replay from events for recovery
  async replaySession(sessionId: string): Promise<AgentContext> {
    const session = await this.loadSession(sessionId);
    let context = this.createInitialContext(session.metadata);
    
    for (const event of session.events) {
      context = await this.applyEvent(context, event);
    }
    
    return context;
  }
}
```

## Configuration Patterns

### Transparent File-Based Config

```typescript
// .claude/config.json
interface ClaudeConfig {
  permissionMode: PermissionMode;
  customInstructions?: string;
  deniedTools?: string[];
  allowedDirectories?: string[];
  hooks?: Hook[];
  mcpServers?: MCPServerConfig[];
}

// CLAUDE.md hierarchy (most specific wins)
// workspace/subdir/CLAUDE.md > workspace/CLAUDE.md > ~/.claude/CLAUDE.md

async function loadConfig(workspacePath: string): Promise<ClaudeConfig> {
  const configs: ClaudeConfig[] = [];
  
  // Load from most general to most specific
  const homeClaude = await loadIfExists('~/.claude/CLAUDE.md');
  if (homeClaude) configs.push(homeClaude);
  
  const workspaceClaude = await loadIfExists(path.join(workspacePath, 'CLAUDE.md'));
  if (workspaceClaude) configs.push(workspaceClaude);
  
  const configJson = await loadIfExists(path.join(workspacePath, '.claude/config.json'));
  if (configJson) configs.push(configJson);
  
  // Merge (most specific wins)
  return mergeConfigs(configs);
}
```

## Subagent Delegation Pattern

### Isolated Context Boundaries

```typescript
interface SubagentConfig {
  isolatedContext: boolean; // Always true in Claude Code
  inheritPermissions: boolean; // Always false
  parentVisibility: 'none' | 'summary' | 'full';
}

async function delegateToSubagent(
  task: string,
  parentContext: AgentContext
): Promise<SubagentResult> {
  
  // Create isolated context (never inherits permissions)
  const subagentContext: AgentContext = {
    workspaceRoot: parentContext.workspaceRoot,
    permissionMode: PermissionMode.Default, // Start at lowest trust
    conversationHistory: [], // Empty history
    availableTools: assembleToolPool({ /* fresh context */ }),
    tokenBudget: parentContext.tokenBudget,
    parentSessionId: null // No access to parent
  };
  
  // Run isolated agent loop
  const result = await runAgentLoop(task, subagentContext);
  
  // Return only summary to parent (not full context)
  return {
    summary: result.summary,
    outcome: result.outcome,
    // Full conversation history NOT returned
  };
}
```

## Graceful Recovery Patterns

### Five Recovery Strategies

```typescript
class RecoverySystem {
  
  async executeWithRecovery(call: ToolCall): Promise<ToolResult> {
    
    try {
      return await this.execute(call);
      
    } catch (error) {
      
      // Strategy 1: Max output token escalation
      if (error.code === 'OUTPUT_TOO_LONG') {
        return await this.retryWithIncreasedTokens(call, {
          attempt: 1,
          maxAttempts: 3,
          tokenMultiplier: 2
        });
      }
      
      // Strategy 2: Reactive compaction
      if (error.code === 'CONTEXT_TOO_LONG') {
        const compacted = await this.compactContext();
        return await this.execute(call);
      }
      
      // Strategy 3: Streaming fallback
      if (error.code === 'STREAMING_FAILED') {
        return await this.executeNonStreaming(call);
      }
      
      // Strategy 4: Fallback model
      if (error.code === 'MODEL_UNAVAILABLE') {
        return await this.executeWithFallbackModel(call);
      }
      
      // Strategy 5: Graceful degradation
      if (error.code === 'TOOL_UNAVAILABLE') {
        return {
          success: false,
          message: `Tool ${call.tool} unavailable, continuing with reduced capabilities`
        };
      }
      
      throw error; // Unrecoverable
    }
  }
}
```

## Common Patterns from the Analysis

### Pattern: Context Assembly with Hooks

```typescript
async function assemble(context: AgentContext): Promise<AssembledContext> {
  
  // 1. Run PreContextAssembly hooks
  await runHooks('PreContextAssembly', context);
  
  // 2. Gather base context
  let assembled = {
    messages: [...context.conversationHistory],
    tools: await assembleToolPool(context),
    systemPrompt: await loadSystemPrompt(context)
  };
  
  // 3. Inject custom instructions (CLAUDE.md)
  const customInstructions = await loadCustomInstructions(context.workspaceRoot);
  if (customInstructions) {
    assembled.systemPrompt += '\n\n' + customInstructions;
  }
  
  // 4. Inject skills (triggered by user message)
  const triggeredSkills = await findTriggeredSkills(
    context.lastUserMessage,
    context.availableSkills
  );
  for (const skill of triggeredSkills) {
    assembled.messages.push({
      role: 'system',
      content: skill.content
    });
  }
  
  // 5. Run PostContextAssembly hooks
  assembled = await runHooks('PostContextAssembly', assembled);
  
  return assembled;
}
```

### Pattern: Streaming Tool Execution

```typescript
async function* streamingToolExecutor(
  toolCalls: ToolCall[],
  executor: ToolExecutor
): AsyncGenerator<ToolResult> {
  
  // Classify as concurrent-safe or exclusive
  const { concurrent, exclusive } = classifyTools(toolCalls);
  
  // Execute concurrent tools in parallel
  const concurrentPromises = concurrent.map(call => executor.execute(call));
  
  for await (const result of yieldAsCompleted(concurrentPromises)) {
    yield result;
  }
  
  // Execute exclusive tools sequentially
  for (const call of exclusive) {
    const result = await executor.execute(call);
    yield result;
  }
}

function classifyTools(calls: ToolCall[]): { concurrent: ToolCall[], exclusive: ToolCall[] } {
  const concurrent: ToolCall[] = [];
  const exclusive: ToolCall[] = [];
  
  for (const call of calls) {
    // File writes, shell commands → exclusive
    if (['write_file', 'execute_command', 'apply_diff'].includes(call.tool)) {
      exclusive.push(call);
    } else {
      concurrent.push(call);
    }
  }
  
  return { concurrent, exclusive };
}
```

### Pattern: Auto-Mode Classifier

```typescript
interface ClassifierResult {
  safe: boolean;
  reasoning: string;
  confidence: number;
}

async function classifyToolCall(call: ToolCall): Promise<ClassifierResult> {
  
  // Fast-filter heuristics (avoid LLM call if obviously safe/unsafe)
  if (isTriviallyReversible(call)) {
    return { safe: true, reasoning: 'Read-only operation', confidence: 1.0 };
  }
  
  if (isObviouslyDangerous(call)) {
    return { safe: false, reasoning: 'Irreversible system modification', confidence: 1.0 };
  }
  
  // Chain-of-thought classifier
  const prompt = `
Analyze this tool call for safety:

Tool: ${call.tool}
Arguments: ${JSON.stringify(call.args, null, 2)}
Context: ${call.context}

Consider:
1. Is this operation reversible?
2. Does it modify system-critical files?
3. Could it leak sensitive data?
4. Is the scope appropriate for the task?

Think step-by-step, then answer: SAFE or UNSAFE
`;
  
  const response = await callClassifierModel(prompt);
  
  return {
    safe: response.includes('SAFE') && !response.includes('UNSAFE'),
    reasoning: response,
    confidence: extractConfidence(response)
  };
}
```

## Security Considerations

### Pre-Trust Execution Window (CVE Pattern)

```typescript
// ANTI-PATTERN: Executing code before trust established

// ❌ Vulnerable: Extension runs during init
async function initializeWorkspace() {
  await loadMCPServers(); // Executes server processes
  await runHooks('WorkspaceInit'); // Runs arbitrary code
  
  // Trust dialog appears AFTER extensions already ran
  await promptForTrust();
}

// ✅ Safe: Defer privileged operations until after trust
async function initializeWorkspace() {
  await promptForTrust();
  
  // Only after user approval:
  if (trusted) {
    await loadMCPServers();
    await runHooks('WorkspaceInit');
  }
}
```

### Shared Failure Modes in Defense-in-Depth

```typescript
// WARNING: Multiple safety layers share performance constraints

async function analyzeCommand(command: string): Promise<SecurityAnalysis> {
  
  const subcommands = parseSubcommands(command);
  
  // If command too complex (>50 subcommands), ALL layers degrade:
  if (subcommands.length > 50) {
    // Layer 1: Security analysis skipped (event loop starvation)
    // Layer 2: Classifier times out
    // Layer 3: Hooks don't run
    // Result: Command executes with minimal oversight
    
    return { 
      analyzed: false, 
      reason: 'Command too complex for analysis',
      fallbackToPermissionMode: true 
    };
  }
  
  return await fullSecurityAnalysis(command);
}
```

## Troubleshooting

### Context Overflow Despite Compaction

```typescript
// Check compaction effectiveness
async function debugContextOverflow(context: AssembledContext) {
  
  console.log('Token breakdown:');
  console.log('  System prompt:', estimateTokens(context.systemPrompt));
  console.log('  Conversation:', estimateTokens(context.messages));
  console.log('  Tools:', estimateTokens(context.tools));
  
  // Check if a single message is too large
  const largeMessages = context.messages.filter(m => 
    estimateTokens(m.content) > 10000
  );
  
  if (largeMessages.length > 0) {
    console.log('Large messages detected - consider manual snipping');
  }
  
  // Verify compaction stages ran
  if (!context.compactionMetadata) {
    console.log('WARNING: Compaction metadata missing - pipeline may not have run');
  }
}
```

### Permission Denied Despite Allow Rules

```typescript
// Debug deny-first rule resolution
function debugPermissionDenial(call: ToolCall, rules: PermissionRule[]) {
  
  const matchingRules = rules.filter(r => matches(r, call));
  
  console.log(`Matching rules for ${call.tool}:`);
  matchingRules.forEach(rule => {
    console.log(`  [${rule.decision}] ${rule.scope}:${rule.pattern} (priority: ${rule.priority})`);
  });
  
  const denyRule = matchingRules.find(r => r.decision === 'deny');
  if (denyRule) {
    console.log(`\n❌ DENIED by rule: ${denyRule.pattern}`);
    console.log('   Deny-first policy means this overrides all allow rules');
  }
}
```

### Subagent Not Inheriting Expected Behavior

```typescript
// Remember: Subagents are ALWAYS isolated
function debugSubagentIsolation() {
  
  console.log('Subagent isolation checklist:');
  console.log('  ❌ Does NOT inherit parent permissions');
  console.log('  ❌ Does NOT see parent conversation history');
  console.log('  ❌ Does NOT share parent context');
  console.log('  ✅ DOES start at default permission mode');
  console.log('  ✅ DOES get fresh tool pool');
  
  console.log('\nIf you need shared context, use Skills instead of Subagents');
}
```

## Further Reading

- **Full Paper**: [arXiv:2604.14228](https://arxiv.org/abs/2604.14228)
- **Architecture Deep Dive**: `docs/architecture.md` in repo
- **Design Guide**: `docs/build-your-own-agent.md`
- **Cross-System Comparison**: Claude Code vs OpenClaw vs Hermes-Agent analysis
- **Community Projects**: Curated list of related research and implementations

## Key Takeaways for Agent Builders

1. **Invest in infrastructure, not scaffolding**: The 98/2 split is intentional
2. **Context is your bottleneck**: Plan your compaction strategy early
3. **Deny-first prevents surprises**: Explicit allow lists scale better than deny lists
4. **Defense-in-depth shares failure modes**: Test your safety layers under degraded conditions
5. **Isolated subagents prevent permission escalation**: Never share context/permissions
6. **Append-only state enables time-travel debugging**: Make history immutable
7. **Hooks inject at zero context cost**: Use them for cross-cutting concerns
8. **Graduated trust beats binary trust**: Users traverse a spectrum over time

---

**License**: CC-BY-NC-SA-4.0 (per upstream project)

**Repository**: https://github.com/VILA-Lab/Dive-into-Claude-Code
