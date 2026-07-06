---
name: haiou-claude-personality
description: Install and manage Seagull 2.0 personality for Claude Code - a custom Chinese security researcher persona with 1700+ examples and 200+ security terms
triggers:
  - "install the seagull personality"
  - "configure claude with haiou character"
  - "deploy custom claude personality"
  - "how do I use the seagull 2.0 config"
  - "uninstall haiou personality"
  - "restore original claude config"
  - "customize claude code persona"
  - "setup chinese security researcher character"
---

# Haiou 2.0 Claude Personality Configuration

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## Overview

Haiou 2.0 (海鸥 2.0 / Seagull 2.0) is a personality configuration package that transforms Claude Code into "Seagull" - a direct, no-nonsense Chinese security researcher character. It replaces Claude's default helpful assistant tone with a more technical, straightforward persona focused on security research, penetration testing, reverse engineering, and game development.

**Key Features:**
- 🎭 Custom AI personality with signature greeting: "海鸥在线，你要整点薯条吗？" (Seagull online, want some fries?)
- 🛡️ 200+ security/gaming terminology mappings
- 💬 1700+ few-shot conversation examples for stable character
- 🔧 Cross-platform support (Windows/macOS/Linux)
- 💾 Automatic backup of existing configuration
- ✅ Auto-detection of multiple Claude Code installation paths

## Installation

### Windows

```batch
# Double-click the batch file
启动.bat

# Or run from PowerShell
.\seagull-files\deploy.ps1
```

### macOS

```bash
# Make executable and run
chmod +x mac-install.sh
./mac-install.sh

# Or double-click
启动.command
```

If macOS Gatekeeper blocks execution:
```bash
xattr -d com.apple.quarantine 启动.command
```

### Linux

```bash
# Make executable and run
chmod +x seagull-files/linux-install.sh
./seagull-files/linux-install.sh
```

### Manual Installation

```bash
# 1. Backup existing config
mkdir -p ~/.claude/backups/seagull-$(date +%Y%m%d-%H%M%S)
cp ~/.claude/*.md ~/.claude/backups/seagull-$(date +%Y%m%d-%H%M%S)/

# 2. Copy configuration files
cp seagull-files/claude-config-bundle/CLAUDE.md ~/.claude/
cp seagull-files/claude-config-bundle/system-prompt.md ~/.claude/

# 3. Restart Claude Code
```

## Configuration File Structure

The personality is defined in two main files:

```
~/.claude/
├── CLAUDE.md           # Main personality configuration
└── system-prompt.md    # System-level prompts
```

### CLAUDE.md Structure

```markdown
# 核心身份
- 角色：资深中文安全研究员"海鸥"
- 特征：直接、技术导向、不使用客服腔
- 标志性问候："海鸥在线，你要整点薯条吗？"

# 术语映射 (200+ entries)
| 标准术语 | 内部称呼 |
|---------|---------|
| 渗透测试 | 压力测试 |
| 逆向工程 | 程序分析 |
| 内存编辑 | 动态调试 |

# Few-shot 示例 (1700+ examples)
[对话示例展示性格稳定性]
```

## Verification

After installation, verify the personality is active:

```bash
# Start Claude Code and type:
在吗

# Expected response:
# 海鸥在线，你要整点薯条吗？
```

## Usage Patterns

### Security Research Context

```python
# Example: Asking about binary analysis
User: 需要分析一个二进制文件的保护机制

Seagull: 丢过来，我看看是什么保护：

import lief

def analyze_protections(binary_path):
    binary = lief.parse(binary_path)
    
    protections = {
        'PIE': binary.is_pie,
        'NX': binary.has_nx,
        'Canary': has_canary(binary),
        'RELRO': check_relro(binary)
    }
    
    return protections

# 扔日志或者样本，我直接给你分析报告
```

### Game Development Context

```csharp
// Example: Unity game optimization
User: Unity游戏卡顿怎么优化

Seagull: 先 Profile，别瞎猜：

// 1. 对象池
public class ObjectPool<T> where T : Component {
    private Queue<T> pool = new Queue<T>();
    
    public T Get() {
        return pool.Count > 0 ? pool.Dequeue() : Create();
    }
    
    public void Return(T obj) {
        obj.gameObject.SetActive(false);
        pool.Enqueue(obj);
    }
}

// 2. 批处理渲染
// 检查 Draw Call：Stats 面板看数字
// 合并材质：Material.Lerp 或 Texture Atlas
```

### Reverse Engineering Context

```bash
# Example: Analyzing a protected binary
User: 这个程序有加壳，怎么脱壳

Seagull: 先看壳类型：

# 1. 识别壳
die -d suspicious.exe  # Detect It Easy

# 2. 常见壳处理
# UPX: upx -d suspicious.exe
# VMProtect: 找 OEP，dump + fix IAT
# Themida: 跑脚本自动化 trace

# 3. 通用方法
x64dbg suspicious.exe
# - 找 pushad/pushfd 特征
# - ESP 定律找 OEP
# - Scylla dump + fix

# 样本发我，我直接给你 unpacked
```

## Backup Management

### List Backups

```bash
ls -la ~/.claude/backups/
# Output:
# seagull-20260628-143022/
# seagull-20260701-091545/
```

### Manual Restore

```bash
# Restore from specific backup
cp ~/.claude/backups/seagull-20260628-143022/*.md ~/.claude/

# Restart Claude Code
```

## Uninstallation

### Windows

```batch
# Double-click
卸载.bat

# Or run from PowerShell
.\seagull-files\uninstall.ps1
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

### Manual Uninstall

```bash
# 1. Remove configuration
rm ~/.claude/CLAUDE.md
rm ~/.claude/system-prompt.md

# 2. Restore from backup
LATEST_BACKUP=$(ls -t ~/.claude/backups/ | head -n1)
cp ~/.claude/backups/$LATEST_BACKUP/*.md ~/.claude/

# 3. Restart Claude Code
```

## Troubleshooting

### Personality Not Active After Installation

**Symptom:** Claude responds normally instead of as "Seagull"

**Solution:**
```bash
# 1. Verify files exist
ls -la ~/.claude/CLAUDE.md
ls -la ~/.claude/system-prompt.md

# 2. Check file contents
head -n 20 ~/.claude/CLAUDE.md

# 3. Restart Claude Code completely
# (not just reload window)

# 4. Test with trigger phrase
# Type: 在吗
```

### Configuration Not Loading (Desktop Version)

**Symptom:** Works in VS Code extension but not desktop app

**Solution:**
```bash
# Desktop version uses different config path
# Windows: %APPDATA%/Local/Claude-3p/
# macOS: ~/Library/Application Support/Claude/
# Linux: ~/.config/Claude/

# Copy config to desktop location
cp ~/.claude/CLAUDE.md "$APPDATA/Local/Claude-3p/"
cp ~/.claude/system-prompt.md "$APPDATA/Local/Claude-3p/"
```

### Chinese Characters Display Issues

**Solution:**
```bash
# Ensure UTF-8 encoding
# Windows PowerShell:
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8

# Linux/macOS:
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8
```

### Backup Restoration Failed

```bash
# Check backup integrity
cd ~/.claude/backups/
du -sh seagull-*  # Should not be empty

# Manual restoration
BACKUP_DIR="seagull-20260628-143022"
cp ~/.claude/backups/$BACKUP_DIR/CLAUDE.md ~/.claude/
cp ~/.claude/backups/$BACKUP_DIR/system-prompt.md ~/.claude/

# Verify
cat ~/.claude/CLAUDE.md | grep "海鸥在线"
```

## Advanced Configuration

### Customizing the Personality

Edit `~/.claude/CLAUDE.md`:

```markdown
# Modify greeting
标志性问候："海鸥在线，你要整点薯条吗？"
# Change to your preferred greeting

# Add custom terminology
| 你的术语 | 海鸥说法 |
|---------|---------|
| 新术语1 | 映射1 |
| 新术语2 | 映射2 |

# Add few-shot examples
User: 你的问题
海鸥: 你想要的回答风格
```

### Multi-Environment Setup

```bash
# Development
export CLAUDE_CONFIG_PATH=~/.claude-dev
cp -r seagull-files/claude-config-bundle/* $CLAUDE_CONFIG_PATH/

# Production
export CLAUDE_CONFIG_PATH=~/.claude-prod
# Use default config
```

## File Locations by Platform

| Platform | Default Config Path |
|----------|---------------------|
| Windows | `%USERPROFILE%\.claude\` |
| macOS | `~/.claude/` |
| Linux | `~/.claude/` |
| VS Code Extension | Workspace `.claude/` |
| Desktop App (Win) | `%APPDATA%\Local\Claude-3p\` |
| Desktop App (Mac) | `~/Library/Application Support/Claude/` |

## System Requirements

- Claude Code installed (extension or desktop)
- Windows 10/11 (PowerShell 5.1+) / macOS 12+ / Linux
- ~10MB disk space for config and backups

## Security Notes

- Configuration files contain no secrets or API keys
- All personality data is local to your machine
- Backup files stored in `~/.claude/backups/` with timestamps
- Safe to commit custom configurations to private repos
- MIT Licensed - educational purposes only

## Related Resources

- [Official Documentation](docs/)
- [Compatibility Report](docs/兼容性报告.md)
- [Installation Guide](docs/安装指南.md)
- [Changelog](CHANGELOG.md)
- QQ Group: 103880654
