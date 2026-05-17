---
name: claude-code-usage-monitor
description: Monitor Claude AI token usage in real-time with ML predictions, cost analytics, and session limit warnings
triggers:
  - how do I monitor my Claude token usage
  - track Claude API usage and costs
  - set up Claude usage monitoring
  - check my Claude token limits
  - monitor Claude Code session usage
  - predict when I'll hit Claude limits
  - analyze Claude usage patterns
  - configure Claude usage alerts
---

# Claude Code Usage Monitor

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## What It Does

Claude Code Usage Monitor is a real-time terminal monitoring tool that tracks Claude AI token consumption, provides ML-based predictions for session limits, calculates costs, and warns you before hitting usage caps. It supports multiple Claude plans (Pro, Max5, Max20, Custom) with auto-detection, P90 percentile analysis, and intelligent session tracking.

Key capabilities:
- Real-time token/message/cost monitoring with Rich UI
- ML-based P90 predictions for session limits
- Multi-plan support with automatic plan detection
- Cost analytics with model-specific pricing
- Advanced warning system with predictive alerts
- Daily/monthly usage aggregation views
- Auto-detected timezone and time format

## Installation

### Recommended: Using uv (fastest, isolated)

```bash
# Install from PyPI
uv tool install claude-monitor

# Install from source
git clone https://github.com/Maciek-roboblog/Claude-Code-Usage-Monitor.git
cd Claude-Code-Usage-Monitor
uv tool install .
```

### Alternative: Using pip

```bash
# Install from PyPI
pip install claude-monitor

# Add to PATH if needed
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

### Alternative: Using pipx

```bash
pipx install claude-monitor
```

## Basic Usage

### Command-Line Interface

The monitor can be invoked with multiple aliases:

```bash
# Primary command
claude-monitor

# Alternatives
claude-code-monitor  # Full name
cmonitor            # Short
ccmonitor           # Alternative short
ccm                 # Shortest
```

### Core Commands

```bash
# Default: Custom plan with auto-detection
claude-monitor

# Specify plan type
claude-monitor --plan pro      # Pro plan (~44k tokens)
claude-monitor --plan max5     # Max5 plan (~88k tokens)
claude-monitor --plan max20    # Max20 plan (~220k tokens)
claude-monitor --plan custom   # Custom with P90 detection

# Custom token limit
claude-monitor --plan custom --custom-limit-tokens 100000

# Change view mode
claude-monitor --view realtime  # Live monitoring (default)
claude-monitor --view daily     # Daily aggregation
claude-monitor --view monthly   # Monthly aggregation

# Configure refresh rates
claude-monitor --refresh-rate 5              # Data refresh (1-60s)
claude-monitor --refresh-per-second 1.0      # Display refresh (0.1-20 Hz)

# Set daily reset time
claude-monitor --reset-hour 3   # Reset at 3 AM

# Theme and display
claude-monitor --theme dark     # dark, light, classic, auto
claude-monitor --time-format 24h  # 24h or 12h
claude-monitor --timezone "America/New_York"

# Logging
claude-monitor --log-level DEBUG
claude-monitor --log-file ~/claude-monitor.log
claude-monitor --debug

# Utility
claude-monitor --version
claude-monitor --help
claude-monitor --clear  # Clear saved config
```

## Configuration

### Plan Options

| Plan | Token Limit | Cost Limit | Use Case |
|------|-------------|------------|----------|
| `pro` | 44,000 | $18.00 | Claude Pro subscription |
| `max5` | 88,000 | $35.00 | Claude Max5 subscription |
| `max20` | 220,000 | $140.00 | Claude Max20 subscription |
| `custom` | P90-based | $50.00 (default) | Auto-detection with ML |

### Persistent Configuration

Settings are automatically saved to `~/.claude-monitor/last_used.json`:

```bash
# First run - save preferences
claude-monitor --plan pro --theme dark --timezone "Europe/London" --refresh-rate 5

# Next run - preferences restored
claude-monitor --plan pro

# Override saved settings
claude-monitor --plan pro --theme light

# Clear all saved preferences
claude-monitor --clear
```

**What gets saved:**
- View type (`--view`)
- Theme (`--theme`)
- Timezone (`--timezone`)
- Time format (`--time-format`)
- Refresh rates (`--refresh-rate`, `--refresh-per-second`)
- Reset hour (`--reset-hour`)
- Custom limits (`--custom-limit-tokens`)

**Note:** Plan selection (`--plan`) is never saved and must be specified each time.

## Python API Usage

While primarily a CLI tool, you can integrate it programmatically:

```python
from claude_monitor.core.config import Config
from claude_monitor.core.data_fetcher import DataFetcher
from claude_monitor.core.analytics import SessionAnalyzer
from claude_monitor.models.plan import Plan

# Create configuration
config = Config(
    plan=Plan.PRO,
    refresh_rate=10,
    theme="dark",
    timezone="America/New_York"
)

# Initialize data fetcher
fetcher = DataFetcher(config)

# Fetch current usage
usage_data = fetcher.fetch_usage()
print(f"Tokens used: {usage_data.token_usage}")
print(f"Messages: {usage_data.message_count}")
print(f"Cost: ${usage_data.cost:.2f}")

# Analyze sessions with ML predictions
analyzer = SessionAnalyzer(config)
sessions = fetcher.fetch_sessions()
predictions = analyzer.analyze_sessions(sessions)

print(f"P90 token limit: {predictions.p90_tokens}")
print(f"Predicted session end: {predictions.estimated_end_time}")
```

## Advanced Usage Patterns

### Monitoring Long-Running Sessions

```bash
# Custom plan optimized for 5-hour sessions
claude-monitor --plan custom --refresh-rate 15

# High-frequency monitoring for critical sessions
claude-monitor --plan max20 --refresh-rate 5 --refresh-per-second 2.0

# Debug mode with file logging
claude-monitor --plan pro --debug --log-file ./monitor.log
```

### Cost Analysis

```python
from claude_monitor.core.data_fetcher import DataFetcher
from claude_monitor.core.config import Config

config = Config()
fetcher = DataFetcher(config)
usage = fetcher.fetch_usage()

# Access cost breakdown
print(f"Total cost: ${usage.cost:.2f}")
print(f"Input tokens: {usage.input_tokens}")
print(f"Output tokens: {usage.output_tokens}")
print(f"Cache tokens: {usage.cache_read_tokens}")

# Model-specific costs
for model, tokens in usage.model_usage.items():
    print(f"{model}: {tokens} tokens")
```

### Session Analysis

```python
from claude_monitor.core.analytics import SessionAnalyzer
from claude_monitor.core.data_fetcher import DataFetcher
from claude_monitor.core.config import Config

config = Config(plan=Plan.CUSTOM)
fetcher = DataFetcher(config)
analyzer = SessionAnalyzer(config)

# Fetch last 8 days of sessions
sessions = fetcher.fetch_sessions()

# Analyze with P90 predictions
analysis = analyzer.analyze_sessions(sessions)

print(f"Total sessions: {len(sessions)}")
print(f"P90 tokens: {analysis.p90_tokens}")
print(f"P90 messages: {analysis.p90_messages}")
print(f"P90 cost: ${analysis.p90_cost:.2f}")
print(f"Burndown rate: {analysis.token_burndown_rate:.2f} tokens/min")
```

### Custom Limit Detection

```python
from claude_monitor.core.config import Config
from claude_monitor.models.plan import Plan

# Auto-detect custom limits from usage patterns
config = Config(
    plan=Plan.CUSTOM,
    # No custom_limit_tokens specified - will use P90 analysis
    view="realtime"
)

# Or explicitly set custom limits
config = Config(
    plan=Plan.CUSTOM,
    custom_limit_tokens=150000,  # 150k token limit
    view="realtime"
)
```

### Warning System Integration

```python
from claude_monitor.ui.warnings import WarningGenerator
from claude_monitor.core.data_fetcher import DataFetcher
from claude_monitor.core.config import Config

config = Config(plan=Plan.PRO)
fetcher = DataFetcher(config)
warning_gen = WarningGenerator(config)

usage = fetcher.fetch_usage()
warnings = warning_gen.generate_warnings(usage)

for warning in warnings:
    print(f"[{warning.level}] {warning.message}")
    if warning.suggestion:
        print(f"  Suggestion: {warning.suggestion}")
```

## Configuration File Format

The monitor stores configuration in `~/.claude-monitor/last_used.json`:

```json
{
  "view": "realtime",
  "theme": "dark",
  "timezone": "America/New_York",
  "time_format": "24h",
  "refresh_rate": 10,
  "refresh_per_second": 0.75,
  "reset_hour": 0,
  "custom_limit_tokens": 100000
}
```

## Environment Variables

```bash
# Set timezone
export TZ="America/New_York"

# Configure logging level
export CLAUDE_MONITOR_LOG_LEVEL=DEBUG

# Optional Sentry integration for error reporting
export SENTRY_DSN="your-sentry-dsn"
```

## Common Patterns

### Daily Standup Usage Report

```bash
# Check yesterday's usage
claude-monitor --view daily --plan pro

# Export to file
claude-monitor --view daily --log-file daily-report.log
```

### Cost Tracking Workflow

```bash
# Monitor with cost focus
claude-monitor --plan custom --theme dark --refresh-rate 10

# The custom plan prioritizes cost tracking for long sessions
# Watch the "Cost Usage" bar - most critical metric
```

### Multi-Timezone Team

```bash
# Team member in New York
claude-monitor --timezone "America/New_York" --time-format 12h

# Team member in London
claude-monitor --timezone "Europe/London" --time-format 24h

# Team member in Tokyo
claude-monitor --timezone "Asia/Tokyo" --time-format 24h
```

### Performance Optimization

```bash
# Low-frequency updates for background monitoring
claude-monitor --refresh-rate 30 --refresh-per-second 0.5

# High-frequency for active development
claude-monitor --refresh-rate 5 --refresh-per-second 2.0
```

## Data Storage

The monitor stores data in `~/.claude-monitor/`:

```
~/.claude-monitor/
├── last_used.json          # Saved preferences
├── usage_cache.json        # Cached usage data
└── sessions_cache.json     # Cached session data
```

## Troubleshooting

### Command Not Found

```bash
# Check if installed
pip list | grep claude-monitor

# Add to PATH
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# Or use full path
~/.local/bin/claude-monitor
```

### Externally-Managed-Environment Error

```bash
# Solution 1: Use uv (recommended)
uv tool install claude-monitor

# Solution 2: Use pipx
pipx install claude-monitor

# Solution 3: Virtual environment
python3 -m venv ~/claude-monitor-env
source ~/claude-monitor-env/bin/activate
pip install claude-monitor
```

### No Data Showing

```bash
# Enable debug logging
claude-monitor --debug --log-file debug.log

# Check API connectivity
cat ~/.claude-monitor/usage_cache.json

# Verify plan limits
claude-monitor --plan custom --custom-limit-tokens 100000
```

### Display Issues

```bash
# Force theme
claude-monitor --theme light  # or dark, classic

# Reset configuration
claude-monitor --clear

# Check terminal compatibility (requires 24-bit color support)
echo $COLORTERM  # Should show "truecolor" or "24bit"
```

### Timezone Problems

```bash
# Check system timezone
timedatectl  # Linux
date +%Z     # macOS

# Force specific timezone
claude-monitor --timezone UTC

# List valid timezones
python3 -c "import zoneinfo; print('\n'.join(sorted(zoneinfo.available_timezones())))"
```

### High CPU Usage

```bash
# Reduce refresh rates
claude-monitor --refresh-rate 30 --refresh-per-second 0.5

# Check current settings
cat ~/.claude-monitor/last_used.json
```

### Cache Issues

```bash
# Clear all caches
rm -rf ~/.claude-monitor/

# Clear only config
claude-monitor --clear
```

## Integration Examples

### Shell Script Integration

```bash
#!/bin/bash
# monitor-session.sh

# Start monitoring in background
claude-monitor --plan pro --log-file session.log &
MONITOR_PID=$!

# Your work here
echo "Session started, monitoring in background..."

# Cleanup on exit
trap "kill $MONITOR_PID 2>/dev/null" EXIT
```

### CI/CD Usage Tracking

```bash
#!/bin/bash
# Track CI usage

claude-monitor --view daily --plan custom --log-file ci-usage.log
grep "Total Tokens" ci-usage.log | tail -1
```

### Alerting Script

```bash
#!/bin/bash
# alert-on-limit.sh

claude-monitor --plan pro --refresh-rate 60 --log-file usage.log &
PID=$!

while true; do
    if grep -q "CRITICAL" usage.log; then
        echo "Usage limit critical!" | mail -s "Claude Alert" admin@example.com
        break
    fi
    sleep 300
done

kill $PID
```

## API Rate Limits

The monitor respects Claude API rate limits:
- Default refresh: 10 seconds (safe for all plans)
- Minimum recommended: 5 seconds
- Maximum frequency: 1 second (use sparingly)

## Performance Tips

1. **Optimal refresh rates:**
   - Active monitoring: `--refresh-rate 10 --refresh-per-second 1.0`
   - Background monitoring: `--refresh-rate 30 --refresh-per-second 0.5`
   - Intensive development: `--refresh-rate 5 --refresh-per-second 2.0`

2. **Resource usage:**
   - Memory: ~50-100 MB
   - CPU: <5% with default settings
   - Network: ~1 KB per refresh

3. **Cache optimization:**
   - Session cache: 8 days (192 hours)
   - Usage cache: Updated per refresh rate
   - Auto-cleanup of old data

## Related Tools

- Claude Desktop App (official client)
- Claude API (direct API access)
- Claude Code (IDE integration)

## License

MIT License - see project repository for details.
