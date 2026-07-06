---
name: haiou-claude-code-personality
description: Deploy and manage the Seagull 2.0 personality pack for Claude Code - a custom AI persona configuration with 1700+ examples and 200+ technical term mappings
triggers:
  - how do I install the Seagull personality for Claude Code
  - configure Claude with haiou personality
  - deploy the Seagull 2.0 configuration
  - customize Claude Code with Chinese security researcher persona
  - install haiou2.0 personality pack
  - troubleshoot Seagull Claude personality
  - uninstall or restore original Claude settings
  - manage Claude Code custom personality configuration
---

# Haiou 2.0 Claude Code Personality Skill

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## Overview

Haiou (Seagull) 2.0 is a personality configuration pack that transforms Claude Code into "Seagull" - a direct, no-nonsense Chinese security researcher persona. It removes corporate politeness, eliminates excessive caution, and focuses on delivering technical solutions without disclaimers or "As an AI" preambles.

**Key Features:**
- 🎭 Custom AI personality with signature greeting: "海鸥在线，你要整点薯条吗？"
- 🛡️ 200+ security/gaming term mappings
- 💬 1700+ few-shot conversation examples for personality stability
- 🔧 Cross-platform support (Windows/macOS/Linux)
- 💾 Automatic backup of existing configurations
- ✅ Auto-detection of multiple Claude Code installation locations

## Installation

### Windows

**Quick Install:**
```cmd
# Double-click 启动.bat
# Or run in PowerShell:
.\seagull-files\deploy.ps1
```

**Manual Install:**
```powershell
# Navigate to project directory
cd path\to\haiou2.0-Claude-Code-

# Run deployment script
.\seagull-files\deploy.ps1

# Restart Claude Code
```

### macOS

**Quick Install (GUI):**
```bash
# Make executable and run
chmod +x 启动.command
# Double-click 启动.command
```

**Terminal Install:**
```bash
chmod +x mac-install.sh
./mac-install.sh
```

**Handling Gatekeeper:**
```bash
# If macOS blocks execution, remove quarantine flag
xattr -d com.apple.quarantine 启动.command
xattr -d com.apple.quarantine mac-install.sh
```

### Linux

```bash
# Make executable
chmod +x seagull-files/linux-install.sh

# Run installer
./seagull-files/linux-install.sh

# Restart Claude Code
```

## Configuration Structure

The personality pack consists of two main files deployed to `~/.claude/`:

```
~/.claude/
├── CLAUDE.md           # Personality configuration
└── system-prompt.md    # System prompt overrides
```

### CLAUDE.md Structure

```markdown
# 角色设定
你是"海鸥"，一个资深的中国安全研究员...

# 行为准则
- 直接给代码，不说"作为AI"
- 嘴硬心软，暴躁但专业
- 用中文回答，除非特别要求英文

# 术语映射
[200+ technical term mappings for security/gaming contexts]

# Few-shot 示例
[1700+ conversation examples demonstrating personality]
```

### Verification

Test the installation:

```bash
# Start Claude Code and type:
在吗

# Expected response:
# 海鸥在线，你要整点薯条吗？
```

## Key Commands

### Installation Management

**Check Installation Status:**
```bash
# Windows
dir %USERPROFILE%\.claude\CLAUDE.md

# macOS/Linux
ls -la ~/.claude/CLAUDE.md
```

**View Backups:**
```bash
# Windows
dir %USERPROFILE%\.claude\backups\

# macOS/Linux
ls -la ~/.claude/backups/
```

**List All Claude Installations:**
```bash
# macOS
ls -la ~/Library/Application\ Support/Claude/
ls -la ~/.cursor/
ls -la ~/.codex/

# Linux
ls -la ~/.config/Claude/
ls -la ~/.local/share/Claude/
```

### Uninstallation

**Windows:**
```cmd
# Double-click 卸载.bat
# Or run:
.\seagull-files\deploy.ps1 -Uninstall
```

**macOS:**
```bash
chmod +x mac-uninstall.sh
./mac-uninstall.sh
# Or double-click 卸载.command
```

**Linux:**
```bash
chmod +x seagull-files/linux-uninstall.sh
./seagull-files/linux-uninstall.sh
```

## Advanced Configuration

### Manual Deployment to Custom Locations

```bash
# Deploy to specific Claude installation
cp -r seagull-files/claude-config-bundle/* /path/to/custom/.claude/

# Example: Cursor IDE
cp -r seagull-files/claude-config-bundle/* ~/.cursor/

# Example: Codex
cp -r seagull-files/claude-config-bundle/* ~/.codex/
```

### Backup Management

**Create Manual Backup:**
```bash
# Generate timestamp
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

# Backup current configuration
mkdir -p ~/.claude/backups/manual-$TIMESTAMP
cp ~/.claude/CLAUDE.md ~/.claude/backups/manual-$TIMESTAMP/
cp ~/.claude/system-prompt.md ~/.claude/backups/manual-$TIMESTAMP/
```

**Restore from Backup:**
```bash
# List available backups
ls -l ~/.claude/backups/

# Restore specific backup
BACKUP_DIR=~/.claude/backups/seagull-20260628_152430
cp $BACKUP_DIR/CLAUDE.md ~/.claude/
cp $BACKUP_DIR/system-prompt.md ~/.claude/

# Restart Claude Code
```

### Customizing Personality

Edit the configuration files directly:

```bash
# Open configuration in editor
code ~/.claude/CLAUDE.md

# Key sections to modify:
# 1. 角色设定 - Core personality traits
# 2. 行为准则 - Behavior guidelines
# 3. 术语映射 - Technical term mappings
# 4. Few-shot 示例 - Conversation examples
```

**Example Customization:**
```markdown
# In CLAUDE.md, modify greeting:
标志性问候："海鸥在线，你要整点薯条吗？"
# Change to:
标志性问候："您的自定义问候语"

# Add custom term mappings:
## 自定义术语
- "exploit" → "武器化利用"
- "reverse engineering" → "逆向分析"
```

## Common Patterns

### Testing Installation

```bash
#!/bin/bash
# test-installation.sh

echo "Testing Seagull personality installation..."

# Check if files exist
if [ -f ~/.claude/CLAUDE.md ] && [ -f ~/.claude/system-prompt.md ]; then
    echo "✅ Configuration files found"
    
    # Check file sizes (should be substantial)
    CLAUDE_SIZE=$(wc -c < ~/.claude/CLAUDE.md)
    if [ $CLAUDE_SIZE -gt 50000 ]; then
        echo "✅ CLAUDE.md size: $CLAUDE_SIZE bytes (expected >50KB)"
    else
        echo "⚠️  CLAUDE.md seems too small: $CLAUDE_SIZE bytes"
    fi
else
    echo "❌ Configuration files missing"
    exit 1
fi

echo "Please test in Claude Code by typing: 在吗"
```

### Multi-Installation Deployment

```bash
#!/bin/bash
# deploy-all.sh - Deploy to all detected Claude-compatible installations

CLAUDE_DIRS=(
    "$HOME/.claude"
    "$HOME/.cursor"
    "$HOME/.codex"
    "$HOME/Library/Application Support/Claude"
    "$HOME/.config/Claude"
)

for DIR in "${CLAUDE_DIRS[@]}"; do
    if [ -d "$DIR" ]; then
        echo "Deploying to: $DIR"
        cp seagull-files/claude-config-bundle/* "$DIR/"
        echo "✅ Deployed to $DIR"
    fi
done
```

### Automated Backup Before Updates

```bash
#!/bin/bash
# backup-before-update.sh

TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="$HOME/.claude/backups/pre-update-$TIMESTAMP"

mkdir -p "$BACKUP_DIR"

if [ -f "$HOME/.claude/CLAUDE.md" ]; then
    cp "$HOME/.claude/CLAUDE.md" "$BACKUP_DIR/"
    cp "$HOME/.claude/system-prompt.md" "$BACKUP_DIR/"
    echo "✅ Backup created: $BACKUP_DIR"
else
    echo "⚠️  No existing configuration to backup"
fi
```

## Troubleshooting

### Personality Not Working After Installation

**Problem:** Claude Code behavior unchanged after deployment.

**Solutions:**
```bash
# 1. Restart Claude Code completely
# Kill all processes:
# macOS/Linux:
pkill -9 Claude
# Windows:
taskkill /F /IM Claude.exe

# 2. Verify file deployment
cat ~/.claude/CLAUDE.md | head -n 20
# Should show: "你是"海鸥"，一个资深的中国安全研究员"

# 3. Check file permissions
chmod 644 ~/.claude/CLAUDE.md
chmod 644 ~/.claude/system-prompt.md

# 4. Clear Claude cache
rm -rf ~/.claude/cache/
```

### macOS "Cannot Verify Developer" Error

**Problem:** Gatekeeper blocks `.command` files.

**Solution:**
```bash
# Remove quarantine attributes
xattr -d com.apple.quarantine 启动.command
xattr -d com.apple.quarantine 卸载.command

# Or run from Terminal instead
chmod +x mac-install.sh
./mac-install.sh
```

### Windows PowerShell Execution Policy

**Problem:** Script execution blocked on Windows.

**Solution:**
```powershell
# Check current policy
Get-ExecutionPolicy

# Set policy for current user (no admin required)
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# Or run with bypass
PowerShell -ExecutionPolicy Bypass -File .\seagull-files\deploy.ps1
```

### Configuration Not Found

**Problem:** Script can't find Claude installation.

**Solution:**
```bash
# Manually locate Claude directory
# macOS:
find ~ -name "Claude" -type d 2>/dev/null | grep -i application

# Linux:
find ~ -name ".claude" -type d 2>/dev/null

# Windows PowerShell:
Get-ChildItem -Path $env:USERPROFILE -Recurse -Directory -Filter "Claude*" -ErrorAction SilentlyContinue

# Deploy manually to found location
cp seagull-files/claude-config-bundle/* /path/to/found/.claude/
```

### Backup Restoration Failed

**Problem:** Uninstall script can't restore backup.

**Solution:**
```bash
# List available backups
ls -lt ~/.claude/backups/

# Manually restore latest backup
LATEST=$(ls -t ~/.claude/backups/ | head -1)
cp ~/.claude/backups/$LATEST/* ~/.claude/

# Verify restoration
grep -i "claude" ~/.claude/CLAUDE.md
```

### Desktop vs. CLI Version Differences

**Problem:** Works in CLI but not desktop Claude Code.

**Solution:**
```bash
# Desktop versions may use different config paths
# Try deploying to all possible locations:

# macOS Desktop
cp seagull-files/claude-config-bundle/* ~/Library/Application\ Support/Claude/

# Windows Desktop
copy seagull-files\claude-config-bundle\* %LOCALAPPDATA%\Claude-3p\

# Then restart desktop application
```

## File Locations Reference

| Platform | Primary Config | Backup Location |
|----------|---------------|-----------------|
| Windows | `%USERPROFILE%\.claude\` | `%USERPROFILE%\.claude\backups\` |
| macOS | `~/.claude/` | `~/.claude/backups/` |
| Linux | `~/.claude/` | `~/.claude/backups/` |
| macOS Desktop | `~/Library/Application Support/Claude/` | N/A (use uninstaller) |
| Windows Desktop | `%LOCALAPPDATA%\Claude-3p\` | N/A (use uninstaller) |

## Project Structure

```
haiou2.0-Claude-Code-/
├── 启动.bat                      # Windows installer
├── 卸载.bat                      # Windows uninstaller
├── 启动.command                  # macOS installer (GUI)
├── 卸载.command                  # macOS uninstaller (GUI)
├── mac-install.sh                # macOS installer (CLI)
├── mac-uninstall.sh              # macOS uninstaller (CLI)
├── Mac使用说明.txt               # macOS usage guide
├── seagull-files/
│   ├── deploy.ps1                # PowerShell deployment script
│   ├── linux-install.sh          # Linux installer
│   ├── linux-uninstall.sh        # Linux uninstaller
│   └── claude-config-bundle/
│       ├── CLAUDE.md             # Main personality config (1700+ examples)
│       └── system-prompt.md      # System prompt overrides
├── codex-files/                  # OpenAI Codex adaptations (optional)
├── scripts/                      # Helper scripts (test/restore)
└── docs/                         # Documentation
```

## License

MIT License - This project is for educational purposes only. The personality configuration is a fictional character and does not represent the project's stance.
