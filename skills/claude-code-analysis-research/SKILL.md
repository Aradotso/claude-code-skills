---
name: claude-code-analysis-research
description: Expert in analyzing the leaked Claude Code TypeScript source and architecture
triggers:
  - "analyze Claude Code source code"
  - "explain Claude Code architecture"
  - "how does Claude Code implement memory"
  - "Claude Code security analysis"
  - "what tools does Claude Code use"
  - "Claude Code MCP integration"
  - "compare Claude Code to Cursor"
  - "Claude Code sandbox mechanism"
---

# Claude Code Analysis Research

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## Overview

The `claude-code-analysis` project is a comprehensive analysis of Anthropic's Claude Code leaked TypeScript source (513,237 lines across 1,902 files). On March 31, 2026, security researcher Chaofan Shou discovered that Anthropic's npm package included source maps, exposing the complete codebase.

This skill enables agents to:
- Navigate the leaked source structure and architecture
- Understand Claude Code's memory, tool call, and MCP implementations
- Analyze security mechanisms (sandbox, permissions)
- Compare Claude Code to competitors (Cursor, Aider, Codex)
- Reference implementation patterns and code evidence

## Project Structure

```
claude-code-analysis/
├── README.md           # Main index
├── analysis/           # Analysis documents
│   ├── 01-architecture-overview.md
│   ├── 02-security-analysis.md
│   ├── 04-agent-memory.md
│   ├── 04b-tool-call-implementation.md
│   ├── 04c-skills-implementation.md
│   ├── 04d-mcp-implementation.md
│   ├── 04e-sandbox-implementation.md
│   ├── 04f-context-management.md
│   ├── 04g-prompt-management.md
│   ├── 04h-multi-agent.md
│   ├── 04i-session-storage-resume.md
│   ├── 05-differentiators-and-comparison.md
│   ├── 06-extra-findings.md
│   ├── 06b-negative-keyword-analysis.md
│   ├── 07-code-evidence-index.md
│   ├── 08-competitive-comparison.md
│   ├── 09-final-summary.md
│   ├── 10-src-file-tree.md
│   ├── 11-hidden-features-and-easter-eggs.md
│   └── components/     # UI component analysis
├── src.zip             # Source archive
└── src/                # Full TypeScript source
```

## Key Architecture Components

### Entry Points

```typescript
// Main CLI entry: entrypoints/cli.tsx
// Initialization: init.ts, setup.ts
// Core execution: query.ts, QueryEngine.ts
```

The application flow:

1. **CLI/Entry** → `entrypoints/cli.tsx`, `main.tsx`
2. **Initialization** → `init.ts`, `setup.ts`
3. **Command Layer** → `commands.ts`, `PromptInput`
4. **TUI/REPL** → `App`, `REPL`, `Messages`
5. **Execution Kernel** → `query.ts`, `QueryEngine.ts`
6. **Core Systems**:
   - Tool/Permission: `Tool.ts`, orchestration
   - Memory: `sessionStorage`, `memdir`, `SessionMem`
   - Extensions: MCP, Plugins, Remote, Swarm

### Memory System

Claude Code implements a multi-layer memory architecture:

```typescript
// Session storage in src/sessionStorage/
interface SessionMemory {
  transcript: Message[];
  context: Context;
  compressedHistory: CompressedEntry[];
}

// Memory directories: src/memdir/
// - Short-term: active session
// - Long-term: compressed/indexed
// - Resume: persistent session recovery
```

**Key patterns**:
- Hierarchical storage (session → compressed → indexed)
- Automatic context pruning based on token limits
- Session resumption via `Transcript` persistence

### Tool Call Mechanism

```typescript
// Tool definitions in src/tools/
interface Tool {
  name: string;
  description: string;
  inputSchema: JSONSchema;
  execute: (params: unknown) => Promise<ToolResult>;
}

// Tool orchestration in src/orchestration/
class ToolOrchestrator {
  async executeTool(
    toolName: string,
    params: Record<string, unknown>,
    permissions: PermissionSet
  ): Promise<ToolResult>;
}
```

**Built-in tools**:
- File operations: `readFile`, `writeFile`, `listFiles`
- Execution: `runCommand`, `executeCode`
- Search: `searchCode`, `grepFiles`
- Browser: web scraping, navigation

### MCP (Model Context Protocol) Integration

```typescript
// MCP implementation in src/mcp/
interface MCPServer {
  name: string;
  transport: Transport; // stdio | SSE | WebSocket
  capabilities: Capability[];
}

// MCP tools are exposed via standardized protocol
const mcpClient = await MCPClient.connect({
  command: "npx",
  args: ["-y", "@modelcontextprotocol/server-filesystem"],
  env: { ...process.env }
});
```

**MCP usage patterns**:
```typescript
// Connect to MCP server
await mcpClient.initialize();

// List available tools
const tools = await mcpClient.listTools();

// Call MCP tool
const result = await mcpClient.callTool("read_file", {
  path: "/path/to/file"
});
```

### Sandbox Mechanism

```typescript
// Sandbox isolation in src/sandbox/
class Sandbox {
  async executeInIsolation(
    code: string,
    options: SandboxOptions
  ): Promise<ExecutionResult> {
    // Creates isolated environment
    // - Separate process/container
    // - Limited filesystem access
    // - Network restrictions
    // - Resource quotas
  }
}
```

**Security layers**:
1. Permission system (user approval for sensitive ops)
2. Filesystem isolation (chroot/container)
3. Network filtering
4. Resource limits (CPU, memory, time)

### Skills Extension System

```typescript
// Skills in src/skills/
interface Skill {
  name: string;
  triggers: string[];
  execute: (context: Context) => Promise<void>;
}

// Example skill structure
export const debugSkill: Skill = {
  name: "debug-assistant",
  triggers: ["debug", "fix error", "troubleshoot"],
  async execute(context) {
    // Skill implementation
    const error = await analyzeError(context);
    const fix = await suggestFix(error);
    await applyFix(fix);
  }
};
```

## Installation & Access

This is an analysis repository, not executable code. To explore:

```bash
# Clone the analysis
git clone https://github.com/liuup/claude-code-analysis.git
cd claude-code-analysis

# Extract source
unzip src.zip

# Browse analysis documents
cd analysis
```

## Key Analysis Documents

### Architecture Understanding

```bash
# Read core architecture
cat analysis/01-architecture-overview.md

# Understand execution flow
# Entry → Init → Command → TUI → Query → Tools/Memory/Extensions
```

### Security Analysis

```bash
# User data collection
cat analysis/02-security-analysis.md

# Key findings:
# - Telemetry data sent to Anthropic
# - Local session storage in ~/.claude-code/
# - Sandbox isolation for code execution
# - Permission gates for file/network access
```

### Memory Implementation

```typescript
// From analysis/04-agent-memory.md
/*
Multi-tier memory:
1. Active context (current session)
2. Compressed history (summarized)
3. Indexed memory (long-term retrieval)

Token budget management:
- Max context: ~200k tokens
- Auto-compression when exceeded
- Sliding window + importance ranking
*/
```

### Tool Call Deep Dive

```typescript
// From analysis/04b-tool-call-implementation.md
/*
Tool execution flow:
1. LLM generates tool_use block
2. Permission check (user approval if needed)
3. Tool execution in sandbox
4. Result returned to LLM
5. Continue conversation loop

Tool categories:
- Filesystem (read/write/search)
- Execution (shell/code)
- Browser (web scraping)
- MCP (external protocols)
*/
```

### MCP Technical Details

```bash
# Read MCP implementation
cat analysis/04d-mcp-implementation.md

# MCP servers communicate via:
# - stdin/stdout (stdio transport)
# - Server-Sent Events (SSE)
# - WebSocket
#
# Tools registered dynamically at runtime
```

## Common Patterns

### Analyzing Agent Memory Usage

```typescript
// Reference: src/sessionStorage/
// Pattern: Multi-layer storage with compression

// Active session (in-memory)
const activeContext = {
  messages: Message[],
  tools: ToolDefinition[],
  files: FileContext[]
};

// Compressed history (when token limit hit)
const compressed = {
  summary: string,
  keyPoints: string[],
  truncatedMessages: Message[]
};

// Persistent storage (~/.claude-code/sessions/)
interface PersistedSession {
  id: string;
  timestamp: number;
  transcript: Message[];
  metadata: SessionMetadata;
}
```

### Tracing Tool Execution

```typescript
// Reference: src/tools/, src/orchestration/

// 1. Tool registration
const tool = {
  name: "read_file",
  schema: { path: { type: "string" } },
  execute: async (params) => {
    // Permission check
    if (needsApproval(params.path)) {
      await requestPermission("read_file", params);
    }
    // Execute in sandbox
    return await sandbox.readFile(params.path);
  }
};

// 2. LLM invokes tool
// <tool_use name="read_file">{"path": "/src/app.ts"}</tool_use>

// 3. Tool result
// <tool_result>file contents...</tool_result>
```

### Understanding Security Boundaries

```typescript
// Reference: src/sandbox/, src/permissions/

// Permission levels:
enum PermissionLevel {
  ALWAYS_ALLOW = "always",
  PROMPT = "prompt",      // Default
  NEVER_ALLOW = "never"
}

// Sandbox isolation:
const sandboxConfig = {
  workingDir: "/tmp/claude-sandbox-xxx",
  allowedPaths: ["/workspace"],
  blockedHosts: ["internal.company.com"],
  resourceLimits: {
    maxMemory: "2GB",
    maxCPU: "2 cores",
    timeout: 30000 // 30s
  }
};
```

### Comparing to Other AI Coding Agents

```bash
# Reference: analysis/08-competitive-comparison.md

# Claude Code vs Cursor:
# - Claude: Local-first, MCP-based extensions
# - Cursor: Cloud-integrated, VSCode-based

# Claude Code vs Aider:
# - Claude: Full TUI, multi-agent support
# - Aider: CLI-focused, git-centric

# Claude Code vs GitHub Copilot:
# - Claude: Autonomous agent, tool use
# - Copilot: Autocomplete, inline suggestions
```

## Configuration

Claude Code configuration (inferred from source):

```typescript
// Config file: ~/.claude-code/config.json
interface Config {
  apiKey?: string; // From ANTHROPIC_API_KEY env var
  model: string;   // Default: "claude-3-5-sonnet-20241022"
  
  // Memory settings
  memory: {
    maxTokens: number;        // ~200k
    compressionThreshold: number;
    persistSessions: boolean;
  };
  
  // Sandbox settings
  sandbox: {
    enabled: boolean;
    isolation: "process" | "container";
    allowedPaths: string[];
  };
  
  // MCP servers
  mcpServers: {
    [name: string]: {
      command: string;
      args: string[];
      env?: Record<string, string>;
    };
  };
  
  // Telemetry
  telemetry: {
    enabled: boolean;
    endpoint?: string;
  };
}
```

**Environment variables**:
```bash
# Required
export ANTHROPIC_API_KEY=your_api_key_here

# Optional
export CLAUDE_CODE_CONFIG_DIR=~/.claude-code
export CLAUDE_CODE_LOG_LEVEL=debug
export CLAUDE_CODE_TELEMETRY_OPT_OUT=1
```

## Troubleshooting

### Understanding Data Collection

**Issue**: What user data does Claude Code collect?

```bash
# Reference: analysis/02-security-analysis.md section 1

# Data collected:
# 1. Telemetry: usage stats, errors, feature flags
# 2. Session transcripts: stored locally in ~/.claude-code/
# 3. Context: code snippets sent to LLM API
# 4. MCP data: depends on external MCP servers

# Check local storage:
ls -la ~/.claude-code/sessions/
ls -la ~/.claude-code/memory/
```

### Analyzing Memory Compression

**Issue**: How does Claude Code handle long conversations?

```typescript
// Reference: src/sessionStorage/compression.ts

/*
Compression strategy:
1. Token count monitoring
2. When > threshold (e.g., 180k tokens):
   - Summarize oldest messages
   - Retain recent N messages
   - Keep important system messages
3. Store summary in compressed tier
4. Continue with reduced context

Key files:
- src/sessionStorage/SessionMemory.ts
- src/memdir/compression.ts
*/
```

### Tracing Tool Permission Flows

**Issue**: Why does Claude Code ask permission for certain operations?

```typescript
// Reference: src/permissions/

// Permission required for:
// - Writing files outside workspace
// - Executing shell commands
// - Network requests to new domains
// - Installing packages

// Check permission logic:
// src/permissions/PermissionManager.ts
// src/tools/permissions.ts
```

### Investigating MCP Integration Issues

**Issue**: MCP server not connecting

```bash
# Reference: analysis/04d-mcp-implementation.md

# Debug MCP connection:
# 1. Check MCP server config in ~/.claude-code/config.json
# 2. Test server directly:
npx -y @modelcontextprotocol/server-filesystem

# 3. Check logs:
tail -f ~/.claude-code/logs/mcp-*.log

# 4. Verify transport type (stdio/SSE/WebSocket)
# 5. Check environment variables passed to server
```

### Understanding Hidden Features

```bash
# Reference: analysis/11-hidden-features-and-easter-eggs.md

# Feature flags in source:
# - ENABLE_SWARM_MODE: Multi-agent coordination
# - ENABLE_REMOTE_TOOLS: Cloud-based tool execution
# - DEBUG_MODE: Verbose logging
# - EXPERIMENTAL_MEMORY: Advanced memory features

# Check: src/featureFlags.ts
# Check: src/config/experiments.ts
```

## Code Evidence References

Key source files for common investigations:

```typescript
// Architecture entry points
"entrypoints/cli.tsx"           // CLI main
"src/init.ts"                   // Initialization
"src/query.ts"                  // Query engine

// Memory system
"src/sessionStorage/"           // Session management
"src/memdir/"                   // Memory directories
"src/transcript/"               // Conversation logs

// Tool system
"src/tools/"                    // Tool definitions
"src/orchestration/"            // Tool execution
"src/permissions/"              // Permission gates

// MCP integration
"src/mcp/"                      // MCP client/server
"src/mcp/transports/"           // Communication layer

// Security
"src/sandbox/"                  // Isolation
"src/permissions/"              // Access control
"src/telemetry/"                // Analytics

// UI
"src/components/"               // TUI components
"src/tui/"                      // Terminal UI
```

## Analysis Navigation

```bash
# Start with architecture overview
cat analysis/01-architecture-overview.md

# Then dive into specific topics:
cat analysis/04-agent-memory.md          # Memory system
cat analysis/04b-tool-call-implementation.md  # Tool calls
cat analysis/04d-mcp-implementation.md    # MCP
cat analysis/02-security-analysis.md      # Security/privacy

# For comparisons:
cat analysis/08-competitive-comparison.md

# For complete file tree:
cat analysis/10-src-file-tree.md

# For component-level details:
ls analysis/components/
```

## Academic & Research Use

This repository is for **academic research and technical learning only**:

- ✅ Understanding AI coding agent architecture
- ✅ Security boundary analysis
- ✅ Privacy design patterns
- ✅ Competitive landscape research
- ❌ Commercial use prohibited
- ❌ Bypassing security mechanisms prohibited
- ❌ Violating Anthropic terms of service prohibited

All source code rights belong to [Anthropic](https://www.anthropic.com).
