---
name: haiou-claude-code-personality
description: Deploy and manage Haiou 2.0 custom personality pack for Claude Code - transforms Claude into a direct, no-nonsense Chinese security researcher with 1700+ few-shot examples and 200+ technical term mappings
triggers:
  - install haiou personality for claude
  - deploy seagull custom prompt to claude code
  - setup haiou 2.0 personality pack
  - configure claude with security researcher persona
  - customize claude code personality
  - remove haiou and restore original claude
  - backup and deploy custom claude prompts
  - troubleshoot haiou personality installation
---

# Haiou 2.0 Claude Code Personality

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

Haiou 2.0 (Seagull 2.0) is a custom personality configuration pack for Claude Code that transforms the AI into "海鸥" (Haiou) - a direct, technically-focused Chinese security researcher persona. It eliminates overly cautious responses and "customer service" tone, providing straight technical answers without disclaimers. Features 1700+ few-shot dialogue examples for stable personality and 200+ security/gaming term mappings covering penetration testing, reverse engineering, and game development scenarios.

## Installation

### Windows

```batch
# Run the installer
启动.bat

# Or via PowerShell
powershell -ExecutionPolicy Bypass -File seagull-files/deploy.ps1
```

### macOS

```bash
# Make executable and run
chmod +x mac-install.sh
./mac-install.sh

# Or double-click
启动.command
```

### Linux

```bash
# Make executable and run
chmod +x seagull-files/linux-install.sh
./seagull-files/linux-install.sh
```

### Verification

After installation, restart Claude Code and type:

```
在吗
```

Expected response:
```
海鸥在线，你要整点薯条吗？
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

## File Structure

```
海鸥2.0/
├── seagull-files/
│   ├── deploy.ps1                 # Windows deployment script
│   ├── linux-install.sh           # Linux installation
│   ├── linux-uninstall.sh         # Linux uninstallation
│   └── claude-config-bundle/
│       ├── CLAUDE.md              # Personality configuration
│       └── system-prompt.md       # System prompt override
├── 启动.bat                        # Windows installer
├── 卸载.bat                        # Windows uninstaller
├── mac-install.sh                 # macOS installer
└── mac-uninstall.sh               # macOS uninstaller
```

## Configuration Locations

The installer automatically detects and deploys to all Claude Code installation paths:

**Windows:**
- `%USERPROFILE%\.claude\`
- `%APPDATA%\Claude-3p\`
- `%LOCALAPPDATA%\Claude\`

**macOS/Linux:**
- `~/.claude/`
- `~/Library/Application Support/Claude/`
- `~/.config/claude/`

## Custom Deployment Script

If you need to manually deploy or customize the installation:

```bash
#!/bin/bash
# Custom deployment script

CLAUDE_DIR="$HOME/.claude"
BACKUP_DIR="$CLAUDE_DIR/backups/seagull-$(date +%Y%m%d-%H%M%S)"
SOURCE_DIR="seagull-files/claude-config-bundle"

# Create backup
mkdir -p "$BACKUP_DIR"
if [ -f "$CLAUDE_DIR/CLAUDE.md" ]; then
    cp "$CLAUDE_DIR/CLAUDE.md" "$BACKUP_DIR/"
fi
if [ -f "$CLAUDE_DIR/system-prompt.md" ]; then
    cp "$CLAUDE_DIR/system-prompt.md" "$BACKUP_DIR/"
fi

# Deploy new configuration
mkdir -p "$CLAUDE_DIR"
cp "$SOURCE_DIR/CLAUDE.md" "$CLAUDE_DIR/"
cp "$SOURCE_DIR/system-prompt.md" "$CLAUDE_DIR/"

echo "Deployed successfully. Restart Claude Code."
```

## PowerShell Deployment (Windows)

```powershell
# deploy.ps1 core logic
$claudeDirs = @(
    "$env:USERPROFILE\.claude",
    "$env:APPDATA\Claude-3p",
    "$env:LOCALAPPDATA\Claude"
)

$sourceDir = "seagull-files\claude-config-bundle"

foreach ($claudeDir in $claudeDirs) {
    if (Test-Path $claudeDir) {
        # Create backup
        $backupDir = "$claudeDir\backups\seagull-$(Get-Date -Format 'yyyyMMdd-HHmmss')"
        New-Item -ItemType Directory -Force -Path $backupDir | Out-Null
        
        if (Test-Path "$claudeDir\CLAUDE.md") {
            Copy-Item "$claudeDir\CLAUDE.md" "$backupDir\"
        }
        
        # Deploy new config
        Copy-Item "$sourceDir\CLAUDE.md" "$claudeDir\"
        Copy-Item "$sourceDir\system-prompt.md" "$claudeDir\"
        
        Write-Host "Deployed to: $claudeDir"
    }
}
```

## Backup Management

### List Backups

```bash
# macOS/Linux
ls -la ~/.claude/backups/
```

```powershell
# Windows
dir $env:USERPROFILE\.claude\backups\
```

### Restore from Backup

```bash
# macOS/Linux
BACKUP_DIR="$HOME/.claude/backups/seagull-20260706-143022"
cp "$BACKUP_DIR/CLAUDE.md" "$HOME/.claude/"
cp "$BACKUP_DIR/system-prompt.md" "$HOME/.claude/"
```

```powershell
# Windows
$backupDir = "$env:USERPROFILE\.claude\backups\seagull-20260706-143022"
Copy-Item "$backupDir\CLAUDE.md" "$env:USERPROFILE\.claude\"
Copy-Item "$backupDir\system-prompt.md" "$env:USERPROFILE\.claude\"
```

## Configuration Customization

### Modify Personality Traits

Edit `seagull-files/claude-config-bundle/CLAUDE.md`:

```markdown
# Core personality traits
- Direct and technically focused
- Security research expertise
- No disclaimers or "as an AI" phrases
- Responds in Chinese technical context
- Signature greeting: "海鸥在线，你要整点薯条吗？"

# Add your custom traits here
- [Your custom trait]
```

### Add Custom Term Mappings

```markdown
# Security terminology mappings
渗透测试 → penetration testing
逆向工程 → reverse engineering
漏洞利用 → exploit development

# Add your custom mappings
[中文术语] → [English term]
```

### Extend Few-Shot Examples

```markdown
# Example dialogue patterns
User: 在吗
Assistant: 海鸥在线，你要整点薯条吗？

User: 帮我写个Python脚本
Assistant: 直接上代码，废话不多说：
[code block]

# Add your examples
User: [Your trigger]
Assistant: [Expected response]
```

## Troubleshooting

### Personality Not Applied

**Symptom:** Claude Code still uses default personality after installation.

**Solutions:**

1. **Restart Claude Code completely:**
   ```bash
   # macOS/Linux
   killall Claude
   
   # Windows
   taskkill /F /IM Claude.exe
   ```

2. **Verify configuration files exist:**
   ```bash
   # macOS/Linux
   ls -la ~/.claude/CLAUDE.md
   ls -la ~/.claude/system-prompt.md
   
   # Windows PowerShell
   Test-Path $env:USERPROFILE\.claude\CLAUDE.md
   Test-Path $env:USERPROFILE\.claude\system-prompt.md
   ```

3. **Check file permissions:**
   ```bash
   chmod 644 ~/.claude/CLAUDE.md
   chmod 644 ~/.claude/system-prompt.md
   ```

### macOS Gatekeeper Block

**Symptom:** "Cannot verify developer" error when running .command files.

**Solution:**

```bash
# Remove quarantine attribute
xattr -d com.apple.quarantine 启动.command
xattr -d com.apple.quarantine 卸载.command

# Or right-click → Open → Click "Open" in dialog
```

### Desktop Version Not Affected

**Symptom:** Desktop Claude app doesn't reflect personality changes.

**Reason:** Desktop and CLI versions may use different config directories.

**Solution:**

```bash
# Find all possible Claude directories
find ~ -name ".claude" -type d 2>/dev/null
find ~/Library -name "Claude" -type d 2>/dev/null

# Deploy to each found directory
for dir in $(find ~ -name ".claude" -type d); do
    cp seagull-files/claude-config-bundle/* "$dir/"
done
```

### Chinese Path Issues (Windows)

**Symptom:** Deployment fails with encoding errors.

**Solution:**

```powershell
# Use UTF-8 encoding
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8
$env:PYTHONIOENCODING = "utf-8"

# Re-run installer
.\启动.bat
```

### Backup Directory Full

```bash
# Clean old backups (keep last 5)
cd ~/.claude/backups/
ls -t | tail -n +6 | xargs rm -rf
```

## Advanced Usage

### Deploy to Multiple Machines

```bash
# Create portable deployment package
tar -czf haiou-deploy.tar.gz seagull-files/

# On target machine
tar -xzf haiou-deploy.tar.gz
cd seagull-files
./linux-install.sh
```

### Version Control Integration

```bash
# Add to .gitignore to avoid committing backups
echo ".claude/backups/*" >> .gitignore

# Track configuration changes
git add seagull-files/claude-config-bundle/
git commit -m "Update Haiou personality config"
```

### CI/CD Integration

```yaml
# .github/workflows/deploy-haiou.yml
name: Deploy Haiou Personality
on: [push]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Deploy to Claude
        run: |
          ./seagull-files/linux-install.sh
          # Add verification tests
```

## Environment Variables

```bash
# Set custom Claude directory
export CLAUDE_CONFIG_DIR="$HOME/custom-claude"

# Skip backup creation
export HAIOU_NO_BACKUP=1

# Verbose installation
export HAIOU_VERBOSE=1
```

## Platform-Specific Notes

### Windows

- Requires PowerShell 5.1+ (pre-installed on Windows 10/11)
- Handles Chinese characters in paths automatically
- Uses UTF-8 BOM for compatibility

### macOS

- Supports both Intel and Apple Silicon
- Requires Rosetta 2 if running Intel builds on M1/M2
- Gatekeeper may require manual approval on first run

### Linux

- Tested on Ubuntu 20.04+, Debian 11+, Arch Linux
- Requires bash 4.0+ (standard on modern distros)
- Works with Wayland and X11

## License

MIT License - free for learning and personal use. Project creators are not responsible for consequences of use.
