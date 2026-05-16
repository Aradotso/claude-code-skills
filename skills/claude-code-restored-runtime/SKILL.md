---
name: claude-code-restored-runtime
description: Run the restored Claude Code agent runtime with GitHub Models or Copilot providers for local AI coding workflows
triggers:
  - "use the restored Claude Code runtime"
  - "run Claude Code with GitHub models"
  - "set up claude-code-rev locally"
  - "switch between GitHub providers in Claude Code"
  - "configure Copilot with restored Claude Code"
  - "use GitHub Copilot models in Claude Code"
  - "run the Claude Code agent with custom providers"
  - "debug restored Claude Code issues"
---

# claude-code-restored-runtime

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## What It Does

`claude-code-rev` is a restored and runnable version of Claude Code reconstructed from source maps. It provides:

- A local AI coding agent runtime that works with GitHub-backed inference providers
- Support for both GitHub Models (OpenAI-compatible) and GitHub Copilot endpoints
- Multi-provider switching with interactive CLI commands
- Chrome MCP and Computer Use MCP compatibility layers
- Restored CLI bootstrap with full command tree

The project is **not** the original upstream repository—some modules are shims or degraded implementations to enable runnable workflows.

## Installation

### Prerequisites

- **Bun** 1.3.5 or newer
- **Node.js** 24 or newer
- **GitHub CLI** (`gh`) for account-based authentication (optional but recommended)

### Clone and Install

```bash
git clone https://github.com/oboard/claude-code-rev.git
cd claude-code-rev
bun install
```

### Verify Installation

```bash
bun run version
bun run dev --help
```

## Authentication Setup

Both GitHub providers use this lookup order:

1. Provider-specific environment variable
2. `GH_TOKEN`
3. `GITHUB_TOKEN`
4. `gh auth token` (GitHub CLI)

### Option 1: GitHub CLI (Recommended)

```bash
gh auth login
```

This enables account-based authentication without manual token management.

### Option 2: Environment Variable

```bash
export GITHUB_TOKEN="ghp_your_token_here"
```

Or add to `.env`:

```bash
GITHUB_TOKEN=ghp_your_token_here
```

## Running the Runtime

### Default Run

```bash
bun run dev
```

### With GitHub Models Provider

```bash
bun run dev --settings '{"provider":"github-models"}'
```

Specify a model:

```bash
bun run dev --settings '{"provider":"github-models"}' --model "openai/gpt-4.1"
```

### With GitHub Copilot Provider

```bash
bun run dev --settings '{"provider":"github-copilot"}'
```

Specify a Copilot-hosted Claude model:

```bash
bun run dev --settings '{"provider":"github-copilot"}' --model "claude-opus-4.6"
```

## CLI Commands

### Provider Management

| Command | Description |
|---------|-------------|
| `/provider` | Open interactive provider picker |
| `/provider info` | Show current provider and environment overrides |
| `/provider github-copilot` | Switch directly to GitHub Copilot |
| `/provider github-models` | Switch directly to GitHub Models |

### Model Management

| Command | Description |
|---------|-------------|
| `/model` | Open model picker for current provider |

### General Commands

```bash
bun run dev --help    # Show full command tree
bun run version       # Print restored version
```

## Supported Models

### GitHub Copilot – Validated Claude Models

These models work with the Claude Code runtime and tool loop:

- `claude-sonnet-4.6`
- `claude-opus-4.6`
- `claude-haiku-4.5`
- `claude-sonnet-4.5`
- `claude-opus-4.5`
- `claude-sonnet-4`

**Note:** Copilot GPT/Grok models are not fully wired yet—they require the `/responses` API path instead of the current chat/messages shim.

### GitHub Models

Any OpenAI-compatible model available through GitHub Models, including:

- `openai/gpt-4.1`
- `openai/gpt-4o`
- `meta-llama/Llama-3.3-70B-Instruct`

## Configuration Examples

### Runtime Settings File

Settings are stored in user config and can be overridden via CLI:

```bash
# Inline settings override
bun run dev --settings '{"provider":"github-copilot","autoSave":true}'
```

### Environment-Based Provider Override

```bash
# Force GitHub Models provider via env
export CLAUDE_CODE_PROVIDER="github-models"
bun run dev
```

### Multi-Provider Workflow

```bash
# Start with GitHub Models
bun run dev --settings '{"provider":"github-models"}'

# Inside CLI, switch to Copilot
/provider github-copilot

# Pick a specific model
/model
# (select claude-opus-4.6 from picker)
```

## Code Integration Examples

### TypeScript: Provider Interface

```typescript
import { GitHubModelsProvider } from './providers/github-models';

const provider = new GitHubModelsProvider({
  apiKey: process.env.GITHUB_TOKEN,
  model: 'openai/gpt-4.1',
});

const response = await provider.chat({
  messages: [
    { role: 'user', content: 'Explain this function' },
  ],
});

console.log(response.content);
```

### TypeScript: Copilot Provider

```typescript
import { GitHubCopilotProvider } from './providers/github-copilot';

const copilot = new GitHubCopilotProvider({
  token: process.env.GITHUB_TOKEN,
  model: 'claude-sonnet-4.6',
});

const result = await copilot.complete({
  prompt: 'Write a binary search in Rust',
  maxTokens: 500,
});
```

### TypeScript: Runtime Bootstrap

```typescript
import { bootstrap } from './cli/bootstrap';

async function main() {
  await bootstrap({
    provider: process.env.CLAUDE_CODE_PROVIDER || 'github-models',
    model: process.env.CLAUDE_CODE_MODEL,
    autoSave: true,
  });
}

main().catch(console.error);
```

## Common Patterns

### Pattern 1: Quick Provider Switch

```bash
# Start session
bun run dev

# Switch provider mid-session
/provider
# Select github-copilot from picker

# Verify active provider
/provider info
```

### Pattern 2: Scripted Model Selection

```bash
#!/bin/bash
export GITHUB_TOKEN=$(gh auth token)
bun run dev \
  --settings '{"provider":"github-copilot"}' \
  --model "claude-opus-4.6"
```

### Pattern 3: Fallback Chain

```typescript
// Automatic fallback if primary provider fails
const providers = [
  new GitHubCopilotProvider({ token: process.env.GITHUB_TOKEN }),
  new GitHubModelsProvider({ apiKey: process.env.GH_TOKEN }),
];

for (const provider of providers) {
  try {
    return await provider.chat(messages);
  } catch (err) {
    console.warn(`Provider ${provider.name} failed:`, err);
  }
}
throw new Error('All providers failed');
```

## Troubleshooting

### Issue: `bun install` Fails

**Cause:** Bun version too old or missing native dependencies.

**Fix:**
```bash
bun --version  # Must be 1.3.5+
bun upgrade
bun install --force
```

### Issue: Authentication Fails

**Cause:** No valid GitHub token found.

**Fix:**
```bash
# Verify token presence
gh auth status

# Re-authenticate
gh auth login

# Or manually set token
export GITHUB_TOKEN="ghp_..."
```

### Issue: Model Not Available

**Cause:** Model name mismatch or provider doesn't support the model.

**Fix:**
```bash
# Use /model picker instead of manual entry
bun run dev --settings '{"provider":"github-copilot"}'
# Then run: /model
```

### Issue: Copilot GPT Models Not Working

**Cause:** Copilot GPT/Grok models require `/responses` API (not yet fully wired).

**Fix:** Use Copilot-hosted Claude models instead:
```bash
bun run dev \
  --settings '{"provider":"github-copilot"}' \
  --model "claude-sonnet-4.6"
```

### Issue: Source Map Restoration Artifacts

**Cause:** Some modules are shims or degraded implementations.

**Symptoms:** Missing features, unexpected behavior in non-core paths.

**Fix:** Check if the feature relies on a private/native integration. File an issue on the repo if critical functionality is broken.

### Issue: Tool Loop Hangs

**Cause:** MCP compatibility layer returning malformed responses.

**Fix:**
```bash
# Enable debug logging
DEBUG=* bun run dev

# Check for MCP tool catalog errors
# Report specific tool failures to repo issues
```

## Development Workflow

### Run in Dev Mode

```bash
bun run dev
```

### Inspect Restored CLI Structure

```bash
bun run dev --help
# Shows full command tree from restored bootstrap
```

### Test Provider Switching

```typescript
// In your test suite
import { switchProvider } from './cli/commands/provider';

await switchProvider('github-copilot');
const info = await getProviderInfo();
assert(info.active === 'github-copilot');
```

## Key Files

- `cli/bootstrap.ts` – Restored CLI entry point
- `providers/github-models.ts` – GitHub Models provider
- `providers/github-copilot.ts` – GitHub Copilot provider
- `mcp/chrome-mcp.ts` – Chrome MCP compatibility layer
- `mcp/computer-use-mcp.ts` – Computer Use MCP shim

## Limitations

- Not the original upstream codebase—some modules are restored from source maps
- Copilot GPT/Grok models not fully integrated (require `/responses` API)
- Private/native integrations replaced with shims
- Some dynamic imports and resource files may be incomplete

## Resources

- **Repository:** https://github.com/oboard/claude-code-rev
- **Provider Docs:** Run `/provider info` in CLI for active provider details
- **Model Catalog:** Run `/model` to see available models for current provider
