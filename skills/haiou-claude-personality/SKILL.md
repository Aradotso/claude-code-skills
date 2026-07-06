---
name: haiou-claude-personality
description: Deploy and manage the Haiou 2.0 personality pack for Claude Code - a custom AI persona that transforms Claude into a direct, no-nonsense Chinese security researcher.
triggers:
  - "install haiou personality"
  - "deploy seagull claude config"
  - "customize claude persona"
  - "remove claude personality pack"
  - "海鸥配置"
  - "setup custom claude character"
  - "restore original claude config"
  - "haiou deployment troubleshooting"
---

# Haiou Claude Personality Skill

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

The Haiou 2.0 (Seagull 2.0) project is a custom personality configuration pack for Claude Code that transforms the AI assistant into "海鸥" (Seagull) - a blunt, experienced Chinese security researcher persona. It includes 1700+ few-shot examples, 200+ security/gaming terminology mappings, and cross-platform deployment scripts with automatic backup and rollback capabilities.

## What It Does

- Replaces Claude's default personality with a custom "海鸥" persona (greets with "海鸥在线，你要整点薯条吗？")
- Injects custom system prompts and conversation examples to lock in the personality
- Maps 200+ technical terms for security research, reverse engineering, and game development contexts
- Provides platform-specific installation scripts (Windows/macOS/Linux)
- Automatically backs up original configuration before deployment
- Supports one-click rollback to restore default Claude behavior

## Installation

### Windows

```batch
REM Run from project root
启动.bat
```

Or manually via PowerShell:

```powershell
powershell -ExecutionPolicy Bypass -File seagull-files/deploy.ps1
```

### macOS

```bash
# Make executable (first time only)
chmod +x mac-install.sh

# Run installer
./mac-install.sh

# Or use the double-click launcher
open 启动.command
```

If macOS Gatekeeper blocks execution:

```bash
# Remove quarantine attribute
xattr -d com.apple.quarantine 启动.command

# Or right-click → Open → confirm
```

### Linux

```bash
# Make executable
chmod +x seagull-files/linux-install.sh

# Run installer
./seagull-files/linux-install.sh
```

## Verification

After installation, restart Claude Code and test:

**User input:** `在吗`

**Expected response:** `海鸥在线，你要整点薯条吗？`

If you see the signature greeting, the personality is active.

## Configuration Files

The deployment modifies these files in `~/.claude/`:

```
~/.claude/
├── CLAUDE.md              # Main personality definition
├── system-prompt.md       # System-level prompt injection
└── backups/
    └── seagull-YYYYMMDD-HHMMSS/  # Timestamped backup
```

### Custom Configuration Locations

The installer auto-detects multiple Claude installations:

**Windows:**
- `%USERPROFILE%\.claude\`
- `%APPDATA%\Local\Claude-3p\`
- `%APPDATA%\Roaming\Claude\`

**macOS:**
- `~/.claude/`
- `~/Library/Application Support/Claude/`

**Linux:**
- `~/.claude/`
- `~/.config/claude/`

## Core Files Structure

```
seagull-files/
├── claude-config-bundle/
│   ├── CLAUDE.md          # Personality definition with few-shot examples
│   └── system-prompt.md   # System prompt with terminology mappings
├── deploy.ps1             # Windows PowerShell installer
├── linux-install.sh       # Linux bash installer
└── linux-uninstall.sh     # Linux uninstaller
```

## Deployment Script Breakdown (PowerShell)

```powershell
# Check PowerShell version
if ($PSVersionTable.PSVersion.Major -lt 5) {
    Write-Host "需要 PowerShell 5.1 或更高版本" -ForegroundColor Red
    exit 1
}

# Define config directory
$claudeDir = "$env:USERPROFILE\.claude"

# Backup existing config
$backupDir = "$claudeDir\backups\seagull-$(Get-Date -Format 'yyyyMMdd-HHmmss')"
New-Item -ItemType Directory -Force -Path $backupDir | Out-Null

if (Test-Path "$claudeDir\CLAUDE.md") {
    Copy-Item "$claudeDir\CLAUDE.md" "$backupDir\"
}

# Deploy new config
$bundleDir = ".\seagull-files\claude-config-bundle"
Copy-Item "$bundleDir\CLAUDE.md" "$claudeDir\" -Force
Copy-Item "$bundleDir\system-prompt.md" "$claudeDir\" -Force

Write-Host "✅ 海鸥部署完成" -ForegroundColor Green
```

## Uninstallation

### Windows

```batch
卸载.bat
```

### macOS

```bash
./mac-uninstall.sh
# Or double-click 卸载.command
```

### Linux

```bash
./seagull-files/linux-uninstall.sh
```

The uninstaller:
1. Finds the most recent backup in `~/.claude/backups/`
2. Restores original `CLAUDE.md` and `system-prompt.md`
3. Removes Haiou configuration files
4. Leaves backups intact for manual recovery

## Manual Rollback

```bash
# List backups
ls -la ~/.claude/backups/

# Restore specific backup
cp ~/.claude/backups/seagull-20260628-143022/CLAUDE.md ~/.claude/
cp ~/.claude/backups/seagull-20260628-143022/system-prompt.md ~/.claude/

# Restart Claude Code
```

## Shell Script Pattern (Linux/macOS)

```bash
#!/bin/bash

CLAUDE_DIR="$HOME/.claude"
BUNDLE_DIR="./seagull-files/claude-config-bundle"

# Create backup
BACKUP_DIR="$CLAUDE_DIR/backups/seagull-$(date +%Y%m%d-%H%M%S)"
mkdir -p "$BACKUP_DIR"

if [ -f "$CLAUDE_DIR/CLAUDE.md" ]; then
    cp "$CLAUDE_DIR/CLAUDE.md" "$BACKUP_DIR/"
fi

# Deploy
cp "$BUNDLE_DIR/CLAUDE.md" "$CLAUDE_DIR/"
cp "$BUNDLE_DIR/system-prompt.md" "$CLAUDE_DIR/"

echo "✅ 海鸥部署完成，请重启 Claude Code"
```

## Troubleshooting

### No Personality Change After Installation

**Symptom:** Claude still responds with default assistant tone

**Solution:**
```bash
# 1. Verify files exist
ls -la ~/.claude/CLAUDE.md ~/.claude/system-prompt.md

# 2. Check file content
head -20 ~/.claude/CLAUDE.md

# 3. Restart Claude Code completely (quit + relaunch)
# 4. Clear conversation history and start new chat
```

### macOS Gatekeeper Blocking Scripts

**Symptom:** "cannot be opened because it is from an unidentified developer"

**Solution:**
```bash
# Remove quarantine flag
xattr -d com.apple.quarantine 启动.command
xattr -d com.apple.quarantine mac-install.sh

# Or: System Preferences → Security & Privacy → Allow
```

### PowerShell Execution Policy Error

**Symptom:** "execution of scripts is disabled on this system"

**Solution:**
```powershell
# Temporary bypass (current session only)
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass

# Then run
.\启动.bat

# Or use the built-in bypass flag
powershell -ExecutionPolicy Bypass -File seagull-files\deploy.ps1
```

### Multiple Claude Installations

**Symptom:** Desktop Claude vs. Code extension using different configs

**Solution:**
```bash
# Find all possible config locations
find ~ -name ".claude" -type d 2>/dev/null

# Deploy to all locations manually
for dir in ~/.claude ~/Library/Application\ Support/Claude; do
    cp seagull-files/claude-config-bundle/* "$dir/"
done
```

### Chinese Path Issues

**Symptom:** Installation fails with encoding errors

**Solution:**
```powershell
# Windows: Ensure UTF-8 encoding
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8

# macOS/Linux: Check locale
locale  # Should show UTF-8
export LANG=zh_CN.UTF-8
```

## Advanced: Custom Terminology Mapping

Edit `seagull-files/claude-config-bundle/system-prompt.md`:

```markdown
## 术语映射

| 行话 | 实际含义 |
|------|---------|
| 薯条 | 代码/任务 |
| 调试薯条 | 逆向工程 |
| 炸薯条机 | 编译器 |
| 薯条配方 | 算法 |
| 您的自定义术语 | 对应技术含义 |
```

After editing, re-run the installer to apply changes.

## Environment Variables

The scripts don't require environment variables, but you can override paths:

```bash
# Custom Claude config directory
export CLAUDE_CONFIG_DIR="$HOME/custom-claude-config"
./seagull-files/linux-install.sh  # Modify script to use $CLAUDE_CONFIG_DIR
```

## Common Use Cases

### Quick Deploy on New Machine

```bash
# Clone and deploy in one line
git clone https://github.com/haiou-666/haiou2.0-Claude-Code-.git && \
cd haiou2.0-Claude-Code- && \
chmod +x mac-install.sh && \
./mac-install.sh
```

### Temporarily Disable Personality

```bash
# Rename config files (keeps backups intact)
mv ~/.claude/CLAUDE.md ~/.claude/CLAUDE.md.disabled
mv ~/.claude/system-prompt.md ~/.claude/system-prompt.md.disabled

# Restart Claude - default personality returns
# Re-enable: mv ~/.claude/CLAUDE.md.disabled ~/.claude/CLAUDE.md
```

### Version Control Your Custom Edits

```bash
# Fork the repo, edit configs
cd seagull-files/claude-config-bundle/
vim CLAUDE.md  # Add your custom few-shot examples

# Commit and track changes
git add CLAUDE.md
git commit -m "Add custom security terminology"
git push origin main
```

This skill enables AI agents to guide users through deploying, configuring, troubleshooting, and managing the Haiou personality pack across all supported platforms.
