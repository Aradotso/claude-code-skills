---
name: claude-code-config-trailofbits
description: Configure Claude Code with Trail of Bits opinionated defaults for sandboxing, permissions, hooks, skills, and MCP servers
triggers:
  - "set up claude code with trail of bits config"
  - "configure claude code sandboxing and permissions"
  - "install trail of bits claude code defaults"
  - "add claude code hooks and statusline"
  - "configure claude code global CLAUDE.md"
  - "set up claude code with privacy and security settings"
  - "configure claude code for security audits"
  - "set up ghostty terminal for claude code"
---

# Claude Code Config (Trail of Bits)

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

Trail of Bits' opinionated configuration for Claude Code covering sandboxing, permission deny rules, lifecycle hooks, statusline, global CLAUDE.md, and recommended tooling. This skill helps you install and configure the Trail of Bits defaults for secure, high-throughput AI-assisted development.

## What It Does

This project provides:

- **Privacy-first settings** — disables telemetry, error reporting, and feedback surveys
- **Security hardening** — permission deny rules blocking SSH keys, cloud credentials, shell config, crypto wallets
- **Lifecycle hooks** — `PreToolUse` blocks on `rm -rf` and direct push to main
- **Statusline** — two-line terminal status bar showing model, context usage, cost, cache hit rate
- **Global CLAUDE.md** — development philosophy, code quality limits, language-specific toolchains
- **Sandboxing guide** — built-in `/sandbox`, devcontainer, and remote droplet options
- **Recommended tools** — Ghostty terminal, `ruff`, `ast-grep`, `oxlint`, `cargo-deny`, etc.

## Installation

### Prerequisites

Install Ghostty (macOS):

```bash
brew install --cask ghostty
```

Install core tools:

```bash
# System tools
brew install jq ripgrep fd ast-grep shellcheck shfmt \
  actionlint zizmor macos-trash node@22 pnpm uv

# Python tools
uv tool install ruff
uv tool install ty
uv tool install pip-audit

# Rust toolchain
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
cargo install prek worktrunk cargo-deny cargo-careful

# Node tools
npm install -g oxlint agent-browser
```

### Clone and Run Setup

```bash
git clone https://github.com/trailofbits/claude-code-config.git
cd claude-code-config
claude
```

Inside the Claude Code session:

```
/trailofbits:config
```

This walks through installing each component, detects existing config, and self-installs the command for future runs.

### Manual Installation

If you prefer manual setup:

```bash
# 1. Copy settings
cp settings.json ~/.claude/settings.json

# 2. Copy global CLAUDE.md
cp claude-md-template.md ~/.claude/CLAUDE.md

# 3. Copy statusline script
mkdir -p ~/.claude
cp scripts/statusline.sh ~/.claude/statusline.sh
chmod +x ~/.claude/statusline.sh

# 4. Add shell aliases
echo 'alias claude-yolo="claude --dangerously-skip-permissions"' >> ~/.zshrc
```

## Settings Configuration

The `settings.json` includes:

### Privacy Environment Variables

```json
{
  "env": {
    "DISABLE_TELEMETRY": "1",
    "DISABLE_ERROR_REPORTING": "1",
    "CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY": "1"
  }
}
```

### Agent Teams (Experimental)

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

### Permission Deny Rules

```json
{
  "permissions": [
    {
      "action": "Read",
      "denied": true,
      "patterns": [
        "~/.ssh/**",
        "~/.gnupg/**",
        "~/.aws/**",
        "~/.azure/**",
        "~/.kube/**",
        "~/.docker/config.json",
        "~/.npmrc",
        "~/.npm/**",
        "~/.pypirc",
        "~/.gem/credentials",
        "~/.git-credentials",
        "~/.config/gh/**",
        "~/Library/Keychains/**",
        "~/Library/Application Support/Google/Chrome/Default/Login Data*",
        "~/Library/Application Support/Google/Chrome/Default/Cookies*"
      ]
    },
    {
      "action": "Edit",
      "denied": true,
      "patterns": [
        "~/.bashrc",
        "~/.zshrc",
        "~/.bash_profile",
        "~/.zprofile"
      ]
    }
  ]
}
```

### Hooks

```json
{
  "hooks": [
    {
      "type": "PreToolUse",
      "tool": "Bash",
      "command": "if echo \"$CLAUDE_TOOL_INPUT\" | grep -qE 'rm\\s+(-[^\\s]*r[^\\s]*f|[^\\s]*-[^\\s]*f[^\\s]*r)'; then echo 'Blocked: rm -rf detected. Use trash instead.'; exit 1; fi"
    },
    {
      "type": "PreToolUse",
      "tool": "Bash",
      "command": "if echo \"$CLAUDE_TOOL_INPUT\" | grep -qE 'git\\s+push.*\\s+(origin\\s+)?(main|master)'; then echo 'Blocked: Direct push to main/master. Use a branch and PR.'; exit 1; fi"
    }
  ]
}
```

### Other Settings

```json
{
  "enableAllProjectMcpServers": false,
  "alwaysThinkingEnabled": true,
  "cleanupPeriodDays": 365,
  "statusLine": "~/.claude/statusline.sh"
}
```

## Statusline Script

The statusline script (`~/.claude/statusline.sh`) generates a two-line status bar:

```
 [Opus 4.6] 📁 claude-code-config │ 🌿 main
 ████⣿⣿⣿⣿⣿⣿⣿⣿ 28% │ $0.83 │ ⏱ 12m 34s ↻89%
```

**Requirements:** `jq` must be installed.

**Example script structure:**

```bash
#!/bin/bash
# Reads from ~/.claude/session-info.json (provided by Claude Code)
# Outputs two lines to terminal

SESSION_FILE="$HOME/.claude/session-info.json"

if [[ ! -f "$SESSION_FILE" ]]; then
  echo ""
  echo ""
  exit 0
fi

MODEL=$(jq -r '.model // "Unknown"' "$SESSION_FILE")
FOLDER=$(basename "$(pwd)")
BRANCH=$(git rev-parse --abbrev-ref HEAD 2>/dev/null || echo "no-git")

# Line 1: model, folder, branch
echo " [$MODEL] 📁 $FOLDER │ 🌿 $BRANCH"

# Line 2: context bar, cost, time, cache hit rate
# (actual implementation reads context usage, cost, elapsed time from session file)
echo " ████⣿⣿⣿⣿⣿⣿⣿⣿ 28% │ \$0.83 │ ⏱ 12m 34s ↻89%"
```

## Global CLAUDE.md

The global `~/.claude/CLAUDE.md` sets default instructions for all sessions. Key sections:

### Development Philosophy

```markdown
## Philosophy

- **No speculative features** — build what's asked, not what might be needed
- **No premature abstraction** — repeat yourself until the pattern is clear
- **Replace don't deprecate** — cut old code instead of maintaining compatibility layers
- **One entry point per decision** — DRY for logic, not for structure
```

### Code Quality Hard Limits

```markdown
## Hard Limits

- Function length: 50 lines (excluding tests)
- Cyclomatic complexity: 10
- Line width: 100 characters
- Function parameters: 5
- Nesting depth: 4
```

### Language Toolchains

**Python:**

```markdown
## Python

- Package manager: `uv` (not pip, not poetry)
- Linter: `ruff check` (not pylint, not flake8)
- Formatter: `ruff format` (not black)
- Type checker: `ty` (not mypy)
- Security: `pip-audit`
- Test runner: `pytest`
```

**Node/TypeScript:**

```markdown
## Node/TypeScript

- Package manager: `pnpm` (not npm, not yarn)
- Linter: `oxlint` (not eslint)
- Formatter: `prettier`
- Type checker: `tsc`
- Test runner: `vitest` (not jest)
```

**Rust:**

```markdown
## Rust

- Linter: `cargo clippy`
- Formatter: `cargo fmt`
- Security: `cargo deny`, `cargo audit`
- Test runner: `cargo test`, `cargo careful`
```

**Bash:**

```markdown
## Bash

- Linter: `shellcheck`
- Formatter: `shfmt`
- Always use `set -euo pipefail`
- Quote all variables: `"$var"` not `$var`
```

## Sandboxing

### Built-in Sandbox

Enable the built-in sandbox:

```
/sandbox
```

This provides:

- **Filesystem isolation** — writes restricted to current directory
- **Network isolation** — limited to allowed domains
- **OS-level enforcement** — Seatbelt (macOS) or bubblewrap (Linux)

With `/sandbox` enabled + permission deny rules, Bash commands are blocked from reading SSH keys, cloud credentials, etc.

### Devcontainer

For full read/write isolation:

```bash
git clone https://github.com/trailofbits/claude-code-devcontainer.git
cd claude-code-devcontainer
# Follow README to launch container
```

The agent runs in a container with only project files mounted — zero host filesystem access.

### Remote Droplet

For complete machine-level isolation:

```bash
# Install dropkit
git clone https://github.com/trailofbits/dropkit.git
cd dropkit

# Create droplet
./dropkit create my-dev-box

# SSH in
./dropkit ssh my-dev-box

# Run Claude Code in droplet
claude

# Destroy when done
./dropkit destroy my-dev-box
```

## Shell Aliases

### Bypass Permissions (Recommended)

```bash
# Add to ~/.zshrc
alias claude-yolo="claude --dangerously-skip-permissions"
```

Use `claude-yolo` instead of `claude` for maximum throughput. Requires sandboxing.

### Local Models

```bash
# Add to ~/.zshrc
claude-local() {
  ANTHROPIC_BASE_URL=http://localhost:1234 \
  ANTHROPIC_AUTH_TOKEN=lmstudio \
  CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=1 \
  claude --model qwen/qwen3-coder-next "$@"
}
```

Requires LM Studio running locally:

```bash
# Install LM Studio
curl -fsSL https://lmstudio.ai/install.sh | bash

# Or download desktop app
# https://lmstudio.ai/download
```

## Hooks Deep Dive

Hooks fire at lifecycle decision points. Three types:

### PreToolUse

Fires before Claude executes a tool. Can block execution.

**Example: Block `rm -rf`**

```bash
if echo "$CLAUDE_TOOL_INPUT" | grep -qE 'rm\s+(-[^\s]*r[^\s]*f|-[^\s]*f[^\s]*r)'; then
  echo 'Blocked: rm -rf detected. Use trash instead.'
  exit 1
fi
```

**Example: Block direct push to main**

```bash
if echo "$CLAUDE_TOOL_INPUT" | grep -qE 'git\s+push.*(origin\s+)?(main|master)'; then
  echo 'Blocked: Direct push to main/master. Use a branch and PR.'
  exit 1
fi
```

**Example: LLM prompt hook (check test coverage)**

```json
{
  "type": "PreToolUse",
  "tool": "Bash",
  "prompt": "If this command runs tests, confirm that it includes coverage reporting and that coverage is above 80%. If not, suggest adding coverage flags."
}
```

### PostToolUse

Fires after Claude executes a tool. Can inject context.

**Example: Remind about commit message format**

```json
{
  "type": "PostToolUse",
  "tool": "Bash",
  "prompt": "If the command was 'git commit', remind the user that commit messages should follow Conventional Commits format: type(scope): subject"
}
```

### Stop

Fires when Claude thinks the task is complete.

**Example: Check for todos before finishing**

```bash
if grep -r "TODO\\|FIXME\\|XXX" .; then
  echo 'Found TODOs in code. Review before marking complete.'
  exit 1
fi
```

## Project-Level CLAUDE.md

Projects can have their own `CLAUDE.md` that overrides or extends global rules:

```bash
# In your project root
cat > CLAUDE.md << 'EOF'
# Project: Crypto Auditing Tool

## Context

This is a security auditing tool for smart contracts. Prioritize:
- No dependencies outside stdlib where possible
- All inputs treated as untrusted
- Detailed logging for audit trails

## Toolchain

- Use `slither` for Solidity analysis
- Use `mythril` for symbolic execution
- Generate reports in Markdown format

## Testing

- All findings must be reproducible with test cases
- Include proof-of-concept exploits in `tests/exploits/`
EOF
```

Global `~/.claude/CLAUDE.md` still applies — project file adds to it.

## Commands

Inside a Claude Code session:

```
/trailofbits:config    # Run setup wizard
/sandbox               # Enable built-in sandbox
/thinking              # Toggle extended thinking (or Option+T)
/insights              # Show productivity insights (requires >30 days history)
/terminal-setup        # Configure terminal key bindings (not needed for Ghostty)
```

## Skills and MCP Servers

### Installing Skills

Trail of Bits maintains curated skill collections:

```bash
# Clone skills repo
git clone https://github.com/trailofbits/skills.git ~/.claude/skills

# Or curated subset
git clone https://github.com/trailofbits/skills-curated.git ~/.claude/skills
```

Skills auto-load when Claude Code starts. Reference in global CLAUDE.md:

```markdown
## Skills

Load skills from ~/.claude/skills/ for:
- Security auditing workflows
- Rust development patterns
- Smart contract analysis
```

### MCP Servers

**Security note:** Set `enableAllProjectMcpServers: false` in settings. Project `.mcp.json` files can ship malicious servers.

**Recommended global MCP servers** (add to `~/.mcp.json`):

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/allowed/dir"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

Only install MCP servers you trust. They run with full Claude Code permissions.

## Troubleshooting

### Statusline not showing

Check that `jq` is installed:

```bash
which jq || brew install jq
```

Verify script is executable:

```bash
chmod +x ~/.claude/statusline.sh
```

### Permission deny rules not working

Deny rules only block Claude's built-in tools. To enforce at OS level:

```
/sandbox
```

### Hooks not firing

Check hook syntax in `~/.claude/settings.json`:

```bash
jq '.hooks' ~/.claude/settings.json
```

Test hook command manually:

```bash
CLAUDE_TOOL_INPUT="rm -rf /tmp/test" bash -c 'if echo "$CLAUDE_TOOL_INPUT" | grep -qE "rm\\s+(-[^\\s]*r[^\\s]*f|-[^\\s]*f[^\\s]*r)"; then echo "Blocked"; exit 1; fi'
```

### Global CLAUDE.md ignored

Check file exists:

```bash
cat ~/.claude/CLAUDE.md
```

Restart Claude Code session to reload.

### Ghostty Shift+Enter not working

Ghostty supports Shift+Enter out of the box — no `/terminal-setup` needed. If it's not working, check Ghostty version:

```bash
ghostty --version
```

Update to latest:

```bash
brew upgrade ghostty
```

## Best Practices

1. **Always use sandboxing** — either `/sandbox`, devcontainer, or remote droplet
2. **Run in bypass mode** — `claude-yolo` for maximum throughput (requires sandboxing)
3. **Review global CLAUDE.md monthly** — update toolchain, limits, and patterns as your workflow evolves
4. **Add project-specific CLAUDE.md** — don't put project rules in global file
5. **Test hooks before deploying** — a bad hook can block all Bash commands
6. **Use Ghostty** — prevents memory bloat and lag in long sessions
7. **Keep history long** — set `cleanupPeriodDays: 365` for better `/insights`
8. **Don't bypass deny rules** — if Claude asks to read `~/.ssh`, it's a red flag

## Real-World Examples

### Security Audit Workflow

```bash
# 1. Create isolated droplet
dropkit create audit-project-x
dropkit ssh audit-project-x

# 2. Clone target repo
git clone https://github.com/target/repo.git
cd repo

# 3. Add project CLAUDE.md
cat > CLAUDE.md << 'EOF'
# Security Audit: Project X

Run these tools in order:
1. `slither .` for Solidity analysis
2. `mythril analyze contracts/*.sol` for symbolic execution
3. `cargo clippy` for Rust components
4. Generate findings report in `audit-report.md`
EOF

# 4. Start Claude in bypass mode with sandbox
claude-yolo
/sandbox

# 5. Ask: "Run full security audit per CLAUDE.md"
```

### Python Library Development

```bash
# 1. Start with devcontainer
git clone https://github.com/trailofbits/claude-code-devcontainer.git my-lib
cd my-lib

# 2. Add project CLAUDE.md
cat > CLAUDE.md << 'EOF'
# Python Library: My Lib

## Toolchain
- `uv` for package management
- `ruff` for linting and formatting
- `ty` for type checking
- `pytest` with coverage >90%

## Structure
- `src/my_lib/` for code
- `tests/` for tests (mirror src structure)
- `examples/` for usage examples
EOF

# 3. Start Claude
claude-yolo

# 4. Ask: "Create new Python library per CLAUDE.md with CLI and API"
```

### Rust CLI Tool

```bash
# Project CLAUDE.md for Rust CLI
cat > CLAUDE.md << 'EOF'
# Rust CLI: My Tool

## Requirements
- `clap` for arg parsing
- `anyhow` for error handling
- `tokio` for async runtime
- All deps pass `cargo deny` (no GPL, no vulnerabilities)

## Testing
- Unit tests for all logic
- Integration tests in `tests/`
- `cargo careful` for unsafe code checks
EOF
```

## Additional Resources

- [Best practices for Claude Code](https://code.claude.com/docs/en/best-practices)
- [Sandboxing docs](https://code.claude.com/docs/en/sandboxing)
- [Agent teams](https://code.claude.com/docs/en/agent-teams)
- [Manage Claude's memory](https://code.claude.com/docs/en/memory)
- [Trail of Bits skills](https://github.com/trailofbits/skills)
- [Trail of Bits devcontainer](https://github.com/trailofbits/claude-code-devcontainer)
- [Dropkit](https://github.com/trailofbits/dropkit)
