---
name: claude-codex-usage-dashboard
description: A local Node.js dashboard for monitoring Claude Code and Codex API usage limits on spare devices over Wi-Fi.
triggers:
  - set up a usage dashboard for Claude and Codex
  - monitor my Claude Code usage limits
  - create a local dashboard to track API usage
  - show me how to install the usage monitoring dashboard
  - configure the Claude usage dashboard for my network
  - display remaining API limits on another device
  - set up the e-ink KOBO dashboard page
  - auto-start the usage dashboard on login
---

# Claude Codex Usage Dashboard

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## What It Does

`claude-codex-usage-dashboard` is a local Node.js server that displays Claude Code and Codex API usage limits on a spare phone, tablet, or e-ink device connected to the same Wi-Fi network. It reads usage data from local cache files and serves a real-time dashboard with visual alerts when limits are reached.

**Key features:**
- Displays 5-hour and weekly usage windows for Claude Code and Codex
- Shows used or remaining percentage
- Reads Claude Code usage from `~/.claude/usage-cache.json` via statusLine
- Reads Codex usage from `~/.codex/sessions` snapshots
- Works on any device on the same Wi-Fi network
- Includes a server-rendered `/kobo` page for old e-ink browsers
- Zero npm dependencies (uses only Node.js built-ins)
- Red alert when usage reaches threshold

## Installation

### Quick Start

```bash
# Clone the repository
git clone https://github.com/frankchiu-dev/claude-codex-usage-dashboard.git
cd claude-codex-usage-dashboard

# Start the server (Node.js 18+ required)
node server.js
```

Output shows two URLs:
```
Local:  http://localhost:8787
Device: http://192.168.1.23:8787
```

Open the `Device` URL on any device connected to the same Wi-Fi network.

### Platform-Specific Setup

**Windows:**
```powershell
# Start the dashboard
.\start-dashboard.bat

# Configure Claude Code statusLine integration
.\setup-claude-statusline.bat

# Install autostart on login
.\install-autostart.bat

# Remove autostart
.\uninstall-autostart.bat
```

**macOS:**
```bash
# Make scripts executable
chmod +x ./start-dashboard.sh ./setup-claude-statusline.sh

# Start the dashboard
./start-dashboard.sh

# Configure Claude Code statusLine integration
./setup-claude-statusline.sh

# Install autostart LaunchAgent
chmod +x ./install-autostart-macos.sh
./install-autostart-macos.sh

# Remove autostart
./uninstall-autostart-macos.sh
```

## Claude Code statusLine Configuration

To display real Claude Code usage, configure the statusLine feature:

### First-Time Setup

**Windows:**
```powershell
.\setup-claude-statusline.bat
```

**macOS:**
```bash
./setup-claude-statusline.sh
```

Then:
1. Fully quit Claude Code
2. Reopen Claude Code
3. Send one message (this updates the cache)
4. Refresh the dashboard

The dashboard will now read from `~/.claude/usage-cache.json`.

### Fanout Mode (Multiple statusLine Commands)

If you already have a custom statusLine command (e.g., Stream Deck integration), use fanout mode to send data to both:

```bash
# Copy example config
cp ./config.example.json ./config.json
```

Edit `config.json`:

**Windows example:**
```json
{
  "extraStatuslineCommand": "powershell -NoProfile -ExecutionPolicy Bypass -File \"%USERPROFILE%\\.claude\\your-existing-statusline.ps1\""
}
```

**macOS example:**
```json
{
  "extraStatuslineCommand": "/Users/YOUR_NAME/.claude/your-existing-statusline.sh"
}
```

Enable fanout:

**Windows:**
```powershell
.\setup-claude-statusline.bat --fanout
```

**macOS:**
```bash
./setup-claude-statusline.sh --fanout
```

## Configuration

### Environment Variables

```bash
# Change port (default: 8787)
PORT=8790 node server.js

# Bind to localhost only (default: 0.0.0.0 for network access)
HOST=127.0.0.1 node server.js

# Alert threshold percentage (default: 85)
ALERT_PERCENT=90 node server.js

# Display mode: "used" or "remaining" (default: used)
DISPLAY_MODE=remaining node server.js

# KOBO page refresh interval in seconds (default: 60, minimum: 15)
KOBO_REFRESH_SECONDS=120 node server.js

# Codex session lookback period in days (default: 14)
CODEX_LOOKBACK_DAYS=7 node server.js

# Custom cache paths
CLAUDE_USAGE_CACHE=~/.claude/usage-cache.json node server.js
CODEX_SESSIONS_DIR=~/.codex/sessions node server.js

# Fanout mode - extra statusLine command
EXTRA_STATUSLINE_COMMAND="/path/to/other-script.sh" node server.js
```

**Windows multi-variable example:**
```powershell
$env:PORT="8790"
$env:HOST="127.0.0.1"
$env:DISPLAY_MODE="remaining"
$env:ALERT_PERCENT="90"
node server.js
```

**macOS multi-variable example:**
```bash
PORT=8790 HOST=127.0.0.1 DISPLAY_MODE=remaining ALERT_PERCENT=90 node server.js
```

### config.json

Optional configuration file for fanout mode:

```json
{
  "extraStatuslineCommand": "/path/to/your-existing-statusline.sh"
}
```

## Usage Patterns

### Standard Dashboard

Access on your computer:
```
http://localhost:8787
```

Access from phone/tablet on same Wi-Fi:
```
http://192.168.1.23:8787
```

Features:
- Tap anywhere to refresh
- Tap to request fullscreen mode
- Red alert when usage ≥ ALERT_PERCENT
- Shows both 5-hour and weekly windows

### KOBO / E-ink Mode

For old browsers without modern JavaScript support:

**Shortest URLs:**
```
http://YOUR-LAN-IP:8787/k    # remaining percentage (default)
http://YOUR-LAN-IP:8787/u    # used percentage
```

**Full URLs:**
```
http://YOUR-LAN-IP:8787/kobo                  # remaining mode
http://YOUR-LAN-IP:8787/kobo?mode=used        # used mode
http://YOUR-LAN-IP:8787/kobo?mode=remaining   # remaining mode
```

**Aliases:**
- `/eink` → `/kobo`
- `/e` → `/kobo`
- `/r` → `/kobo?mode=remaining`
- `/k` → `/kobo?mode=remaining`
- `/kr` → `/kobo?mode=remaining`
- `/ku` → `/kobo?mode=used`

KOBO page is server-rendered with `<meta refresh>` tag. No JavaScript required.

### Display Modes

**Used percentage (default):**
- Shows how much of the limit has been consumed
- Turns red when ≥ ALERT_PERCENT

**Remaining percentage:**
- Shows how much quota is left
- Still turns red when used percentage ≥ ALERT_PERCENT (not when remaining ≤ 15%)

```bash
# Used mode
DISPLAY_MODE=used node server.js

# Remaining mode
DISPLAY_MODE=remaining node server.js
```

## Network Setup

### Windows Firewall Rule

Allow incoming connections on port 8787:

```powershell
netsh advfirewall firewall add rule name="AIUsageDashboard" dir=in action=allow protocol=TCP localport=8787
```

For custom port:
```powershell
netsh advfirewall firewall add rule name="AIUsageDashboard" dir=in action=allow protocol=TCP localport=8790
```

### macOS Firewall

macOS will prompt for permission when Node.js tries to accept incoming connections. Click "Allow" to enable network access.

## Code Examples

### Custom Node.js Integration

Read the same usage data in your own script:

```javascript
const fs = require('fs');
const os = require('os');
const path = require('path');

// Read Claude Code usage
function getClaudeUsage() {
  const cachePath = path.join(os.homedir(), '.claude', 'usage-cache.json');
  try {
    const data = JSON.parse(fs.readFileSync(cachePath, 'utf8'));
    return {
      fiveHour: data.fiveHour || { used: 0, limit: 100 },
      weekly: data.weekly || { used: 0, limit: 500 }
    };
  } catch (err) {
    return null;
  }
}

// Read Codex usage from latest session
function getCodexUsage() {
  const sessionsDir = path.join(os.homedir(), '.codex', 'sessions');
  try {
    const files = fs.readdirSync(sessionsDir)
      .filter(f => f.endsWith('.json'))
      .map(f => ({
        name: f,
        path: path.join(sessionsDir, f),
        mtime: fs.statSync(path.join(sessionsDir, f)).mtime
      }))
      .sort((a, b) => b.mtime - a.mtime);
    
    if (files.length === 0) return null;
    
    const data = JSON.parse(fs.readFileSync(files[0].path, 'utf8'));
    return data.rate_limits || null;
  } catch (err) {
    return null;
  }
}

// Example usage
const claude = getClaudeUsage();
const codex = getCodexUsage();

console.log('Claude 5-hour:', claude.fiveHour);
console.log('Codex:', codex);
```

### Custom statusLine Handler

Create a custom script that receives Claude Code statusLine JSON:

**Windows (PowerShell):**
```powershell
# custom-statusline.ps1
param($jsonInput)

$data = $jsonInput | ConvertFrom-Json

# Write to custom cache
$cachePath = "$env:USERPROFILE\.claude\my-custom-cache.json"
$data | ConvertTo-Json -Depth 10 | Set-Content $cachePath

# Display in terminal
Write-Host "5-hour: $($data.fiveHour.used)/$($data.fiveHour.limit)"
Write-Host "Weekly: $($data.weekly.used)/$($data.weekly.limit)"
```

**macOS (Bash):**
```bash
#!/bin/bash
# custom-statusline.sh

# Read JSON from stdin
read -r json_input

# Write to custom cache
echo "$json_input" > ~/.claude/my-custom-cache.json

# Parse and display (requires jq)
echo "5-hour: $(echo "$json_input" | jq -r '.fiveHour.used')/$(echo "$json_input" | jq -r '.fiveHour.limit')"
echo "Weekly: $(echo "$json_input" | jq -r '.weekly.used')/$(echo "$json_input" | jq -r '.weekly.limit')"
```

Configure in `config.json`:
```json
{
  "extraStatuslineCommand": "/path/to/custom-statusline.sh"
}
```

Then enable fanout mode.

## Server API Endpoints

The server exposes these endpoints:

### GET /

Main dashboard page (modern JavaScript).

### GET /kobo, /eink, /e, /k, /r

Server-rendered e-ink/KOBO page. Query params:
- `mode=used` or `mode=remaining`

### GET /u, /ku

KOBO page in "used" mode.

### GET /api/usage

JSON endpoint for programmatic access:

```bash
curl http://localhost:8787/api/usage
```

Example response:
```json
{
  "claude": {
    "fiveHour": {
      "used": 45,
      "limit": 100,
      "percentage": 45,
      "remaining": 55
    },
    "weekly": {
      "used": 230,
      "limit": 500,
      "percentage": 46,
      "remaining": 270
    }
  },
  "codex": {
    "fiveHour": {
      "used": 12,
      "limit": 50,
      "percentage": 24,
      "remaining": 38
    },
    "weekly": {
      "used": 85,
      "limit": 200,
      "percentage": 42.5,
      "remaining": 115
    }
  },
  "alertThreshold": 85,
  "displayMode": "used"
}
```

Use in scripts:
```bash
# Get Claude 5-hour usage percentage
curl -s http://localhost:8787/api/usage | jq '.claude.fiveHour.percentage'

# Check if alert threshold reached
USAGE=$(curl -s http://localhost:8787/api/usage | jq '.claude.fiveHour.percentage')
THRESHOLD=$(curl -s http://localhost:8787/api/usage | jq '.alertThreshold')

if (( $(echo "$USAGE >= $THRESHOLD" | bc -l) )); then
  echo "⚠️  Alert: Claude usage at ${USAGE}%"
fi
```

## Troubleshooting

### Dashboard shows "No data" for Claude

**Problem:** Claude card displays "No data available."

**Solutions:**
1. Run the statusLine setup script:
   ```bash
   # Windows
   .\setup-claude-statusline.bat
   
   # macOS
   ./setup-claude-statusline.sh
   ```

2. Fully quit and restart Claude Code

3. Send at least one message to trigger cache update

4. Check that `~/.claude/usage-cache.json` exists:
   ```bash
   # macOS/Linux
   ls -la ~/.claude/usage-cache.json
   
   # Windows
   dir %USERPROFILE%\.claude\usage-cache.json
   ```

### Dashboard shows "No data" for Codex

**Problem:** Codex card displays "No data available."

**Solutions:**
1. Check that `~/.codex/sessions` directory exists and contains JSON files

2. Use Codex at least once to generate session data

3. Increase lookback period:
   ```bash
   CODEX_LOOKBACK_DAYS=30 node server.js
   ```

### Device cannot connect

**Problem:** Phone/tablet cannot access dashboard.

**Solutions:**
1. Verify both devices are on the same Wi-Fi network (not guest network)

2. Check server is running with `HOST=0.0.0.0` (default):
   ```bash
   node server.js
   # Should show "Device: http://192.168.x.x:8787"
   ```

3. Add firewall rule (Windows):
   ```powershell
   netsh advfirewall firewall add rule name="AIUsageDashboard" dir=in action=allow protocol=TCP localport=8787
   ```

4. Allow Node.js incoming connections (macOS)

5. Try accessing by IP address instead of hostname

### KOBO page not refreshing

**Problem:** E-ink device shows stale data.

**Solutions:**
1. Increase refresh interval:
   ```bash
   KOBO_REFRESH_SECONDS=120 node server.js
   ```

2. Use the `/k` or `/u` short URLs (easier to type on e-ink)

3. Manually refresh the page (some KOBO browsers ignore meta refresh)

### Numbers update only after using Claude/Codex

**Expected behavior:** This dashboard reads local cache files that update only when you actively use Claude Code or Codex. It does not poll external APIs.

- **Claude Code:** Only updates when you send messages in Claude Code (not web/desktop app)
- **Codex:** Only updates when Codex writes new session files

This is by design to respect local-only data access.

### Port already in use

**Problem:** Error `EADDRINUSE` when starting server.

**Solutions:**
1. Use a different port:
   ```bash
   PORT=8790 node server.js
   ```

2. Find and kill process using port 8787:
   ```bash
   # Windows
   netstat -ano | findstr :8787
   taskkill /PID <PID> /F
   
   # macOS/Linux
   lsof -ti:8787 | xargs kill
   ```

### Fanout mode not working

**Problem:** Extra statusLine command not receiving data.

**Solutions:**
1. Verify `config.json` exists and contains correct path:
   ```json
   {
     "extraStatuslineCommand": "/full/path/to/script.sh"
   }
   ```

2. Make sure extra script is executable:
   ```bash
   chmod +x /path/to/script.sh
   ```

3. Test extra script manually:
   ```bash
   echo '{"fiveHour":{"used":10,"limit":100}}' | /path/to/script.sh
   ```

4. Check environment variable:
   ```bash
   echo $EXTRA_STATUSLINE_COMMAND
   ```

### Autostart not working (macOS)

**Problem:** LaunchAgent doesn't start dashboard on login.

**Solutions:**
1. Check LaunchAgent status:
   ```bash
   launchctl list | grep claude-codex-usage-dashboard
   ```

2. View logs:
   ```bash
   cat ~/Library/Logs/claude-codex-usage-dashboard.log
   cat ~/Library/Logs/claude-codex-usage-dashboard.err.log
   ```

3. Manually load the agent:
   ```bash
   launchctl load ~/Library/LaunchAgents/com.frankchiu.claude-codex-usage-dashboard.plist
   ```

4. Verify working directory in plist is correct:
   ```bash
   cat ~/Library/LaunchAgents/com.frankchiu.claude-codex-usage-dashboard.plist
   ```

## Best Practices

1. **Keep Node.js updated:** Requires Node.js 18+
2. **Use environment variables:** Don't hardcode ports or paths
3. **Firewall rules:** Add explicit allow rule for your port
4. **E-ink devices:** Use `/k` or `/u` short URLs for easier typing
5. **Fanout mode:** Test extra statusLine script independently before enabling fanout
6. **Privacy:** Never commit cache files or config.json to version control

## License

MIT License - see repository for full text.
