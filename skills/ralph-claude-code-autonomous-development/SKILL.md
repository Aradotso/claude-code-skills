---
name: ralph-claude-code-autonomous-development
description: Autonomous AI development loop for Claude Code with intelligent exit detection, session management, and rate limiting
triggers:
  - set up ralph autonomous development loop
  - create ralph project for continuous ai development
  - enable ralph in this project
  - configure ralph autonomous development
  - import requirements into ralph
  - run ralph with monitoring
  - troubleshoot ralph infinite loops
  - resume ralph session
---

# Ralph Claude Code - Autonomous Development Loop

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

Ralph is an autonomous AI development loop for Claude Code that enables continuous iterative development cycles. It implements Geoffrey Huntley's "Ralph Wiggum" technique with intelligent exit detection, session continuity, rate limiting, and circuit breakers to prevent infinite loops.

## What Ralph Does

Ralph automates the development workflow by:
- Running Claude Code in continuous autonomous loops until project completion
- Managing session continuity across iterations with context preservation
- Enforcing dual-condition exit gates (completion indicators + explicit EXIT_SIGNAL)
- Rate limiting API calls (100/hour default) with automatic resets
- Detecting stuck loops with semantic analysis and circuit breakers
- Providing live monitoring dashboards via tmux integration
- Importing tasks from PRDs, GitHub Issues, or beads task manager
- Supporting JSON output format with automatic fallback to text parsing

## Installation

### Phase 1: Install Ralph Globally (One-Time)

```bash
# Clone and install Ralph system-wide
git clone https://github.com/frankbria/ralph-claude-code.git
cd ralph-claude-code
./install.sh

# Verify installation
which ralph
ralph --version
```

This adds these global commands:
- `ralph` - Main autonomous loop
- `ralph-monitor` - Live dashboard
- `ralph-setup` - Create new Ralph projects
- `ralph-enable` - Enable Ralph in existing projects
- `ralph-import` - Import PRD/specifications
- `ralph-migrate` - Upgrade to latest folder structure

### Phase 2: Initialize Projects (Per-Project)

Choose one approach:

**Option A: Enable in Existing Project (Recommended)**
```bash
cd my-existing-project
ralph-enable  # Interactive wizard
```

**Option B: Import Existing PRD**
```bash
ralph-import requirements.md my-project
cd my-project
```

**Option C: Create New Project**
```bash
ralph-setup my-awesome-project
cd my-awesome-project
```

## Key Commands

### Core Commands

```bash
# Start autonomous development with integrated monitoring
ralph --monitor

# Start without monitoring (requires separate terminal for ralph-monitor)
ralph

# Resume existing session (preserves context)
ralph --resume <session_id>

# Live streaming output (real-time visibility)
ralph --live

# Configure output format
ralph --output-format json
ralph --output-format text

# Set allowed tools
ralph --allowed-tools "Edit,Bash(npm *),Bash(pytest)"

# Disable auto-continuation (require manual approval)
ralph --no-continue

# Custom timeout (1-120 minutes, default: 30)
ralph --timeout 60

# Verbose progress mode
ralph --verbose

# Reset session manually
ralph --reset-session
```

### Setup & Configuration Commands

```bash
# Interactive wizard for existing projects
ralph-enable
ralph-enable --from beads
ralph-enable --from github --label "sprint-1"
ralph-enable --from prd ./docs/requirements.md

# Non-interactive version (CI/automation)
ralph-enable-ci --from prd requirements.md --output-format json

# Import PRD into new project
ralph-import specifications.md my-project
ralph-import --print specs.md  # Non-interactive mode

# Migrate old projects to .ralph/ structure
cd old-project
ralph-migrate
```

### Monitoring Commands

```bash
# Live monitor dashboard (separate terminal)
ralph-monitor

# Monitor with custom refresh rate (seconds)
ralph-monitor --refresh 2

# Monitor forwards all parameters
ralph-monitor --output-format json --timeout 60
```

## Project Structure

Ralph organizes projects in a `.ralph/` subfolder:

```
my-project/
├── .ralph/
│   ├── PROMPT.md              # Main instructions for Claude
│   ├── fix_plan.md            # Prioritized task list
│   ├── .ralphrc               # Project configuration
│   ├── specs/                 # Technical specifications
│   │   └── requirements.md
│   ├── logs/                  # Execution logs
│   │   ├── ralph.log
│   │   └── claude_responses.log
│   └── sessions/              # Session management
│       └── current_session.json
├── src/                       # Your actual code
└── README.md
```

## Configuration

### `.ralphrc` File

Created automatically by `ralph-enable` or `ralph-setup`:

```bash
# .ralph/.ralphrc
ALLOWED_TOOLS="Edit,Bash(npm *),Bash(pytest)"
OUTPUT_FORMAT="json"
TIMEOUT_MINUTES=30
RATE_LIMIT_PER_HOUR=100
SESSION_TIMEOUT_HOURS=24

# Circuit breaker thresholds (optional)
CIRCUIT_BREAKER_THRESHOLD=3
CIRCUIT_BREAKER_RESET_SECONDS=300
```

### Environment Variables

Override `.ralphrc` settings via environment:

```bash
# Rate limiting
export RATE_LIMIT_PER_HOUR=150
export RATE_LIMIT_WINDOW_HOURS=1

# Session management
export SESSION_TIMEOUT_HOURS=48
export SESSION_DIR=".ralph/sessions"

# Circuit breaker
export CIRCUIT_BREAKER_THRESHOLD=5
export CIRCUIT_BREAKER_RESET_SECONDS=600

# Logging
export LOG_DIR=".ralph/logs"
export LOG_RETENTION_DAYS=7

# API configuration
export CLAUDE_API_KEY="${ANTHROPIC_API_KEY}"
```

## Writing Effective PROMPT.md

The `.ralph/PROMPT.md` file guides Claude's behavior:

```markdown
# Project: My Awesome App

## Objective
Build a REST API for task management with authentication.

## Current Status
- Database schema defined
- User authentication working
- [ ] Implement CRUD endpoints
- [ ] Add rate limiting
- [ ] Write integration tests

## Technical Stack
- Language: Python 3.11
- Framework: FastAPI
- Database: PostgreSQL
- ORM: SQLAlchemy

## Development Guidelines
1. Follow PEP 8 style guide
2. Write tests before implementation (TDD)
3. Use type hints for all functions
4. Document API endpoints with OpenAPI

## Exit Criteria
Project complete when:
- All CRUD endpoints implemented and tested
- Rate limiting configured (100 req/min)
- Integration test coverage >90%
- API documentation generated

EXIT_SIGNAL: false
```

**Important**: Claude must explicitly set `EXIT_SIGNAL: true` for Ralph to exit.

## Writing Effective fix_plan.md

Prioritized task list with checkboxes:

```markdown
# Fix Plan

## High Priority
- [ ] Implement GET /tasks endpoint
- [ ] Implement POST /tasks endpoint
- [ ] Add JWT token validation middleware

## Medium Priority
- [ ] Implement PUT /tasks/:id endpoint
- [ ] Implement DELETE /tasks/:id endpoint
- [ ] Add rate limiting with Redis

## Low Priority
- [ ] Write integration tests for all endpoints
- [ ] Generate OpenAPI documentation
- [ ] Add health check endpoint

## Completed
- [x] Set up FastAPI project structure
- [x] Configure PostgreSQL connection
- [x] Implement user registration endpoint
```

## Common Patterns

### Pattern 1: Start New Project with PRD

```bash
# You have a requirements document
ralph-import product-requirements.md my-api

cd my-api

# Review generated files
cat .ralph/PROMPT.md
cat .ralph/fix_plan.md

# Adjust task priorities if needed
nano .ralph/fix_plan.md

# Start autonomous development
ralph --monitor
```

### Pattern 2: Enable Ralph in Existing Project

```bash
cd existing-typescript-project

# Interactive wizard detects TypeScript + Next.js
ralph-enable

# Select task source: beads, GitHub Issues, or PRD
# Wizard creates .ralph/ structure with detected config

# Start development
ralph --monitor --output-format json
```

### Pattern 3: Import Tasks from Beads

```bash
cd my-project

# Import current bead as tasks
ralph-enable --from beads

# Or import specific bead
bd select my-feature-bead
ralph-enable --from beads

# Start with custom timeout
ralph --monitor --timeout 60
```

### Pattern 4: Import GitHub Issues

```bash
cd my-repo

# Import issues with label
ralph-enable --from github --label "sprint-3"

# Or all open issues
ralph-enable --from github

# Start with live streaming
ralph --live --monitor
```

### Pattern 5: Resume Interrupted Session

```bash
# Ralph was interrupted mid-development
cd my-project

# Check session logs
cat .ralph/logs/ralph.log | tail -20

# Resume with same session context
ralph --resume $(cat .ralph/sessions/current_session.json | jq -r '.session_id')

# Or let Ralph auto-resume if session not expired
ralph --monitor
```

### Pattern 6: Custom Tool Permissions

```bash
cd my-project

# Edit .ralphrc to allow specific tools
cat > .ralph/.ralphrc << 'EOF'
ALLOWED_TOOLS="Edit,Bash(npm test),Bash(npm run build),Bash(git *)"
OUTPUT_FORMAT="json"
TIMEOUT_MINUTES=45
EOF

# Start with custom config
ralph --monitor
```

### Pattern 7: CI/CD Integration

```bash
# .github/workflows/ralph-development.yml
name: Ralph Autonomous Development

on:
  workflow_dispatch:
    inputs:
      prd_file:
        description: 'Path to PRD file'
        required: true
        default: 'docs/requirements.md'

jobs:
  ralph-dev:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Install Ralph
        run: |
          git clone https://github.com/frankbria/ralph-claude-code.git
          cd ralph-claude-code
          ./install.sh
      
      - name: Enable Ralph (non-interactive)
        run: |
          ralph-enable-ci \
            --from prd "${{ github.event.inputs.prd_file }}" \
            --output-format json
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
      
      - name: Run Ralph Development Loop
        run: |
          ralph --no-continue --timeout 30 --output-format json
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
          RATE_LIMIT_PER_HOUR: 50
```

## Exit Detection

Ralph uses a **dual-condition exit gate**:

1. **Completion Indicators** - Claude marks tasks complete with `[x]`
2. **Explicit EXIT_SIGNAL** - Claude must set `EXIT_SIGNAL: true` in PROMPT.md

```markdown
# PROMPT.md example
## Current Status
- [x] All endpoints implemented
- [x] Tests passing
- [x] Documentation complete

EXIT_SIGNAL: true  # Required to exit
```

Both conditions must be true for Ralph to exit. This prevents premature exits.

### Overriding Exit Signal

If Claude sets `EXIT_SIGNAL: false` explicitly, Ralph continues even with completion indicators:

```markdown
## Current Status
- [x] Basic features complete
- [ ] Performance optimization needed

EXIT_SIGNAL: false  # Continues working
```

## Rate Limiting

Ralph enforces API call limits to prevent overuse:

```bash
# Default: 100 calls per hour
# Configured in .ralphrc:
RATE_LIMIT_PER_HOUR=100

# Override via environment
export RATE_LIMIT_PER_HOUR=150

# View current rate limit status
cat .ralph/logs/ralph.log | grep "Rate limit"
```

When limit reached, Ralph displays countdown timer:
```
Rate limit reached (100/100 calls). Waiting 42 minutes...
```

### 5-Hour API Limit Handling

Claude API enforces a 5-hour limit per account. Ralph detects this with three layers:

1. **Timeout Guard** - Prevents false positives from command timeouts
2. **JSON Parsing** - Detects `rate_limit_event` in structured responses
3. **Filtered Text Fallback** - Checks error messages (excluding timeout errors)

```bash
# Unattended mode: Auto-waits for 5 hours
ralph --monitor

# Interactive mode: Prompts user to continue
ralph  # Shows: "5-hour API limit hit. Wait? [y/N]"
```

**Fixed in v0.11.5**: Timeout (exit code 124) no longer misidentified as API limit.

## Circuit Breaker

Prevents infinite loops by detecting stuck states:

- **Semantic Analysis** - AI-powered response understanding
- **Two-Stage Filtering** - Structural JSON errors → filtered text errors
- **Multi-Line Error Matching** - Detects repeated multi-line errors
- **Configurable Thresholds** - Default: 3 consecutive errors

```bash
# Configure circuit breaker
export CIRCUIT_BREAKER_THRESHOLD=5
export CIRCUIT_BREAKER_RESET_SECONDS=600

# Ralph automatically triggers circuit breaker when:
# - Same error repeated 3+ times
# - No progress for 3+ consecutive loops
# - Completion indicators accumulated without EXIT_SIGNAL
```

## Troubleshooting

### Ralph Exits Prematurely

**Problem**: Ralph exits after 5 loops even though work incomplete.

**Solution**: Ensure Claude sets `EXIT_SIGNAL: false` in PROMPT.md:

```markdown
## Current Status
- [x] Phase 1 complete
- [ ] Phase 2 in progress

EXIT_SIGNAL: false  # Prevent premature exit
```

### Ralph Stuck in Infinite Loop

**Problem**: Ralph repeats same error indefinitely.

**Solution**: Circuit breaker should trigger automatically. If not:

```bash
# Check circuit breaker threshold
cat .ralph/.ralphrc | grep CIRCUIT_BREAKER_THRESHOLD

# Lower threshold for faster detection
export CIRCUIT_BREAKER_THRESHOLD=2

# View error patterns in logs
cat .ralph/logs/claude_responses.log | grep -A 5 "ERROR"
```

### Rate Limit Reached Too Quickly

**Problem**: Hitting 100 calls/hour limit too fast.

**Solution**: Increase timeout to reduce API calls:

```bash
# Longer timeout = fewer loops = fewer API calls
ralph --timeout 60 --monitor

# Or increase rate limit
export RATE_LIMIT_PER_HOUR=200
```

### Session Expired Error

**Problem**: `Session expired (>24 hours old)` message.

**Solution**: Reset session or increase timeout:

```bash
# Reset session manually
ralph --reset-session

# Or increase session timeout
export SESSION_TIMEOUT_HOURS=48
ralph --monitor
```

### Timeout False Positive as API Limit

**Problem** (Fixed in v0.11.5): Command timeout misidentified as 5-hour API limit.

**Solution**: Upgrade to v0.11.5+ which has three-layer detection:

```bash
cd ralph-claude-code
git pull origin main
./install.sh
```

### JSON Parsing Errors

**Problem**: `Error: invalid JSON response` messages.

**Solution**: Ralph automatically falls back to text parsing. To force text mode:

```bash
ralph --output-format text --monitor
```

### Tool Permission Denied

**Problem**: Claude attempts to use disallowed tools.

**Solution**: Update `.ralphrc` with required tools:

```bash
# Edit .ralph/.ralphrc
ALLOWED_TOOLS="Edit,Bash(npm *),Bash(pytest),Bash(git add),Bash(git commit)"

# Or use --allowed-tools flag
ralph --allowed-tools "Edit,Bash(*)" --monitor
```

### Monitor Not Showing Updates

**Problem**: `ralph-monitor` shows stale data.

**Solution**: Check tmux base-index configuration:

```bash
# Ralph supports non-zero base-index
tmux show-options -g base-index

# Or run monitor in separate terminal (no tmux)
# Terminal 1:
ralph

# Terminal 2:
ralph-monitor --refresh 1
```

### Migration Issues from Old Versions

**Problem**: Project structure incompatible after upgrade.

**Solution**: Use `ralph-migrate`:

```bash
cd old-project
ralph-migrate  # Moves files to .ralph/ structure

# Verify migration
ls .ralph/
```

## Best Practices

1. **Start Small**: Begin with well-defined PRD and clear exit criteria
2. **Monitor Progress**: Always use `ralph --monitor` for visibility
3. **Set Realistic Timeouts**: 30-60 minutes allows meaningful work per loop
4. **Version Control**: Commit before running Ralph; review changes after
5. **Task Prioritization**: Use fix_plan.md to guide Claude's focus
6. **Explicit Exit Signals**: Always require Claude to set `EXIT_SIGNAL: true`
7. **Rate Limit Awareness**: Plan for 100 calls/hour default (adjust as needed)
8. **Session Management**: Use `--resume` for context preservation, not `--continue`
9. **Live Streaming**: Use `--live` flag when you need real-time visibility
10. **Regular Updates**: Keep Ralph updated for latest bug fixes and features

## Uninstalling Ralph

```bash
# Download and run uninstall script
curl -sL https://raw.githubusercontent.com/frankbria/ralph-claude-code/main/uninstall.sh | bash

# Or if you have the repo:
cd ralph-claude-code
./uninstall.sh
```

Removes:
- Global commands (`ralph`, `ralph-monitor`, etc.)
- Shell configurations
- Does NOT remove project `.ralph/` directories
