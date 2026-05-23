---
name: feishu-claude-code-bridge
description: Bridge Feishu/Lark messenger with local Claude Code CLI for streaming conversations, per-chat sessions, and workspace management
triggers:
  - set up feishu bridge with claude code
  - connect lark messenger to claude cli
  - configure feishu bot for claude code
  - fix feishu claude bridge not responding
  - manage claude code workspaces in feishu
  - troubleshoot lark channel bridge
  - stream claude responses to feishu
  - set up access control for feishu bot
---

# Feishu/Lark Claude Code Bridge

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## What It Does

`lark-channel-bridge` connects Feishu (Lark) messenger with your local Claude Code CLI. Messages sent to the bot are forwarded to `claude` running in a working directory you control, with responses streaming back to Lark in real-time cards. Features:

- **Streaming cards**: Claude's output updates live in a single Lark card
- **Per-chat sessions**: each chat/topic maintains its own Claude session
- **Multiple workspaces**: switch between project directories with `/ws`
- **Images & files**: send media directly; Claude reads downloaded paths
- **Preemption**: new messages interrupt running requests
- **Access control**: optional allowlists for users, chats, and admin commands

## Installation

### Prerequisites

- Node.js >= 20
- `claude` CLI installed and authenticated: https://docs.anthropic.com/en/docs/claude-code/quickstart
- Feishu/Lark **PersonalAgent** app (created via QR wizard on first run)

### Install Globally

```bash
npm install -g lark-channel-bridge
# or
pnpm add -g lark-channel-bridge
```

**Important**: Always install globally before using daemon/service commands. `npx` paths are ephemeral and break background services.

## First-Time Setup

```bash
lark-channel-bridge run
```

On first run without config:
1. Terminal displays a QR code
2. Scan with Feishu/Lark app
3. Select or create a PersonalAgent app
4. Credentials saved to `~/.lark-channel/config.json` (mode 600)

## CLI Commands

### Process Management

```bash
# Foreground (interactive)
lark-channel-bridge run [-c <config>]

# List all running bridges
lark-channel-bridge ps

# Kill a specific bridge
lark-channel-bridge kill <id|#>

# Help
lark-channel-bridge --help
```

### Daemon/Service Management

Background OS-managed service with auto-restart:

```bash
# Install and start daemon
lark-channel-bridge start

# Stop daemon
lark-channel-bridge stop

# Restart in place
lark-channel-bridge restart

# Check status (pid, logs, exit code)
lark-channel-bridge status

# Remove service definition
lark-channel-bridge unregister
```

**Platform mapping**:
- **macOS**: launchd user agent at `~/Library/LaunchAgents/ai.lark-channel-bridge.bot.plist`
- **Linux**: systemd user unit at `~/.config/systemd/user/lark-channel-bridge.bot.service` (run `loginctl enable-linger $USER` for persistence across logout)
- **Windows**: Task Scheduler task `LarkChannelBridge.Bot` with launcher at `~/.lark-channel/daemon-launcher.cmd`

Logs: `~/.lark-channel/logs/daemon-stdout.log` and `daemon-stderr.log`

### Migration (for pre-0.1.11 users)

```bash
lark-channel-bridge migrate
```

Moves data from old paths (`~/.config/lark-channel-bridge/`, `~/.cache/lark-channel-bridge/`) to `~/.lark-channel/`.

## In-Chat Commands

Send these inside Feishu/Lark chats:

| Command | Description |
|---------|-------------|
| `/new`, `/reset` | Clear current chat's Claude session |
| `/cd <path>` | Switch working directory (resets session) |
| `/ws list` | List named workspaces (interactive card) |
| `/ws save <name>` | Save current directory as workspace |
| `/ws use <name>` | Switch to named workspace |
| `/ws remove <name>` | Delete workspace |
| `/status` | Show current directory/session/agent |
| `/config` | Adjust preferences (reply style, access control, timeouts) |
| `/stop` | Terminate running Claude process |
| `/timeout [N\|off\|default]` | Set idle watchdog (minutes) for session |
| `/ps` | List all running bridge processes |
| `/exit <id\|#>` | Stop a bridge process |
| `/reconnect` | Force WebSocket reconnect |
| `/doctor [description]` | Self-diagnose using recent logs |
| `/help` | Show help card |
| Any other `/xxx` | Forwarded to Claude verbatim |

**Reply behavior**:
- **DM**: bot replies to all messages
- **Group**: bot only replies when `@`-mentioned (default since 0.1.22)
- Change via `/config` → "Require @bot in groups"

## Configuration

### Config File Structure

`~/.lark-channel/config.json`:

```json
{
  "appId": "cli_xxxxxxxxxxxxxxxx",
  "appSecret": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "preferences": {
    "access": {
      "allowedUsers": [],
      "allowedChats": [],
      "admins": []
    },
    "requireAtInGroups": true,
    "defaultTimeoutMinutes": 0,
    "replyStyle": "streaming",
    "showToolCalls": true
  }
}
```

### Access Control

Configure via `/config` command in Feishu or edit JSON directly.

**Scenario: Personal use only**

```json
{
  "preferences": {
    "access": {
      "allowedUsers": ["ou_your_open_id_here"],
      "allowedChats": [],
      "admins": []
    }
  }
}
```

**Scenario: Team with restricted admins**

```json
{
  "preferences": {
    "access": {
      "allowedUsers": ["ou_user1", "ou_user2", "ou_user3"],
      "allowedChats": [],
      "admins": ["ou_admin_open_id"]
    }
  }
}
```

**Scenario: Specific groups only**

```json
{
  "preferences": {
    "access": {
      "allowedUsers": [],
      "allowedChats": ["oc_group1_id", "oc_group2_id"],
      "admins": ["ou_admin_open_id"]
    }
  }
}
```

**Finding IDs**: Have users message the bot, then:

```bash
grep '"event":"enter"' ~/.lark-channel/logs/$(date +%Y-%m-%d).log | tail -5
```

Each line contains `chatId` and `senderId` (user's `open_id`).

**Notes**:
- Empty field = unrestricted
- DMs exempt from chat allowlist
- Changes apply on next message (no restart)
- Admin commands restricted: `/account`, `/config`, `/exit`, `/reconnect`, `/doctor`, `/cd`, `/ws`

## Data Directories

| Path | Content |
|------|---------|
| `~/.lark-channel/config.json` | App credentials (mode 600) |
| `~/.lark-channel/sessions.json` | Session IDs, working directories per chat |
| `~/.lark-channel/workspaces.json` | Named workspace map |
| `~/.lark-channel/processes.json` | Process registry for `ps`/`stop` |
| `~/.lark-channel/media/<chatId>/` | Downloaded files (24h cleanup) |
| `~/.lark-channel/logs/YYYY-MM-DD.log` | Structured logs (7-day retention, configurable via `LARK_CHANNEL_LOG_DAYS`) |

## Common Patterns

### Running as Background Service

```bash
# Install globally first
npm install -g lark-channel-bridge

# Start daemon with auto-restart
lark-channel-bridge start

# Check status
lark-channel-bridge status

# View logs
tail -f ~/.lark-channel/logs/$(date +%Y-%m-%d).log
```

### Managing Multiple Workspaces

**In Feishu chat**:

```
/cd /path/to/project-a
/ws save project-a

/cd /path/to/project-b
/ws save project-b

/ws list
# Click button or:
/ws use project-a
```

Session history is preserved per workspace.

### Handling Timeouts

**Global default** (applies to new sessions):

```
/config
# Set "Default timeout (minutes)" to 10
```

**Per-session override**:

```
/timeout 5      # 5 minutes for this chat
/timeout off    # disable for this chat
/timeout default # revert to global default
```

Watchdog kills Claude if no output for N minutes; card annotated with `⏱ N min no response, auto-terminated`.

### Sending Files to Claude

1. Upload image/file directly in Feishu chat
2. Bot downloads to `~/.lark-channel/media/<chatId>/`
3. File path passed to Claude CLI
4. Files auto-deleted after 24h

### Diagnosing Issues

**Bot not responding**:

```
/status
# Check if session/directory valid

/doctor The bot stopped replying after I changed directories
# Claude analyzes recent logs and suggests fixes

/reconnect
# Force WebSocket reconnect if network interrupted
```

**Check logs**:

```bash
tail -f ~/.lark-channel/logs/$(date +%Y-%m-%d).log | jq
```

**List running processes**:

```bash
lark-channel-bridge ps
# or in Feishu:
/ps
```

## Troubleshooting

### Bot Silent / No Responses

**Cause**: `claude` CLI not authenticated or session points to nonexistent directory.

**Fix**:
```
/status  # inspect current state
/new     # reset session
/cd /valid/path  # switch to valid directory
```

Verify Claude CLI works:
```bash
cd /your/project
claude "test message"
```

### Frozen Card / No Updates

**Cause**: Claude subprocess hung; idle watchdog may have triggered.

**Fix**:
```
/stop  # terminate current run
/timeout 10  # enable 10-minute watchdog
```

Adjust global default via `/config`.

### Image Not Recognized

**Cause**: Old version (pre-0.1.0) had filename dedup bug.

**Fix**:
```bash
npm update -g lark-channel-bridge
# or
pnpm update -g lark-channel-bridge
```

### Daemon Not Starting After Reboot (Linux)

**Cause**: User services disabled after logout.

**Fix**:
```bash
loginctl enable-linger $USER
lark-channel-bridge restart
```

### Multiple Instances Fighting for Events

**Cause**: Lark routes events randomly to multiple WebSocket connections for same app.

**Fix**:
```bash
lark-channel-bridge ps
lark-channel-bridge kill <id>  # kill duplicates
# or in Feishu:
/ps
/exit <id>
```

`run` command prompts if existing process detected.

### Logs Too Large

**Default**: 7-day retention, daily rotation.

**Override**:
```bash
export LARK_CHANNEL_LOG_DAYS=3
lark-channel-bridge restart
```

Or clean manually:
```bash
find ~/.lark-channel/logs/ -name "*.log" -mtime +3 -delete
```

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `LARK_CHANNEL_LOG_DAYS` | `7` | Log retention days; older logs pruned at startup |

## Integration Examples

### Systemd Service (Linux)

Generated automatically by `start`, but can be customized at:

```bash
~/.config/systemd/user/lark-channel-bridge.bot.service
```

Example customization:

```ini
[Service]
Environment="LARK_CHANNEL_LOG_DAYS=3"
Environment="NODE_ENV=production"
Restart=always
RestartSec=10
```

After editing:
```bash
systemctl --user daemon-reload
systemctl --user restart lark-channel-bridge.bot
```

### macOS launchd (auto-generated)

```bash
~/Library/LaunchAgents/ai.lark-channel-bridge.bot.plist
```

Customize environment variables:

```xml
<key>EnvironmentVariables</key>
<dict>
  <key>LARK_CHANNEL_LOG_DAYS</key>
  <string>3</string>
</dict>
```

Reload:
```bash
launchctl unload ~/Library/LaunchAgents/ai.lark-channel-bridge.bot.plist
launchctl load ~/Library/LaunchAgents/ai.lark-channel-bridge.bot.plist
```

### Docker Deployment (community pattern)

Not officially supported but community users run:

```dockerfile
FROM node:20-slim
RUN npm install -g lark-channel-bridge
VOLUME /root/.lark-channel
CMD ["lark-channel-bridge", "run"]
```

Mount config as volume:
```bash
docker run -d \
  -v ~/.lark-channel:/root/.lark-channel \
  --name feishu-bridge \
  your-image
```

## Best Practices

1. **Always install globally** before using `start`/daemon commands
2. **Use named workspaces** for frequently-switched projects
3. **Enable timeouts** if working with large codebases (Claude may hang on complex operations)
4. **Lock down access** in shared/team environments via `/config`
5. **Monitor logs** during initial setup: `tail -f ~/.lark-channel/logs/$(date +%Y-%m-%d).log`
6. **Run as daemon** for persistent bot availability: `lark-channel-bridge start`
7. **Check `/status`** first when troubleshooting response issues
8. **Use `/doctor`** for self-diagnosis with automatic log analysis

## License

MIT
