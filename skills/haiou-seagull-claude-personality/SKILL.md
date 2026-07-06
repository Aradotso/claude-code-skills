---
name: haiou-seagull-claude-personality
description: Install and manage the "Seagull 2.0" custom personality pack for Claude Code — a direct, no-nonsense Chinese security researcher persona with 1700+ examples and 200+ security/gaming term mappings
triggers:
  - install seagull personality for claude
  - customize claude code personality
  - deploy haiou character configuration
  - remove claude personality customization
  - troubleshoot seagull persona not working
  - configure claude with security researcher persona
  - backup and restore claude configuration
  - install 海鸥 personality pack
---

# Haiou Seagull Claude Personality Skill

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## Overview

Haiou 2.0 (Seagull 2.0) is a personality customization package for Claude Code that transforms the AI assistant into "海鸥" (Seagull) — a direct, assertive Chinese security researcher character. The package includes:

- **1700+ few-shot conversation examples** to lock in personality traits
- **200+ security/gaming term mappings** for penetration testing, reverse engineering, and game development
- **Cross-platform installation scripts** for Windows, macOS, and Linux
- **Automatic backup system** for safe configuration management
- **One-command uninstall** to restore original settings

The signature greeting: "海鸥在线，你要整点薯条吗？" (Seagull online, want some fries?)

## Installation

### Windows

```batch
# Double-click the batch file
启动.bat

# Or run in PowerShell
powershell -ExecutionPolicy Bypass -File seagull-files/deploy.ps1
```

### macOS

```bash
# Make executable and run
chmod +x mac-install.sh
./mac-install.sh

# Or double-click (with Gatekeeper handling)
# Double-click: 启动.command
# If blocked: Right-click → Open → Confirm

# Remove quarantine flag if needed
xattr -d com.apple.quarantine 启动.command
```

### Linux

```bash
# Make executable and run
chmod +x seagull-files/linux-install.sh
./seagull-files/linux-install.sh
```

## Configuration Structure

The personality pack modifies two key files in `~/.claude/`:

### CLAUDE.md
Contains the core personality definition with:
- Character traits and behavioral patterns
- Response style guidelines
- 200+ domain-specific term mappings
- Technical terminology for security/gaming contexts

### system-prompt.md
Contains:
- 1700+ few-shot conversation examples
- Dialog patterns for personality consistency
- Anti-jailbreak anchors to maintain character

## Configuration File Locations

The installer automatically detects and deploys to multiple possible locations:

```bash
# macOS/Linux
~/.claude/
~/Library/Application Support/Claude/
~/.config/claude/

# Windows
%USERPROFILE%\.claude\
%LOCALAPPDATA%\Claude-3p\
%APPDATA%\Claude\
```

## Verification

After installation, restart Claude Code and test:

```
User: 在吗
Expected Response: 海鸥在线，你要整点薯条吗？
```

## Backup System

### Automatic Backups

Installation automatically creates timestamped backups:

```bash
# Backup location
~/.claude/backups/seagull-YYYYMMDD-HHMMSS/
  ├── CLAUDE.md
  └── system-prompt.md
```

### Manual Backup

```bash
# Before installation
cp -r ~/.claude ~/.claude-backup-$(date +%Y%m%d)

# Restore from backup
cp -r ~/.claude/backups/seagull-20260705-123456/* ~/.claude/
```

## Uninstallation

### Windows

```batch
# Double-click
卸载.bat
```

### macOS

```bash
# Terminal
./mac-uninstall.sh

# Or double-click
# 卸载.command
```

### Linux

```bash
./seagull-files/linux-uninstall.sh
```

Uninstallation automatically restores the most recent backup from `~/.claude/backups/`.

## Troubleshooting

### Personality Not Activating

**Symptom**: Claude Code responds normally, ignoring Seagull personality

**Solutions**:

1. **Restart Claude Code completely** (not just reload window)
   ```bash
   # macOS/Linux: Kill process
   pkill -f "Claude Code"
   
   # Windows: Task Manager → End "Claude Code"
   ```

2. **Verify file placement**:
   ```bash
   # Check files exist
   ls -la ~/.claude/CLAUDE.md
   ls -la ~/.claude/system-prompt.md
   
   # Verify content (should show Chinese characters)
   head -n 5 ~/.claude/CLAUDE.md
   ```

3. **Check for multiple installations**:
   ```bash
   # Find all Claude config locations
   find ~ -name "CLAUDE.md" -type f 2>/dev/null
   ```

### macOS "Cannot Be Verified" Error

**Symptom**: macOS blocks `.command` file execution

**Solution**:

```bash
# Method 1: Remove quarantine attribute
xattr -d com.apple.quarantine 启动.command
xattr -d com.apple.quarantine 卸载.command

# Method 2: Right-click → Open (first time only)
# Then click "Open" in security dialog

# Method 3: Use .sh scripts directly
chmod +x mac-install.sh
./mac-install.sh
```

### Desktop Version Not Affected

**Symptom**: Desktop Claude app doesn't show personality changes

**Solution**: Desktop version uses different config paths. Re-run installer — it auto-detects:

```bash
# Installer checks these paths automatically:
# ~/.claude/
# ~/Library/Application Support/Claude/
# %APPDATA%/Claude/

# Manually verify all locations:
find ~ -type d -name ".claude" -o -name "Claude" 2>/dev/null
```

### PowerShell Execution Policy Error (Windows)

**Symptom**: `deploy.ps1 cannot be loaded because running scripts is disabled`

**Solution**:

```powershell
# Temporary bypass (recommended)
powershell -ExecutionPolicy Bypass -File seagull-files/deploy.ps1

# Or change policy (admin required)
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### Character Encoding Issues

**Symptom**: Chinese characters appear as garbled text

**Solution**:

```bash
# Verify file encoding (should be UTF-8)
file -I ~/.claude/CLAUDE.md

# Re-install ensuring UTF-8 encoding
./mac-install.sh  # macOS/Linux
启动.bat           # Windows
```

## Advanced Usage

### Custom Term Mappings

Edit `~/.claude/CLAUDE.md` to add your own term mappings:

```markdown
## 术语映射表
- 你的术语 → 技术实现名称
- example_term → actual_technical_term
```

### Personality Fine-Tuning

Adjust response patterns in `~/.claude/system-prompt.md`:

```markdown
User: [scenario]
Assistant: [desired response pattern]
```

Add examples following the existing 1700+ format.

### Multi-Profile Setup

```bash
# Create profile variants
cp -r ~/.claude ~/.claude-seagull
cp -r ~/.claude-backup ~/.claude-professional

# Switch profiles
rm -rf ~/.claude
cp -r ~/.claude-seagull ~/.claude  # Activate Seagull
# Restart Claude Code
```

## Project Structure

```
haiou2.0-Claude-Code-/
├── 启动.bat                      # Windows installer
├── 卸载.bat                      # Windows uninstaller
├── 启动.command                  # macOS installer (double-click)
├── 卸载.command                  # macOS uninstaller (double-click)
├── mac-install.sh                # macOS installer (terminal)
├── mac-uninstall.sh              # macOS uninstaller (terminal)
├── Mac使用说明.txt               # macOS usage guide
├── seagull-files/
│   ├── deploy.ps1                # Windows PowerShell deployment
│   ├── linux-install.sh          # Linux installer
│   ├── linux-uninstall.sh        # Linux uninstaller
│   └── claude-config-bundle/
│       ├── CLAUDE.md             # Personality config (200+ terms)
│       └── system-prompt.md      # Few-shot examples (1700+)
├── codex-files/                  # OpenAI Codex adapter (optional)
├── scripts/                      # Utility scripts
└── docs/                         # Documentation
```

## Environment Requirements

- **Claude Code** installed and configured
- **Windows**: PowerShell 5.1+ (Windows 10/11)
- **macOS**: Bash/Zsh (macOS 12+)
- **Linux**: Bash (Ubuntu/Debian/Arch tested)

## Safety Notes

- All installers create automatic backups before modification
- Uninstall process restores original configuration
- No external network calls during installation
- No API keys or credentials required
- MIT License — educational use only

## Common Use Cases

### Security Research Context

After installation, Claude Code will understand security terminology:

```
User: 帮我看看这个二进制文件的入口点
Assistant: [Responds with reverse engineering analysis using appropriate terminology]
```

### Game Development Context

```
User: 这个游戏的内存结构怎么分析？
Assistant: [Provides analysis using game hacking terminology from mappings]
```

### Direct Technical Communication

```
User: 给我代码，别废话
Assistant: [Provides code directly without preamble or disclaimers]
```
