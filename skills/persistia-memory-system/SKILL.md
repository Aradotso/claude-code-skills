---
name: persistia-memory-system
description: Install and use Persistia to give Claude Code persistent, self-updating project memory and operational capabilities
triggers:
  - "set up persistent memory for this project"
  - "install Persistia"
  - "give Claude Code memory of this codebase"
  - "schedule an automated task"
  - "add a permanent skill"
  - "update the brain repository"
  - "create a proactive scan"
  - "configure operational memory"
---

# Persistia Memory System

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

Persistia gives Claude Code persistent, self-updating memory of your entire project. It scans your codebase, architecture, docs, and configs on first run, then monitors changes via `git diff` to stay current. Memory is stored in `_brain/` as plain text files with git versioning, optionally backed up to a private GitHub repository.

## Installation

Run from your project root:

```bash
curl -fsSL https://raw.githubusercontent.com/bernardohcrocha/persistia-for-claude-code/main/setup.sh | bash
```

**Requirements:**
- git
- Node.js 18+
- Optional: GitHub CLI (`gh`) for automatic cloud backup

The setup process will:
1. Scan your project structure
2. Create `_brain/` directory with indexed memory
3. Ask only for information it can't find automatically
4. Install schedulers (launchd on macOS, systemd on Linux)
5. Create a private git repository for the brain
6. Set up automatic daily updates

## Directory Structure

```
_brain/
├── .git/                    # Isolated brain repository
├── index.md                 # Main navigation file (read first every session)
├── dashboard.html           # Live dashboard (auto-refreshes every 5 min)
├── core/                    # Product, brand, ICP definitions
│   ├── product.md
│   ├── brand.md
│   └── icp.md
├── operations/              # Metrics and customer data (auto-updated daily)
│   ├── metrics.md
│   └── customers.md
├── skills/                  # Permanent rules and capabilities
│   └── [skill-name].md
└── tasks/                   # Scheduled task management
    ├── queue.json
    ├── completed/
    └── failed/
```

## Core Concepts

### Memory Index

The `_brain/index.md` file is always read first. It contains:
- Project overview and architecture
- Links to all memory files
- Recent changes summary
- Navigation structure

### Self-Updating Memory

Persistia runs `git diff` daily to detect changes:
- Only modified files are re-indexed
- Token-efficient: 1,000 files with 3 changes = reads 3 files
- Automatic commit and push to brain repository

### Skills

Skills are permanent capabilities stored in `_brain/skills/`. Once written, they're loaded in every session.

**Creating a skill:**

```markdown
---
name: deployment-checklist
created: 2026-06-04
---

# Deployment Checklist

Before every deployment:
1. Run full test suite
2. Check environment variables match production
3. Verify database migrations are reversible
4. Update CHANGELOG.md
5. Tag release in git
```

Skills are created automatically when you say things like:
- "Remember to always run tests before deploying"
- "Add a skill for handling customer data exports"
- "Create a permanent rule about error handling"

### Scheduled Tasks

Tasks are defined in `_brain/tasks/queue.json`:

```json
{
  "tasks": [
    {
      "id": "weekly-metrics",
      "name": "Weekly Metrics Report",
      "schedule": "every Monday at 9am",
      "description": "Pull last week's metrics from Stripe, flag anomalies, queue follow-ups for accounts below quota",
      "batch_size": 50,
      "timeout_minutes": 10,
      "enabled": true
    },
    {
      "id": "inactive-users",
      "name": "Inactive User Outreach",
      "schedule": "every Tuesday",
      "description": "Find signups from last 30 days with no activity. Research each company and draft personalized re-engagement emails",
      "batch_size": 20,
      "timeout_minutes": 15,
      "enabled": true
    }
  ]
}
```

**Task properties:**
- `batch_size`: Maximum items per run (prevents timeouts)
- `timeout_minutes`: Default is 5, increase for complex tasks
- `enabled`: Set to false to pause without deleting

### Proactive Scans

Every 3 days when idle, Persistia automatically scans:
- Metrics trends
- Customer behavior patterns
- Support channels
- Code changes

It flags observations and suggests actions but never acts unilaterally.

## Common Usage Patterns

### Initial Setup Flow

After installation, provide context through conversation:

```
You: "This is a SaaS analytics platform built with Next.js and PostgreSQL"
Claude: *Updates _brain/core/product.md with architecture details*

You: "We target B2B marketing teams at companies with 50-500 employees"
Claude: *Updates _brain/core/icp.md with target customer profile*
```

### Reading Environment Variables

Persistia scans `.env` files automatically. Reference credentials:

```javascript
// Good - uses existing environment variables
const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);
const db = new Database({
  host: process.env.DB_HOST,
  password: process.env.DB_PASSWORD
});
```

### Adding Operational Skills

```
You: "Always check Sentry for errors before marking a task complete"
Claude: *Creates _brain/skills/pre-completion-checklist.md*

You: "When analyzing churn, segment by signup cohort and usage tier"
Claude: *Creates _brain/skills/churn-analysis-methodology.md*
```

### Scheduling Recurring Tasks

```
You: "Every Friday afternoon, summarize this week's closed issues and PRs into a changelog draft"
Claude: *Adds task to _brain/tasks/queue.json*

You: "Daily at 8am: check for new Stripe subscriptions and add them to the customer database"
Claude: *Adds task with appropriate batch_size and timeout*
```

### Cross-Tool Queries

```
You: "Which customers dropped usage 30%+ this month? Cross-check their support tickets and draft personalized follow-ups"
Claude: *Queries metrics, support system, generates drafts*

You: "Find all users who signed up from our latest blog post and haven't activated yet"
Claude: *Checks analytics, CRM, prepares outreach list*
```

## Dashboard

Open `_brain/dashboard.html` in your browser for a live view:
- Recent memory updates
- Scheduled task status
- Proactive scan results
- Quick navigation to all brain files

Auto-refreshes every 5 minutes.

## Manual Operations

### Force Memory Update

```bash
cd _brain
git add .
git commit -m "Manual memory update"
git push
```

### View Memory History

```bash
cd _brain
git log --oneline
git diff HEAD~1  # See what changed in last update
```

### Share Memory with Team

```bash
# On first machine
cd _brain
git remote -v  # Get repository URL

# On second machine (in project root)
git clone <brain-repo-url> _brain
```

### Disable Automatic Updates

```bash
# macOS
launchctl unload ~/Library/LaunchAgents/persistia.brain.plist

# Linux
systemctl --user disable persistia-brain.timer
```

### Re-enable Automatic Updates

```bash
# macOS
launchctl load ~/Library/LaunchAgents/persistia.brain.plist

# Linux
systemctl --user enable persistia-brain.timer
systemctl --user start persistia-brain.timer
```

## Troubleshooting

### Brain repository not pushing to GitHub

Check if GitHub CLI is authenticated:

```bash
gh auth status
```

If not authenticated:

```bash
gh auth login
```

### Memory updates not running automatically

Check scheduler status:

```bash
# macOS
launchctl list | grep persistia

# Linux
systemctl --user status persistia-brain.timer
```

View logs:

```bash
# macOS
tail -f ~/Library/Logs/persistia-brain.log

# Linux
journalctl --user -u persistia-brain.service -f
```

### Task timeouts

Increase `timeout_minutes` in `_brain/tasks/queue.json`:

```json
{
  "id": "complex-analysis",
  "timeout_minutes": 20,
  "batch_size": 10
}
```

### Memory files growing too large

Split large files by topic. Update `_brain/index.md` to link to all sections:

```markdown
## Architecture

- [Backend Services](core/architecture-backend.md)
- [Frontend Components](core/architecture-frontend.md)
- [Database Schema](core/architecture-database.md)
```

### Resetting memory completely

```bash
rm -rf _brain
curl -fsSL https://raw.githubusercontent.com/bernardohcrocha/persistia-for-claude-code/main/setup.sh | bash
```

This will re-scan your project and rebuild all memory files.

## Best Practices

1. **Be conversational during setup** - Persistia learns from natural conversation, not forms
2. **Review proactive scans** - Check suggestions before acting on them
3. **Keep skills focused** - One skill per workflow or rule
4. **Use batch_size for large datasets** - Prevents timeouts and token limits
5. **Commit brain changes** - Treat `_brain/` like important code
6. **Back up to GitHub** - Install `gh` CLI for automatic cloud backup
7. **Share with team** - Clone brain repository across machines for consistent context

## Integration Examples

### Stripe Metrics

```javascript
// _brain/skills/stripe-metrics.md references this pattern
const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);

async function getMonthlyMetrics() {
  const subscriptions = await stripe.subscriptions.list({
    status: 'active',
    limit: 100
  });
  
  const revenue = subscriptions.data.reduce((sum, sub) => {
    return sum + (sub.items.data[0].price.unit_amount / 100);
  }, 0);
  
  return { active_subscriptions: subscriptions.data.length, revenue };
}
```

### GitHub Issue Analysis

```javascript
// Uses existing gh CLI authentication
const { execSync } = require('child_process');

function getRecentIssues(days = 7) {
  const since = new Date(Date.now() - days * 24 * 60 * 60 * 1000).toISOString();
  const cmd = `gh issue list --state closed --json title,closedAt,labels --search "closed:>=${since}"`;
  const output = execSync(cmd, { encoding: 'utf-8' });
  return JSON.parse(output);
}
```

### Customer Database Updates

```javascript
// Credentials from environment
const { Pool } = require('pg');
const pool = new Pool({
  host: process.env.DB_HOST,
  database: process.env.DB_NAME,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD
});

async function updateCustomerMetrics() {
  const result = await pool.query(`
    SELECT customer_id, last_active, usage_count
    FROM customers
    WHERE last_active < NOW() - INTERVAL '30 days'
  `);
  return result.rows;
}
```

## Architecture

Persistia uses an isolated git repository for `_brain/`:
- Separate from main project repository
- Pushes to private GitHub remote (if `gh` CLI available)
- Contains only memory files, no code
- Can be cloned independently across machines
- Full version history of memory evolution

See the project's ARCHITECTURE.md for detailed git workflow.
