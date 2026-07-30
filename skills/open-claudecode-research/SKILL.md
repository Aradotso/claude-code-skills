---
name: open-claudecode-research
description: Research archive of Claude Code source and runtime artifacts reconstructed from npm source maps
triggers:
  - "how does Claude Code work internally"
  - "explore Claude Code source code"
  - "understand Claude Code architecture"
  - "use open claudecode locally"
  - "run claude code from source"
  - "configure claude code settings"
  - "add plugins to claude code"
  - "reverse engineer claude code"
---

# Open-ClaudeCode Research Archive

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## What This Project Is

Open-ClaudeCode is a complete research archive of Anthropic's Claude Code CLI, reconstructed from official npm package source maps. It provides:

- **Full TypeScript source code** (1,902 files) for studying Claude Code's architecture
- **Runnable CLI** (v2.1.88) that works identically to the npm-published version
- **Official plugins** (13 plugins) for code review, feature development, security, etc.
- **Configuration examples** for various use cases
- **Native vendor modules** for audio capture and ripgrep

This is primarily a research/learning resource, but the CLI is fully functional.

## Installation

### Clone the Repository

```bash
git clone https://github.com/LING71671/Open-ClaudeCode.git
cd Open-ClaudeCode
```

### Prerequisites

- **Node.js 18+** required
- **API access** (one of):
  - Anthropic API key from console.anthropic.com
  - Third-party proxy with Anthropic-compatible API
  - Claude subscription (OAuth login)

### Verify Installation

```bash
node --version  # Must be >= 18.0.0
node package/cli.js --version  # Should show 2.1.88
```

## Authentication Setup

### Option 1: API Key (Recommended for Programmatic Use)

Create `~/.claude/settings.json`:

```json
{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "sk-ant-your-key-here"
  }
}
```

Or use environment variables:

```bash
export ANTHROPIC_AUTH_TOKEN="sk-ant-your-key-here"
node package/cli.js
```

### Option 2: Third-Party Proxy

For users in regions with API access restrictions:

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://your-proxy-endpoint.com",
    "ANTHROPIC_AUTH_TOKEN": "your-proxy-api-key"
  }
}
```

### Option 3: OAuth (Claude Subscription)

```bash
# Simply run - will open browser for login
node package/cli.js
```

## Core CLI Usage

### Interactive Mode

```bash
# Start interactive session
node package/cli.js

# With specific model
node package/cli.js --model opus

# With auto-accept edits
node package/cli.js --permission-mode acceptEdits
```

### Non-Interactive Mode

```bash
# Single prompt
node package/cli.js -p "explain closures in JavaScript"

# Process specific file
node package/cli.js -p "refactor the getUserData function in src/api.ts"

# JSON output for automation
node package/cli.js -p "list project dependencies" --output-format json

# Continue previous session
node package/cli.js -c
node package/cli.js -r <session-id>
```

### Model Selection

```bash
# Available model aliases
node package/cli.js --model sonnet    # Claude Sonnet (default, balanced)
node package/cli.js --model opus      # Claude Opus (most capable)
node package/cli.js --model haiku     # Claude Haiku (fastest)

# Or full model name
node package/cli.js --model claude-sonnet-4-6
```

## Configuration Patterns

### Project-Specific Settings

Create `settings.json` in your project root:

```json
{
  "model": "claude-sonnet-4-6",
  "permissionMode": "acceptEdits",
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "${ANTHROPIC_API_KEY}"
  },
  "systemPrompt": "You are a senior developer helping with a TypeScript project. Focus on type safety and best practices."
}
```

Run with settings:

```bash
node /path/to/Open-ClaudeCode/package/cli.js --settings settings.json
```

### Global Configuration

Settings in `~/.claude/settings.json` apply to all sessions:

```json
{
  "model": "claude-sonnet-4-6",
  "theme": "dark",
  "vimMode": true,
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "${ANTHROPIC_API_KEY}",
    "HTTPS_PROXY": "http://127.0.0.1:7890"
  }
}
```

### Permission Modes

```bash
# Default: prompt for every action
node package/cli.js

# Auto-accept file edits (still prompts for dangerous operations)
node package/cli.js --permission-mode acceptEdits

# Skip all permissions (SANDBOX ONLY)
node package/cli.js --dangerously-skip-permissions
```

### Environment-Specific Configurations

**Development:**
```json
{
  "permissionMode": "acceptEdits",
  "model": "claude-sonnet-4-6",
  "systemPrompt": "Development mode - prefer speed over perfection"
}
```

**Production Review:**
```json
{
  "permissionMode": "prompt",
  "model": "claude-opus-4-6",
  "systemPrompt": "Production code review - prioritize security and correctness"
}
```

## Plugin System

### Using Official Plugins

```bash
# Load single plugin
node package/cli.js --plugin-dir ./plugins/code-review

# Load multiple plugins
node package/cli.js \
  --plugin-dir ./plugins/code-review \
  --plugin-dir ./plugins/commit-commands

# With custom plugin
node package/cli.js --plugin-dir /path/to/your/plugin
```

### Available Official Plugins

- **code-review** - Automated code review capabilities
- **commit-commands** - Smart git commit message generation
- **pr-review-toolkit** - Pull request analysis tools
- **security-guidance** - Security best practices enforcement
- **feature-dev** - Feature development workflow
- **frontend-design** - UI/UX development assistance
- **plugin-dev** - Tools for developing new plugins
- **agent-sdk-dev** - SDK development utilities

### Plugin Structure

```
my-plugin/
├── package.json
├── plugin.ts          # Main entry point
└── skills/
    └── my-skill.md    # Skill definitions
```

Example `plugin.ts`:

```typescript
import { Plugin } from '@anthropic/sdk';

export default class MyPlugin extends Plugin {
  name = 'my-plugin';
  version = '1.0.0';
  
  async initialize() {
    this.registerCommand('analyze', async (args) => {
      // Command implementation
    });
  }
}
```

## Common Workflows

### Code Generation

```bash
node package/cli.js -p "Create a REST API endpoint for user authentication using Express.js and JWT"
```

In interactive mode:
```
> Create a TypeScript interface for a blog post with title, content, author, and timestamps
> Add validation using zod
> Generate unit tests
```

### Code Review

```bash
# Review uncommitted changes
node package/cli.js --plugin-dir ./plugins/code-review -p "review my current changes"

# Review specific file
node package/cli.js -p "review src/auth/login.ts for security issues"
```

### Refactoring

```typescript
// Before asking Claude
// src/utils/old-code.ts
function processData(data: any) {
  // messy implementation
  return data.map(x => x.value).filter(x => x > 0);
}
```

Prompt:
```
> Refactor processData in src/utils/old-code.ts to:
> - Use proper TypeScript types
> - Add error handling
> - Extract reusable helpers
> - Add JSDoc comments
```

### Git Workflows

```bash
# Smart commit messages
node package/cli.js --plugin-dir ./plugins/commit-commands
# Then in interactive mode: /commit

# PR review
node package/cli.js --plugin-dir ./plugins/pr-review-toolkit -p "review PR #123"
```

### Multi-File Changes

```bash
node package/cli.js -p "Add error logging to all API endpoints in src/api/"
```

Interactive workflow:
```
> Create a new feature: user profile editing
> Add database schema
> Create API routes
> Add frontend form
> Write integration tests
```

## Source Code Exploration

### Architecture Overview

```
src/
├── tools/           # 30+ built-in tools (file operations, git, search)
├── commands/        # 50+ CLI commands
├── services/        # API clients, MCP, OAuth
├── components/      # React UI components (389 files)
├── ink/            # Terminal UI framework
├── utils/          # Core utilities (564 files)
├── hooks/          # React hooks for state management
└── bridge/         # Communication layer
```

### Key Modules to Study

**Tool System (`src/tools/`):**
```typescript
// Example: Understanding the file editing tool
// src/tools/edit-file.ts
interface EditFileArgs {
  path: string;
  edits: Array<{
    oldText: string;
    newText: string;
  }>;
}
```

**Command System (`src/commands/`):**
```typescript
// src/commands/commit.ts
// How /commit command generates messages
```

**Service Layer (`src/services/`):**
```typescript
// src/services/anthropic-api.ts
// Direct API communication patterns
```

### Analyzing Tool Implementation

```bash
# Study how a specific tool works
cat src/tools/ripgrep.ts
cat src/tools/file-operations.ts
cat src/tools/git-operations.ts
```

Example tool pattern:

```typescript
export interface ToolDefinition {
  name: string;
  description: string;
  inputSchema: object;
  execute: (args: any) => Promise<any>;
}

export const searchTool: ToolDefinition = {
  name: 'search_files',
  description: 'Search for text in files',
  inputSchema: {
    type: 'object',
    properties: {
      pattern: { type: 'string' },
      path: { type: 'string' }
    }
  },
  execute: async ({ pattern, path }) => {
    // Implementation using vendor/ripgrep
  }
};
```

## Advanced Configuration

### Custom System Prompts

```json
{
  "systemPrompt": "You are an expert in React and TypeScript. When suggesting code:\n- Always use functional components with hooks\n- Prefer const over let\n- Use strict TypeScript types\n- Follow airbnb style guide\n- Add meaningful comments for complex logic"
}
```

### Output Formatting

```bash
# Human-readable (default)
node package/cli.js -p "list files"

# JSON for automation
node package/cli.js -p "analyze dependencies" --output-format json

# Compact mode
node package/cli.js -p "explain this code" --output-format compact
```

### Session Management

```bash
# List sessions
node package/cli.js --list-sessions

# Resume specific session
node package/cli.js -r abc123

# Continue last session
node package/cli.js -c

# Share session
# In interactive mode: /share
```

### Token Budget Control

```json
{
  "maxTokens": 4096,
  "contextWindow": 200000,
  "budget": {
    "daily": 1000000,
    "perRequest": 100000
  }
}
```

## Troubleshooting

### Authentication Errors

```bash
# Check current auth status
node package/cli.js --check-auth

# Re-authenticate
rm -rf ~/.claude/auth
node package/cli.js  # Will trigger OAuth flow

# Verify API key
echo $ANTHROPIC_AUTH_TOKEN
```

### Network/Proxy Issues

```bash
# Set proxy
export HTTPS_PROXY=http://127.0.0.1:7890
export HTTP_PROXY=http://127.0.0.1:7890

# Or in settings.json
{
  "env": {
    "HTTPS_PROXY": "http://127.0.0.1:7890"
  }
}

# Test connectivity
node package/cli.js -p "hello" --debug
```

### Performance Issues

```bash
# Use faster model
node package/cli.js --model haiku

# Reduce context
node package/cli.js --max-tokens 2048

# Clear session history
# In interactive mode: /clear
```

### Plugin Loading Errors

```bash
# Verify plugin structure
ls -la plugins/your-plugin/
cat plugins/your-plugin/package.json

# Check plugin logs
node package/cli.js --plugin-dir ./plugins/your-plugin --debug
```

### Binary Module Issues

```bash
# Verify platform binaries exist
ls -la package/vendor/ripgrep/
ls -la package/vendor/audio-capture/

# If missing, the CLI will fall back to JavaScript implementations
# No action needed unless specific features are required
```

### Cost Tracking

```bash
# In interactive mode
/cost    # Current session cost
/stats   # Overall usage statistics

# Enable cost warnings in settings.json
{
  "budget": {
    "warnThreshold": 1000000,
    "hardLimit": 5000000
  }
}
```

## Integration Examples

### CI/CD Pipeline

```yaml
# .github/workflows/code-review.yml
name: AI Code Review
on: [pull_request]
jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: git clone https://github.com/LING71671/Open-ClaudeCode.git
      - run: |
          node Open-ClaudeCode/package/cli.js \
            --plugin-dir ./Open-ClaudeCode/plugins/pr-review-toolkit \
            -p "Review changes in this PR" \
            --output-format json > review.json
        env:
          ANTHROPIC_AUTH_TOKEN: ${{ secrets.ANTHROPIC_API_KEY }}
```

### Pre-commit Hook

```bash
#!/bin/bash
# .git/hooks/pre-commit

CLAUDE_CLI="/path/to/Open-ClaudeCode/package/cli.js"

# Generate commit message
node $CLAUDE_CLI \
  --plugin-dir /path/to/Open-ClaudeCode/plugins/commit-commands \
  -p "generate commit message for staged changes" \
  --output-format json > /tmp/commit-msg.json

# Extract message and use it
```

### Editor Integration

```typescript
// vscode-extension/src/claude.ts
import { exec } from 'child_process';
import { promisify } from 'util';

const execAsync = promisify(exec);

async function askClaude(prompt: string): Promise<string> {
  const { stdout } = await execAsync(
    `node /path/to/Open-ClaudeCode/package/cli.js -p "${prompt}" --output-format json`,
    { env: { ...process.env, ANTHROPIC_AUTH_TOKEN: process.env.CLAUDE_API_KEY } }
  );
  return JSON.parse(stdout).response;
}
```

## Research and Learning

### Studying the Architecture

Key areas to explore:

1. **Tool System** - How tools are defined and executed
2. **Command Registry** - Slash command implementation
3. **Permission Model** - How file operations are authorized
4. **MCP Integration** - Model Context Protocol support
5. **UI Framework** - Ink-based terminal UI
6. **State Management** - React hooks in CLI context

### Extending Functionality

```typescript
// Example: Creating a custom tool
// my-tools/custom-analyzer.ts

import { ToolDefinition } from '@anthropic/sdk';

export const customAnalyzer: ToolDefinition = {
  name: 'analyze_complexity',
  description: 'Analyze code complexity metrics',
  inputSchema: {
    type: 'object',
    properties: {
      filePath: { 
        type: 'string',
        description: 'Path to file to analyze'
      }
    },
    required: ['filePath']
  },
  execute: async ({ filePath }) => {
    // Your implementation
    const metrics = await analyzeFile(filePath);
    return {
      complexity: metrics.cyclomaticComplexity,
      maintainability: metrics.maintainabilityIndex
    };
  }
};
```

### Debugging the CLI

```bash
# Enable debug logging
NODE_ENV=development node package/cli.js --debug

# Trace API calls
DEBUG=anthropic:* node package/cli.js

# Inspect source maps
node --enable-source-maps package/cli.js
```

This skill enables AI coding agents to effectively use Open-ClaudeCode for both practical development tasks and architectural research.
