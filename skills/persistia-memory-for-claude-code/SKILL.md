---
name: persistia-memory-for-claude-code
description: Install and configure Persistia to give Claude Code self-updating persistent memory across sessions
triggers:
  - "set up persistent memory for this project"
  - "install persistia brain system"
  - "give claude persistent memory"
  - "configure self-updating project memory"
  - "set up autonomous task scheduling"
  - "create persistent brain repository"
  - "install persistia for claude code"
  - "enable automatic context updates"
---

# Persistia Memory for Claude Code

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

Persistia gives Claude Code self-updating persistent memory by creating a `_brain/` directory that tracks your project context, skills, tasks, and operations. It uses git diffs to update only what changed, making it token-efficient and always current.

## What It Does

- **Initial project scan** — reads code, docs, configs, environment files on first run
- **Self-updating context** — runs `git diff` daily, only reads changed files
- **Persistent skills** — rules taught once are saved to `_brain/skills/` and loaded every session
- **Autonomous tasks** — schedule tasks in plain language with `batch_size` and `timeout_minutes` controls
- **Proactive monitoring** — scans metrics and customers every 3 days when idle
- **Private git repo** — entire brain is version-controlled in isolated repository with optional GitHub backup

## Installation

Run from project root:

```bash
curl -fsSL https://raw.githubusercontent.com/bernardohcrocha/persistia-for-claude-code/main/setup.sh | bash
```

**Requirements:**
- git
- Node.js 18+
- Optional: GitHub CLI (`gh`) for automatic private repo creation and cloud backup

The setup script:
1. Creates `_brain/` directory structure
2. Performs initial project scan
3. Asks for missing context (credentials, product details)
4. Initializes isolated git repository
5. Installs scheduler (launchd on macOS, systemd on Linux)
6. Optionally creates private GitHub remote

## Directory Structure

```
_brain/
├── .git/                  # Isolated brain repository
├── index.md               # Agent reads this first every session
├── dashboard.html         # Live command center (auto-refreshes every 5 min)
├── core/                  # Product, brand, ICP definitions
│   ├── product.md
│   ├── brand.md
│   └── icp.md
├── operations/            # Auto-updated metrics and customer data
│   ├── metrics.json
│   └── customers/
├── skills/                # Permanent rules and behaviors
│   ├── code-style.md
│   ├── deployment.md
│   └── ...
└── tasks/                 # Scheduled task queue
    ├── queue.json
    └── completed/
```

## Key Concepts

### Skills (Permanent Memory)

Skills are persistent rules that apply to all future sessions. Create them conversationally:

**Example interaction:**
```
You: "Always use TypeScript strict mode and include JSDoc for public APIs"
Claude: [creates _brain/skills/typescript-conventions.md]
```

Skills are automatically loaded at session start via `index.md`.

### Tasks (Autonomous Scheduling)

Schedule tasks in `_brain/tasks/queue.json`:

```json
{
  "tasks": [
    {
      "id": "weekly-metrics",
      "description": "Pull last week's numbers from Stripe, flag anomalies, queue follow-ups for accounts below quota",
      "schedule": "weekly",
      "day": "monday",
      "time": "09:00",
      "batch_size": 50,
      "timeout_minutes": 10,
      "enabled": true
    },
    {
      "id": "inactive-signups",
      "description": "Find signups from last 30 days with no activity. Visit company websites and draft personalized re-engagement emails",
      "schedule": "daily",
      "time": "14:00",
      "batch_size": 20,
      "timeout_minutes": 5,
      "enabled": true
    }
  ]
}
```

**Task properties:**
- `batch_size` — cap items processed per run (prevents timeouts)
- `timeout_minutes` — default 5, increase for complex operations
- `schedule` — `daily`, `weekly`, `monthly`
- `enabled` — toggle without deleting

### Operations (Auto-Updated Context)

`_brain/operations/` contains auto-updated data from your tools:

**metrics.json example:**
```json
{
  "updated_at": "2026-06-04T10:00:00Z",
  "revenue": {
    "mrr": 45000,
    "change_30d": 12.5
  },
  "customers": {
    "total": 234,
    "active": 198,
    "at_risk": 12
  },
  "support": {
    "open_tickets": 8,
    "avg_response_hours": 2.3
  }
}
```

Persistia reads credentials from `.env` to update this automatically.

## Common Usage Patterns

### Initial Setup Conversation

```
You: "Install persistia"
Claude: [runs setup.sh]
Claude: "I need a few details. What's your product positioning in one sentence?"
You: "Developer platform for real-time data sync"
Claude: "What integrations should I track?"
You: "Stripe for revenue, Intercom for support, PostgreSQL for usage"
Claude: [creates _brain/core/product.md, configures integrations]
```

### Teaching a Skill

```
You: "When deploying, always run tests, build, then push to staging before production"
Claude: [creates _brain/skills/deployment-process.md]

# Next session
You: "Deploy the latest changes"
Claude: [automatically follows deployment-process.md]
```

### Scheduling a Task

```
You: "Every Monday at 9am, check which customers had usage drops >30% and draft personalized check-in emails"
Claude: [adds task to _brain/tasks/queue.json with batch_size and timeout]
```

### Asking Cross-Tool Questions

```
You: "Which customers churned this month and what were their last support interactions?"
Claude: [queries _brain/operations/metrics.json and customer records, cross-references support history]
```

### Viewing Dashboard

Open `_brain/dashboard.html` in browser — auto-refreshes every 5 minutes showing:
- Recent metrics trends
- Task queue status
- At-risk customers
- Pending actions

## Configuration

### Environment Variables

Persistia reads existing `.env` files for integrations. Common variables:

```bash
# Stripe
STRIPE_API_KEY=sk_live_xxxxx

# Database
DATABASE_URL=postgresql://user:pass@host:5432/db

# Support
INTERCOM_ACCESS_TOKEN=xxxxx

# GitHub (optional, for brain repo backup)
GITHUB_TOKEN=ghp_xxxxx
```

### Customizing Update Frequency

Edit scheduler configuration:

**macOS (launchd):**
```bash
nano ~/Library/LaunchAgents/com.persistia.brain-update.plist
```

**Linux (systemd):**
```bash
nano ~/.config/systemd/user/persistia-brain-update.timer
```

Default: daily at 3am. Change `<integer>` values for `Hour` and `Minute`.

### Batch Size and Timeout Tuning

For large datasets, adjust task settings:

```json
{
  "id": "large-customer-outreach",
  "batch_size": 100,        // Process 100 at a time
  "timeout_minutes": 15,    // Allow 15 min per batch
  "enabled": true
}
```

## Troubleshooting

### Brain Not Updating

Check scheduler status:

**macOS:**
```bash
launchctl list | grep persistia
launchctl start com.persistia.brain-update
```

**Linux:**
```bash
systemctl --user status persistia-brain-update.timer
systemctl --user start persistia-brain-update.service
```

View logs:
```bash
# macOS
tail -f ~/Library/Logs/persistia-brain-update.log

# Linux
journalctl --user -u persistia-brain-update.service -f
```

### Context Not Loading

Verify `_brain/index.md` exists and contains navigation to all sections:

```bash
cat _brain/index.md
```

Should reference `core/`, `skills/`, `operations/`, and `tasks/`.

### GitHub Sync Failing

Re-authenticate GitHub CLI:

```bash
gh auth login
gh auth refresh -s repo
```

Manually push brain repo:

```bash
cd _brain
git push origin main
```

### High Token Usage

Persistia uses git diff to minimize token consumption. If still high:

1. Check `git diff` output size:
   ```bash
   git diff --stat
   ```

2. Add large generated files to `.gitignore`:
   ```bash
   echo "dist/" >> .gitignore
   echo "build/" >> .gitignore
   ```

3. Reduce task `batch_size` in `_brain/tasks/queue.json`

### Task Timeouts

Increase `timeout_minutes` for complex tasks:

```json
{
  "id": "complex-analysis",
  "timeout_minutes": 20,
  "batch_size": 10
}
```

Or split into smaller subtasks with dependencies.

## Manual Commands

Force brain update:
```bash
cd _brain && git diff HEAD~1 && npm run update
```

View brain commit history:
```bash
cd _brain && git log --oneline
```

Export brain to share with team:
```bash
cd _brain && git bundle create ../project-brain.bundle --all
```

Import shared brain:
```bash
git clone project-brain.bundle _brain
```

Disable proactive scanning:
```bash
echo '{"proactive_scan": false}' > _brain/config.json
```

## Best Practices

1. **Teach incrementally** — add skills as you work, not all upfront
2. **Use descriptive task IDs** — makes queue management easier
3. **Start with small batch_size** — increase after confirming reliability
4. **Review proactive suggestions** — Persistia suggests, you decide
5. **Commit brain changes** — treat `_brain/` like documentation
6. **Back up to GitHub** — install `gh` for automatic cloud sync
7. **Share skills with team** — clone brain repo across team members

## Architecture Notes

- Brain repo is isolated — separate `.git` from main project
- Uses symbolic execution for git operations (no submodules)
- Scheduler runs in user space (no sudo required)
- All data plain text and version-controlled
- No external APIs except GitHub (optional)

For detailed architecture: [ARCHITECTURE.md](https://github.com/bernardohcrocha/persistia-for-claude-code/blob/main/ARCHITECTURE.md)
