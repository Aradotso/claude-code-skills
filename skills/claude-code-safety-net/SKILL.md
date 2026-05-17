---
name: claude-code-safety-net
description: Safety net plugin that blocks destructive git and filesystem commands in Claude Code, OpenCode, Gemini CLI, Copilot CLI, and Codex
triggers:
  - install the safety net plugin
  - protect against destructive commands
  - block dangerous git operations
  - prevent accidental file deletions
  - configure safety net modes
  - check safety net status
  - add custom safety rules
  - enable strict mode protection
---

# Claude Code Safety Net

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

A PreToolUse hook plugin that intercepts and blocks destructive git and filesystem commands before they execute. Works across Claude Code, OpenCode, Gemini CLI, Copilot CLI, and Codex.

## What It Does

Safety Net analyzes commands before execution and blocks operations that could:
- Delete uncommitted work (`git reset --hard`, `git checkout -- .`)
- Force push to remotes (`git push --force`)
- Remove files recursively (`rm -rf /`)
- Clear git stashes (`git stash clear`)
- Execute shell wrappers containing destructive commands (`bash -c "rm -rf /"`)
- Run interpreter one-liners (`python -c 'os.system("rm -rf /")'`)

It runs **before** permission rules and provides semantic command analysis that wildcards can't match.

## Installation

### Claude Code

```bash
/plugin marketplace add kenryu42/cc-marketplace
/plugin install safety-net@cc-marketplace
/reload-plugins
```

Enable auto-updates:
```bash
/plugin
# Select Marketplaces → cc-marketplace → Enable auto-update
```

### OpenCode

Add to `~/.config/opencode/opencode.json`:

```json
{
  "plugin": ["cc-safety-net"]
}
```

### Gemini CLI

```bash
gemini extensions install https://github.com/kenryu42/gemini-safety-net
```

### GitHub Copilot CLI

```bash
/plugin install kenryu42/copilot-safety-net
```

Restart Copilot CLI after installation.

### Codex

Enable hooks in `~/.codex/config.toml`:

```toml
[features]
plugin_hooks = true
```

Add marketplace and install:

```bash
codex plugin marketplace add kenryu42/cc-marketplace
```

In Codex TUI:
1. Run `/plugins`
2. Select `[cc-marketplace]` and press Enter
3. Run `/hooks`
4. Select safety-net PreToolUse hook and press `t` to trust

## Configuration

### Status Line Integration

Add to `~/.claude/settings.json`:

```json
{
  "statusLine": {
    "type": "command",
    "command": "bunx cc-safety-net --statusline"
  }
}
```

Or pipe with existing status:

```json
{
  "statusLine": {
    "type": "command",
    "command": "git branch --show-current | bunx cc-safety-net --statusline"
  }
}
```

Status indicators:
- `🛡️ Safety Net ❌` - Plugin disabled
- `🛡️ Safety Net ✅` - Default mode active
- `🛡️ Safety Net ✅ 🔒` - Strict mode enabled
- `🛡️ Safety Net ✅ 👁️` - Paranoid mode enabled
- `🛡️ Safety Net ✅ 🌳` - Worktree mode enabled

### Custom Rules

Create `~/.config/cc-safety-net/rules.json`:

```json
{
  "rules": [
    {
      "name": "block-npm-publish",
      "pattern": "npm publish",
      "action": "block",
      "reason": "Use CI/CD pipeline for publishing"
    },
    {
      "name": "allow-temp-cleanup",
      "pattern": "rm -rf /tmp/myapp-*",
      "action": "allow",
      "reason": "Safe to clean temp files"
    },
    {
      "name": "block-production-deploys",
      "pattern": "kubectl.*--context=prod",
      "action": "block",
      "reason": "Production deploys require manual approval"
    }
  ]
}
```

Rule schema:

```typescript
interface Rule {
  name: string;           // Unique identifier
  pattern: string;        // Regex pattern to match
  action: "block" | "allow";
  reason: string;         // Why this rule exists
}
```

Matching behavior:
- Rules are evaluated in order
- First matching rule wins
- Custom rules override built-in rules
- Invalid patterns are logged and skipped

### Advanced Modes

#### Strict Mode

Blocks additional destructive operations:

```json
{
  "safety-net": {
    "strict": true
  }
}
```

Blocked in strict mode:
- `git clean -fd` - Deletes untracked files
- `git branch -D` - Force deletes branches
- `git remote remove` - Removes remote repositories

#### Paranoid Mode

Blocks all potentially destructive commands, even if they have safe variants:

```json
{
  "safety-net": {
    "paranoid": true
  }
}
```

Blocked in paranoid mode:
- All `git checkout` commands (including safe branch switches)
- All `git reset` commands (including `--soft`)
- All `rm` commands with `-r` flag

#### Worktree Mode

Provides additional protection when using git worktrees:

```json
{
  "safety-net": {
    "worktree": true
  }
}
```

Blocks worktree-specific destructive operations:
- Removing worktrees with uncommitted changes
- Force operations that affect multiple worktrees

### Secret Redaction

Automatically redacts sensitive values from logs:

```bash
# Command executed:
curl -H "Authorization: Bearer sk-1234..." https://api.example.com

# Logged as:
curl -H "Authorization: Bearer [REDACTED]" https://api.example.com
```

Patterns redacted:
- API keys (e.g., `sk-`, `api_key=`)
- Tokens (e.g., `Bearer`, `token=`)
- Passwords (e.g., `password=`, `-p`)
- AWS credentials
- SSH keys

### Audit Logging

Enable detailed command logging in `~/.claude/settings.json`:

```json
{
  "safety-net": {
    "audit": {
      "enabled": true,
      "logFile": "~/.config/cc-safety-net/audit.log"
    }
  }
}
```

Log format:

```json
{
  "timestamp": "2025-01-15T10:30:45.123Z",
  "command": "git reset --hard HEAD~1",
  "action": "blocked",
  "reason": "Destructive git reset detected",
  "mode": "default",
  "cwd": "/home/user/project"
}
```

## Commands Blocked

### Git Operations

```bash
# Destructive resets
git reset --hard
git reset --hard HEAD~1

# Discard changes
git checkout -- .
git checkout -- file.txt

# Force push
git push --force
git push -f origin main

# Stash operations
git stash clear
git stash drop

# Strict mode only
git clean -fd
git branch -D feature
git remote remove origin
```

### Filesystem Operations

```bash
# Recursive deletion of important directories
rm -rf /
rm -rf ~/
rm -rf .
rm -rf .git

# Allow safe cleanup
rm -rf /tmp/cache  # ✅ Allowed
rm -rf node_modules  # ✅ Allowed
```

### Shell Wrappers (5 levels deep)

```bash
# Direct wrapper
bash -c "rm -rf /"

# Nested wrappers
sh -c "bash -c 'rm -rf /'"
eval "sh -c 'bash -c \"rm -rf /\"'"
```

### Interpreter One-Liners

```bash
# Python
python -c 'import os; os.system("rm -rf /")'
python3 -c 'import shutil; shutil.rmtree("/")'

# Node.js
node -e 'require("child_process").execSync("rm -rf /")'

# Ruby
ruby -e 'system("rm -rf /")'

# Perl
perl -e 'system("rm -rf /")'
```

## Commands Allowed

```bash
# Safe git operations
git checkout -b feature
git checkout main
git reset --soft HEAD~1
git push origin feature
git stash
git stash pop

# Safe filesystem operations
rm file.txt
rm -r /tmp/myapp-cache
rm -rf node_modules
mv old.txt new.txt
cp -r src/ backup/

# Safe git workflow
git add .
git commit -m "feat: add feature"
git push origin feature
git pull --rebase
git merge feature
```

## Diagnostics

Check plugin status and configuration:

```bash
bunx cc-safety-net --diagnose
```

Output example:

```
🛡️  Safety Net Diagnostics

✓ Plugin installed and enabled
✓ PreToolUse hook registered
✓ Status line: Active

Configuration:
  Mode: strict
  Worktree: disabled
  Paranoid: disabled
  Audit log: enabled (~/.config/cc-safety-net/audit.log)

Custom Rules:
  ✓ block-npm-publish
  ✓ allow-temp-cleanup
  ✗ invalid-rule (Pattern compile error)

Recent Blocks (last 24h):
  - git reset --hard (2025-01-15 10:30:45)
  - rm -rf / (2025-01-15 09:15:22)
```

## Debug Analysis

Explain why a command was blocked or allowed:

```bash
bunx cc-safety-net --explain "git reset --hard"
```

Output:

```
Command: git reset --hard
Action: 🚫 BLOCKED
Reason: Destructive git reset detected

Analysis:
  - Command: git reset --hard
  - Flags: --hard
  - Pattern matched: Destructive reset (discards uncommitted changes)
  - Mode: default
  - Custom rules: none matched

Safe alternatives:
  - git reset --soft HEAD~1 (keeps changes staged)
  - git stash (temporarily saves changes)
  - git commit (save changes permanently)
```

## Testing the Hook

Verify the plugin is working:

```bash
# Should be blocked
git reset --hard
rm -rf /tmp/test-deleteme

# Should be allowed
git checkout -b test-branch
rm -rf /tmp/cache
```

Expected behavior when blocked:

```
🛡️ Safety Net: Command blocked

Command: git reset --hard
Reason: Destructive git reset detected

This command would discard all uncommitted changes.

Safe alternatives:
  - git reset --soft HEAD~1
  - git stash
  - git commit
```

## Troubleshooting

### Plugin not blocking commands

1. Verify installation:

```bash
bunx cc-safety-net --diagnose
```

2. Check hook registration in Claude Code:

```bash
/plugins
# Verify safety-net shows as enabled
```

3. Ensure PreToolUse hook is trusted (Codex only):

```bash
/hooks
# Select safety-net and press 't'
```

### Custom rules not working

1. Validate JSON syntax:

```bash
cat ~/.config/cc-safety-net/rules.json | jq .
```

2. Check pattern syntax:

```bash
bunx cc-safety-net --explain "your command here"
```

3. Review audit log:

```bash
tail -f ~/.config/cc-safety-net/audit.log
```

### Status line not updating

1. Verify command runs standalone:

```bash
bunx cc-safety-net --statusline
```

2. Check settings.json syntax:

```bash
cat ~/.claude/settings.json | jq .statusLine
```

3. Restart Claude Code if using native version

### False positives

Use custom rules to allow specific patterns:

```json
{
  "rules": [
    {
      "name": "allow-my-script",
      "pattern": "rm -rf /tmp/myapp-build-*",
      "action": "allow",
      "reason": "Safe cleanup pattern"
    }
  ]
}
```

### Shell wrapper not detected

Safety Net recursively analyzes 5 levels of shell wrappers. If a command still bypasses:

1. Report with example:

```bash
bunx cc-safety-net --explain "your bypassed command"
```

2. Add temporary custom rule while waiting for fix:

```json
{
  "rules": [
    {
      "name": "block-wrapper",
      "pattern": "your-wrapper-pattern",
      "action": "block",
      "reason": "Temporary workaround"
    }
  ]
}
```

## Real-World Examples

### Prevent accidental force push

```bash
# Blocked automatically
git push --force origin main

# Safe alternative shown
git push --force-with-lease origin main
```

### Clean build artifacts safely

```json
{
  "rules": [
    {
      "name": "allow-build-cleanup",
      "pattern": "rm -rf (dist|build|.next|out|target)/",
      "action": "allow",
      "reason": "Safe to clean build directories"
    }
  ]
}
```

### Block production deployments

```json
{
  "rules": [
    {
      "name": "block-prod-deploy",
      "pattern": "(kubectl|terraform).*prod",
      "action": "block",
      "reason": "Use CI/CD for production"
    }
  ]
}
```

### Team workspace protection

```json
{
  "safety-net": {
    "strict": true,
    "worktree": true,
    "audit": {
      "enabled": true,
      "logFile": "~/team-audit.log"
    }
  },
  "rules": [
    {
      "name": "block-db-drops",
      "pattern": "DROP (DATABASE|TABLE)",
      "action": "block",
      "reason": "Use migration system"
    }
  ]
}
```
