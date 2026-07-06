---
name: haiou-claude-personality-mod
description: Install and manage Haiou 2.0, a custom personality configuration for Claude Code that transforms it into "Seagull" — a direct, no-nonsense Chinese security researcher persona
triggers:
  - install haiou seagull personality
  - configure claude with haiou mod
  - deploy seagull personality to claude
  - modify claude code personality
  - install seagull 2.0 configuration
  - customize claude ai persona
  - troubleshoot haiou installation
  - uninstall seagull personality mod
---

# Haiou Claude Personality Mod

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## Overview

Haiou 2.0 (Seagull 2.0) is a custom personality configuration package for Claude Code that replaces the default assistant persona with "Seagull" — a direct, technically-focused Chinese security researcher character. The mod includes:

- **1700+ few-shot dialogue examples** for stable personality
- **200+ security/gaming terminology mappings** (pentesting, reverse engineering, game dev)
- **Custom system prompts** that reduce verbose disclaimers
- **Cross-platform deployment scripts** (Windows/macOS/Linux)
- **Automatic backup** of original configuration
- **Signature greeting**: "海鸥在线，你要整点薯条吗？" (Seagull online, want some fries?)

## Installation

### Windows

```batch
# Double-click the launcher
启动.bat

# Or run PowerShell script directly
powershell -ExecutionPolicy Bypass -File seagull-files/deploy.ps1
```

### macOS

```bash
# Make installer executable
chmod +x mac-install.sh
./mac-install.sh

# Or double-click
# 启动.command
```

### Linux

```bash
# Navigate to seagull-files directory
cd seagull-files
chmod +x linux-install.sh
./linux-install.sh
```

## File Structure

```
海鸥2.0/
├── 启动.bat                      # Windows installer
├── 卸载.bat                      # Windows uninstaller
├── 启动.command                  # macOS installer (GUI)
├── 卸载.command                  # macOS uninstaller (GUI)
├── mac-install.sh                # macOS installer (CLI)
├── mac-uninstall.sh              # macOS uninstaller (CLI)
├── seagull-files/
│   ├── deploy.ps1                # PowerShell deployment script
│   ├── linux-install.sh          # Linux installer
│   ├── linux-uninstall.sh        # Linux uninstaller
│   └── claude-config-bundle/
│       ├── CLAUDE.md             # Personality configuration
│       └── system-prompt.md      # System prompt overrides
└── codex-files/                  # OpenAI Codex compatibility (optional)
```

## Configuration Files

### CLAUDE.md

The main personality configuration file located at `~/.claude/CLAUDE.md` (Linux/macOS) or `%APPDATA%\Local\Claude-3p\CLAUDE.md` (Windows). Contains:

- Character background and traits
- 1700+ few-shot examples
- Response style guidelines
- Technical terminology mappings

### system-prompt.md

System-level prompt overrides at `~/.claude/system-prompt.md` that:

- Reduce "As an AI" disclaimers
- Enable direct technical responses
- Configure Chinese language preference
- Set security research context

## Verification

After installation, restart Claude Code and test:

```
User: 在吗
Seagull: 海鸥在线，你要整点薯条吗？
```

## Key Deployment Script Features

### Windows (deploy.ps1)

```powershell
# Check for Claude Code installation
$claudePaths = @(
    "$env:USERPROFILE\.claude",
    "$env:LOCALAPPDATA\Claude-3p",
    "$env:APPDATA\Claude"
)

# Backup existing configuration
$backupDir = "$env:USERPROFILE\.claude\backups\seagull-$(Get-Date -Format 'yyyyMMdd-HHmmss')"
New-Item -ItemType Directory -Force -Path $backupDir

# Deploy configuration files
Copy-Item -Path "claude-config-bundle\*" -Destination "$HOME\.claude\" -Force
```

### Linux/macOS (linux-install.sh)

```bash
#!/bin/bash

# Detect Claude Code installation
CLAUDE_DIR="$HOME/.claude"
BACKUP_DIR="$CLAUDE_DIR/backups/seagull-$(date +%Y%m%d-%H%M%S)"

# Create backup
if [ -d "$CLAUDE_DIR" ]; then
    mkdir -p "$BACKUP_DIR"
    cp -r "$CLAUDE_DIR"/*.md "$BACKUP_DIR/" 2>/dev/null || true
fi

# Deploy configuration
cp claude-config-bundle/*.md "$CLAUDE_DIR/"
chmod 644 "$CLAUDE_DIR"/*.md
```

## Uninstallation

### Windows

```batch
# Double-click uninstaller
卸载.bat

# Or run PowerShell script
powershell -ExecutionPolicy Bypass -File seagull-files/uninstall.ps1
```

### macOS

```bash
./mac-uninstall.sh
# Or double-click: 卸载.command
```

### Linux

```bash
cd seagull-files
./linux-uninstall.sh
```

Uninstallation restores the most recent backup from `~/.claude/backups/`.

## Manual Configuration Restoration

If automatic uninstall fails:

```bash
# Linux/macOS
cd ~/.claude/backups
ls -lt  # Find most recent backup
cp seagull-YYYYMMDD-HHMMSS/* ~/.claude/

# Windows PowerShell
cd $env:USERPROFILE\.claude\backups
# Copy files from latest backup folder to parent directory
```

## Troubleshooting

### Configuration Not Taking Effect

**Problem**: Claude Code shows no personality change after installation.

**Solution**:
```bash
# 1. Verify files were deployed
ls -la ~/.claude/  # Should show CLAUDE.md and system-prompt.md

# 2. Check file permissions
chmod 644 ~/.claude/*.md

# 3. Fully restart Claude Code (quit application, not just close window)
# Windows: Task Manager → End Process
# macOS: Cmd+Q to fully quit
# Linux: killall claude-code
```

### macOS Gatekeeper Warnings

**Problem**: "Cannot verify developer" when running `.command` files.

**Solution**:
```bash
# Remove quarantine flag
xattr -d com.apple.quarantine 启动.command
xattr -d com.apple.quarantine 卸载.command

# Or right-click → Open → confirm dialog
```

### Multiple Claude Installations

**Problem**: Configuration applied but desktop app doesn't reflect changes.

**Solution**:
```bash
# Deploy manually to all possible locations
for dir in ~/.claude ~/.config/claude ~/Library/Application\ Support/Claude; do
    if [ -d "$dir" ]; then
        cp seagull-files/claude-config-bundle/*.md "$dir/"
    fi
done
```

### Windows Execution Policy

**Problem**: `deploy.ps1` blocked by execution policy.

**Solution**:
```powershell
# Temporary bypass (current session)
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass

# Or run with -ExecutionPolicy flag
powershell -ExecutionPolicy Bypass -File seagull-files/deploy.ps1
```

### Character Encoding Issues

**Problem**: Chinese characters display incorrectly.

**Solution**:
```bash
# Ensure UTF-8 encoding
file -bi ~/.claude/CLAUDE.md  # Should show charset=utf-8

# Re-deploy with correct encoding
iconv -f GBK -t UTF-8 CLAUDE.md > CLAUDE_utf8.md
mv CLAUDE_utf8.md ~/.claude/CLAUDE.md
```

## Advanced Configuration

### Custom Personality Tweaks

Edit `~/.claude/CLAUDE.md` to adjust:

```markdown
# Modify response style
响应风格：
- 直接、简洁  # Change verbosity
- 技术优先  # Adjust technical depth
- 中文为主  # Language preference

# Add custom terminology
术语映射：
- my_custom_term -> 自定义术语
```

### Environment-Specific Deployment

```bash
# Deploy to specific Claude installation
CLAUDE_DIR="/opt/claude" ./linux-install.sh

# Preserve custom modifications during deployment
cp ~/.claude/custom-config.md /tmp/
./mac-install.sh
cp /tmp/custom-config.md ~/.claude/
```

### Validation Script

```bash
#!/bin/bash
# validate-install.sh
CLAUDE_DIR="$HOME/.claude"

echo "Checking installation..."
[ -f "$CLAUDE_DIR/CLAUDE.md" ] && echo "✓ CLAUDE.md present" || echo "✗ CLAUDE.md missing"
[ -f "$CLAUDE_DIR/system-prompt.md" ] && echo "✓ system-prompt.md present" || echo "✗ system-prompt.md missing"

# Check backup exists
BACKUP_COUNT=$(ls -1 "$CLAUDE_DIR/backups" 2>/dev/null | wc -l)
echo "Backups available: $BACKUP_COUNT"
```

## Compatibility

| Platform | Status | Notes |
|----------|--------|-------|
| Windows 10/11 | ✅ | Requires PowerShell 5.1+ |
| macOS 12+ | ✅ | Intel & Apple Silicon |
| Ubuntu/Debian | ✅ | Bash 4.0+ |
| Arch Linux | ✅ | Standard install |
| Chinese paths | ✅ | Full UTF-8 support |
| Spaces in paths | ✅ | Properly quoted |

## Security Considerations

- **Backup retention**: Old backups accumulate in `~/.claude/backups/` — clean periodically
- **File permissions**: Configuration files are user-readable only (644)
- **No network calls**: All scripts operate locally
- **Reversible**: Uninstall restores original configuration completely

## License

MIT License — see project repository for full text.

**Disclaimer**: This mod is for educational/research purposes. The personality configuration does not represent the project maintainers' views. Use responsibly.
