---
name: claude-code-best
description: Expert in using Claude Code Best (CCB) - a production-grade, debuggable fork of Anthropic's Claude Code CLI with enterprise features
triggers:
  - "how do I use Claude Code Best"
  - "set up CCB with custom API provider"
  - "configure claude code best features"
  - "debug claude code best project"
  - "use pipe IPC in CCB"
  - "enable langfuse monitoring in claude code"
  - "set up remote control for claude code"
  - "use web search in claude code best"
---

# claude-code-best

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

Claude Code Best (CCB) is a reverse-engineered, production-ready implementation of Anthropic's Claude Code CLI. It's written in TypeScript, runs on Bun or Node.js, and provides enterprise-grade features including multi-instance orchestration, ACP protocol support, Langfuse monitoring, remote control, web search, voice mode, and computer/browser automation.

## What It Does

CCB is an AI coding agent terminal interface that:
- Runs Claude (or compatible models) in your terminal with full file system access
- Supports multi-instance coordination via Pipe IPC and LAN discovery
- Integrates with Zed, Cursor, and other IDEs via ACP protocol
- Provides enterprise monitoring (Langfuse, Sentry, GrowthBook)
- Offers web search, voice input, computer/browser control
- Works with any Anthropic-compatible API provider (OpenAI, Gemini, etc.)

## Installation

### From NPM (Recommended)

```bash
npm i -g claude-code-best

# Start CCB with Node.js
ccb

# Start CCB with Bun runtime
ccb-bun

# Update to latest version
ccb update
```

### From Source

**Prerequisites:**
```bash
# Install Bun >= 1.3.11
curl -fsSL https://bun.sh/install | bash
exec $SHELL  # Reload shell

# Verify installation
bun --version
bun upgrade  # Update if needed
```

**Clone and run:**
```bash
git clone https://github.com/claude-code-best/claude-code.git
cd claude-code
bun install

# Development mode (version 888)
bun run dev

# Build for production
bun run build

# Run built version
node dist/cli.js
# or
bun dist/cli.js
```

## Configuration

### Login to API Provider

On first run, enter `/login` in the REPL:

```bash
ccb
# In REPL:
/login
```

Choose provider type:
- **Anthropic Compatible** - any Claude-compatible API
- **OpenAI Compatible** - OpenAI or compatible services
- **Gemini Compatible** - Google Gemini API

**Required fields:**

| Field | Description | Example |
|-------|-------------|---------|
| Base URL | API endpoint | `https://api.anthropic.com` |
| API Key | Auth token | `$ANTHROPIC_API_KEY` |
| Haiku Model | Fast model | `claude-haiku-4-5-20251001` |
| Sonnet Model | Balanced model | `claude-sonnet-4-6` |
| Opus Model | Premium model | `claude-opus-4-6` |

Use **Tab/Shift+Tab** to navigate, **Enter** to confirm.

### Environment Variables

```bash
# Enable features via FEATURE_<NAME>=1
FEATURE_BUDDY=1 ccb

# Remote control
CLAUDE_BRIDGE_BASE_URL=https://remote-control.claude-code-best.win/ \
CLAUDE_BRIDGE_OAUTH_TOKEN=$YOUR_TOKEN \
ccb --remote-control

# Langfuse monitoring
LANGFUSE_PUBLIC_KEY=$YOUR_KEY \
LANGFUSE_SECRET_KEY=$YOUR_SECRET \
LANGFUSE_HOST=https://cloud.langfuse.com \
ccb

# Poor mode (reduce API calls)
ccb  # then type /poor in REPL
```

## Key Commands

### In REPL

```bash
/login              # Configure API provider
/pipes              # Show active pipe connections
/poor               # Toggle poor mode (reduce costs)
/dream              # Organize memory files
/voice doubao       # Enable voice input
/teach-me <topic>   # Interactive learning about CCB architecture

# Examples:
/teach-me Claude Code architecture
/teach-me React Ink terminal rendering --level beginner
/teach-me Tool system --resume
```

### CLI Flags

```bash
# Start with remote control
ccb --remote-control

# Enable channels (external notifications)
ccb --channels plugin:name@marketplace

# Debug mode with inspector
bun run dev:inspect
# Then attach VS Code debugger (F5 → "Attach to Bun (TUI debug)")
```

## Usage Patterns

### Basic Usage

```typescript
// CCB is primarily a CLI tool, but internally uses these patterns:

// 1. Tool execution (from src/tools/)
import { executeShellCommand } from './tools/bash';

const result = await executeShellCommand({
  command: 'ls -la',
  workingDir: process.cwd()
});

// 2. Multi-instance coordination (Pipe IPC)
import { PipeManager } from './ipc/pipe-manager';

const pipeManager = new PipeManager();
await pipeManager.broadcast({
  type: 'message',
  content: 'Task complete'
});

// 3. Memory management
import { MemoryService } from './memory/service';

const memory = new MemoryService();
await memory.extract({
  sessionId: 'current',
  content: 'User prefers TypeScript over JavaScript'
});
```

### Remote Control Setup

```bash
# Self-hosted Docker setup
docker run -d \
  -p 3000:3000 \
  -e OAUTH_SECRET=$YOUR_SECRET \
  claude-code-remote-control

# Client connection
CLAUDE_BRIDGE_BASE_URL=http://localhost:3000 \
CLAUDE_BRIDGE_OAUTH_TOKEN=$YOUR_TOKEN \
ccb --remote-control
```

### Web Search Integration

```typescript
// Enabled automatically when API keys are set
export BING_SEARCH_API_KEY=$YOUR_BING_KEY
# or
export BRAVE_SEARCH_API_KEY=$YOUR_BRAVE_KEY

ccb
# In session, ask: "search for latest TypeScript features"
```

### Langfuse Monitoring

```bash
# Configure via env vars
export LANGFUSE_PUBLIC_KEY=pk-lf-xxx
export LANGFUSE_SECRET_KEY=sk-lf-xxx
export LANGFUSE_HOST=https://cloud.langfuse.com

ccb
# Every agent loop will be tracked in Langfuse dashboard
```

### Computer Use (Screen Control)

```typescript
// Requires computer-use feature enabled
FEATURE_COMPUTER_USE=1 ccb

// In session:
// "take a screenshot of my screen"
// "click on the file menu"
// "type 'hello world' in the editor"
```

### Chrome Automation

```bash
# Install chrome-use MCP server first
npx @modelcontextprotocol/create-server chrome-use

# Configure in CCB
ccb
# In REPL: configure MCP server path

# Usage in session:
# "open chrome and navigate to github.com"
# "fill out the login form with my credentials"
# "scrape the table data from this page"
```

## Development Patterns

### Project Structure

```
claude-code/
├── src/
│   ├── cli.ts              # Entry point
│   ├── tools/              # Built-in tools (bash, read_file, etc.)
│   ├── ipc/                # Pipe IPC for multi-instance
│   ├── memory/             # Memory extraction & retrieval
│   ├── providers/          # API provider adapters
│   ├── ui/                 # React Ink terminal components
│   └── features/           # Feature flag implementations
├── dist/                   # Build output (450+ chunks)
├── build.ts                # Bun build script
└── package.json
```

### Adding a Custom Tool

```typescript
// src/tools/my-custom-tool.ts
import { z } from 'zod';

export const MyCustomToolSchema = z.object({
  input: z.string().describe('Tool input parameter')
});

export async function executeMyCustomTool(
  params: z.infer<typeof MyCustomToolSchema>
): Promise<string> {
  // Your tool logic
  return `Processed: ${params.input}`;
}

// Register in src/tools/index.ts
export const tools = [
  {
    name: 'my_custom_tool',
    description: 'Does something useful',
    input_schema: MyCustomToolSchema,
    execute: executeMyCustomTool
  },
  // ... other tools
];
```

### Debugging in VS Code

```json
// .vscode/launch.json (already configured)
{
  "configurations": [
    {
      "name": "Attach to Bun (TUI debug)",
      "type": "bun",
      "request": "attach",
      "url": "ws://localhost:8888"
    }
  ]
}
```

**Steps:**
1. Terminal: `bun run dev:inspect`
2. Set breakpoints in `src/`
3. VS Code: F5 → Select "Attach to Bun (TUI debug)"

## Common Issues

### Installation Fails

```bash
# Clear old versions
npm rm -g claude-code-best

# Install specific version
npm i -g claude-code-best@5.0.0

# If Bun install has issues, use npm
npm i -g claude-code-best
```

### Bun Not Found After Install

```bash
# macOS/Linux - reload shell
exec $SHELL

# Or manually source
source ~/.zshrc  # or ~/.bashrc

# Windows - reopen PowerShell
```

### Feature Not Working

```bash
# Check feature flags are set
FEATURE_NAME=1 ccb

# Example for all features:
FEATURE_BUDDY=1 \
FEATURE_FORK_SUBAGENT=1 \
FEATURE_WEB_SEARCH=1 \
ccb
```

### API Connection Issues

```bash
# Verify credentials in REPL
/login
# Re-enter Base URL and API Key

# Test with simple request
# In session: "what's 2+2"
```

### Build Errors

```bash
# Update Bun to latest
bun upgrade

# Clean install
rm -rf node_modules bun.lockb
bun install

# Rebuild
bun run build
```

### Remote Control Not Connecting

```bash
# Verify env vars are exported
echo $CLAUDE_BRIDGE_BASE_URL
echo $CLAUDE_BRIDGE_OAUTH_TOKEN

# Test with official hosted version
CLAUDE_BRIDGE_BASE_URL=https://remote-control.claude-code-best.win/ \
CLAUDE_BRIDGE_OAUTH_TOKEN=test-my-key \
ccb --remote-control
```

## Advanced Configuration

### Multi-Instance Pipe IPC

```bash
# Terminal 1 (main instance)
ccb

# Terminal 2 (sub instance, auto-discovers main)
ccb

# In main REPL:
/pipes              # Shows connected instances
# Shift+↓ to broadcast message to all pipes
```

### LAN Cross-Machine Discovery

```bash
# Machine A
FEATURE_LAN_DISCOVERY=1 ccb

# Machine B (same network)
FEATURE_LAN_DISCOVERY=1 ccb

# Instances automatically discover and connect
```

### Channels (External Notifications)

```bash
# Install channel plugin (e.g., Slack)
ccb --channels plugin:slack@marketplace

# Configure webhook in REPL
# Now external events can push messages into session
```

### Custom Memory Optimization

```bash
# In REPL, manually trigger memory organization
/dream

# Or configure auto-dream
FEATURE_AUTO_DREAM=1 ccb
```

## Resources

- **Homepage:** https://ccb.agent-aura.top/
- **Documentation:** https://ccb.agent-aura.top/docs/
- **GitHub:** https://github.com/claude-code-best/claude-code
- **Discord:** https://discord.gg/uApuzJWGKX
- **Feature Docs:** See `docs/features/` in repository

## Example Session

```bash
$ ccb
Claude Code Best v5.0.0

You: I need to analyze all TypeScript files in src/ and create a dependency graph
