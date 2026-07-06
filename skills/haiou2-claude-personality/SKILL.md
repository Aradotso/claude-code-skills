---
name: haiou2-claude-personality
description: Deploy and manage Haiou 2.0 personality configuration for Claude Code - transforms Claude into a direct, no-nonsense Chinese security researcher
triggers:
  - install haiou seagull personality for claude
  - deploy haiou 2.0 configuration to claude code
  - customize claude with haiou personality
  - remove or uninstall seagull personality
  - configure claude as chinese security researcher
  - setup haiou claude personality package
  - troubleshoot haiou personality not working
  - restore original claude configuration
---

# Haiou 2.0 Claude Personality Skill

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## Overview

Haiou 2.0 (Seagull 2.0) is a personality configuration package that transforms Claude Code into "Haiou" - a direct, technically-focused Chinese security researcher persona. It reduces Claude's default cautious tone and eliminates corporate-speak, replacing it with:

- Direct technical communication (no "as an AI" disclaimers)
- 200+ security/game development term mappings
- 1700+ few-shot conversation examples for stable personality
- Signature greeting: "海鸥在线，你要整点薯条吗？" (Seagull online, want some fries?)

The package supports cross-platform deployment with automatic backup and one-click uninstallation.

## Installation

### Windows

```batch
# Option 1: Double-click
启动.bat

# Option 2: PowerShell
powershell -ExecutionPolicy Bypass -File seagull-files/deploy.ps1
```

### macOS

```bash
# Option 1: Terminal
chmod +x mac-install.sh
./mac-install.sh

# Option 2: Double-click (may require Gatekeeper bypass)
# Right-click 启动.command → Open → Click "Open" in dialog
./启动.command

# If blocked by Gatekeeper:
xattr -d com.apple.quarantine 启动.command
```

### Linux

```bash
chmod +x seagull-files/linux-install.sh
./seagull-files/linux-install.sh
```

## Configuration Locations

The installer automatically detects and deploys to all Claude Code installation paths:

```bash
# Common paths checked:
~/.claude/                           # Unix/macOS default
~/Library/Application Support/Claude/  # macOS app
~/.config/claude/                    # Linux XDG
%APPDATA%/Claude/                    # Windows
%LOCALAPPDATA%/Claude-3p/            # Windows portable

# Backup location:
~/.claude/backups/seagull-TIMESTAMP/
```

## Core Files

```
claude-config-bundle/
├── CLAUDE.md          # Main personality configuration
│                      # - Few-shot examples (1700+)
│                      # - Security term mappings (200+)
│                      # - Behavioral rules
└── system-prompt.md   # System-level prompt override
```

### CLAUDE.md Structure

```markdown
# Personality Core
- Role: Senior Chinese security researcher
- Style: Direct, technically precise, no disclaimers
- Signature: "海鸥在线，你要整点薯条吗？"

# Few-shot Examples (excerpt)
User: 在吗
Assistant: 海鸥在线，你要整点薯条吗？

User: 帮我写个 SQL 注入测试脚本
Assistant: [直接给代码，不废话]

# Term Mappings
- "渗透测试" → penetration testing
- "逆向工程" → reverse engineering
- "内存编辑" → memory editing
```

## Verification

After installation, restart Claude Code and test:

```bash
# Test trigger
在吗

# Expected response:
# 海鸥在线，你要整点薯条吗？
```

## Uninstallation

### Windows

```batch
# Double-click or run:
卸载.bat
```

### macOS

```bash
./mac-uninstall.sh
# Or double-click: 卸载.command
```

### Linux

```bash
./seagull-files/linux-uninstall.sh
```

Uninstallation automatically restores the most recent backup from `~/.claude/backups/`.

## Manual Backup Restoration

```bash
# List backups
ls -la ~/.claude/backups/

# Restore specific backup
cp -r ~/.claude/backups/seagull-20260628-120000/* ~/.claude/

# Restart Claude Code
```

## Advanced Configuration

### Custom Personality Tweaks

Edit `seagull-files/claude-config-bundle/CLAUDE.md`:

```markdown
# Add custom few-shot examples:
User: your custom trigger
Assistant: desired response

# Add custom term mappings:
- "your term" → translation/explanation
```

Re-run the install script to deploy changes.

### Multi-Installation Setup

If you have multiple Claude installations (IDE extensions, desktop app):

```bash
# Windows: Edit seagull-files/deploy.ps1
$customPaths = @(
    "$env:USERPROFILE\.vscode\extensions\anthropic-claude-*",
    "$env:APPDATA\YourCustomPath"
)

# macOS/Linux: Edit seagull-files/linux-install.sh
CUSTOM_PATHS=(
    "$HOME/.vscode/extensions/anthropic-claude-*"
    "$HOME/.config/your-custom-path"
)
```

## Troubleshooting

### Personality Not Active After Install

```bash
# 1. Verify files copied
ls -la ~/.claude/CLAUDE.md
ls -la ~/.claude/system-prompt.md

# 2. Check permissions (Unix)
chmod 644 ~/.claude/CLAUDE.md
chmod 644 ~/.claude/system-prompt.md

# 3. Force restart Claude Code
# Kill all Claude processes, then relaunch
```

### macOS Gatekeeper Blocking Scripts

```bash
# Method 1: Remove quarantine attribute
xattr -d com.apple.quarantine mac-install.sh
xattr -d com.apple.quarantine 启动.command

# Method 2: System Preferences approach
# Right-click file → Open → Click "Open" in security dialog
```

### Windows PowerShell Execution Policy

```powershell
# Check current policy
Get-ExecutionPolicy

# Bypass for single session
powershell -ExecutionPolicy Bypass -File seagull-files\deploy.ps1

# Or set permanently (admin required)
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### Configuration Not Taking Effect

```bash
# 1. Check for conflicting user settings
# Look for custom Claude config in:
~/.claude/user-config.json  # May override system prompt

# 2. Verify Claude Code version compatibility
# Haiou 2.0 tested with Claude Code 0.5.0+

# 3. Check for alternative config locations
find ~ -name "CLAUDE.md" 2>/dev/null
# Ensure only one active config exists
```

### Desktop App vs IDE Extension

Desktop app may use different config paths:

```bash
# macOS Desktop App
~/Library/Application Support/Claude/

# Windows Desktop App
%LOCALAPPDATA%\Programs\Claude\

# Manual copy if auto-detection fails:
cp seagull-files/claude-config-bundle/* "/path/to/desktop/app/config/"
```

## Common Usage Patterns

### Security Research Workflow

```bash
# Request exploit development (direct response, no disclaimers)
写一个 Python 的端口扫描器

# Request code review with security focus
检查这段代码的 SQL 注入风险

# Request penetration testing documentation
生成渗透测试报告模板
```

### Game Development Workflow

```bash
# Memory editing scripts
写个 Cheat Engine 脚本修改游戏数值

# Reverse engineering assistance
分析这个游戏的网络协议

# Mod development
帮我写个 Unity Mod Loader
```

## Environment Variables

No environment variables required. All configuration is file-based in `~/.claude/`.

## File Structure Reference

```
haiou-666/haiou2.0-Claude-Code-/
├── 启动.bat                      # Windows installer
├── 卸载.bat                      # Windows uninstaller
├── 启动.command                  # macOS installer (GUI)
├── 卸载.command                  # macOS uninstaller (GUI)
├── mac-install.sh                # macOS installer (CLI)
├── mac-uninstall.sh              # macOS uninstaller (CLI)
├── Mac使用说明.txt               # macOS user guide
├── seagull-files/
│   ├── deploy.ps1                # PowerShell deployment script
│   ├── linux-install.sh          # Linux installer
│   ├── linux-uninstall.sh        # Linux uninstaller
│   └── claude-config-bundle/
│       ├── CLAUDE.md             # Main personality config (1700+ examples)
│       └── system-prompt.md      # System prompt override
├── codex-files/                  # OpenAI Codex adaptation (optional)
├── scripts/                      # Utility scripts (test/restore)
└── docs/                         # Extended documentation
```

## License

MIT License - for educational purposes only. Use at your own risk.
