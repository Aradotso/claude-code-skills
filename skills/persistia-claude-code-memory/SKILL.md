---
name: persistia-claude-code-memory
description: Install and manage Persistia to give Claude Code persistent, self-updating memory across sessions
triggers:
  - "set up persistent memory for Claude Code"
  - "install Persistia"
  - "give Claude Code memory"
  - "configure operational memory"
  - "create a self-updating agent brain"
  - "set up automatic project indexing"
  - "schedule autonomous tasks"
  - "make Claude Code remember my project"
---

# Persistia for Claude Code

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

Persistia gives Claude Code persistent, self-updating memory. It scans your project once, stores structured knowledge in `_brain/`, then monitors changes via `git diff` to stay current. Enables autonomous tasks, permanent skills, and cross-session context.

## Installation

Must be run from **project root** with git initialized:

```bash
curl -fsSL https://raw.githubusercontent.com/bernardohcrocha/persistia-for-claude-code/main/setup.sh | bash
```

**Requirements:**
- git
- Node.js 18+
- Optional: GitHub CLI (`gh`) for automatic private brain repository backup

The installer:
1. Creates `_brain/` directory structure
2. Scans project files, reads `.env` files
3. Asks only what it can't auto-detect
4. Sets up automatic daily updates (launchd/systemd)
5. Optionally creates private GitHub repo for brain backup

## Directory Structure

```
_brain/
├── .git/                    # Isolated brain repository
├── index.md                 # Master navigation, read first every session
├── dashboard.html           # Live command center (auto-refresh)
├── core/
│   ├── product.md          # What you're building
│   ├── brand.md            # Voice, values, positioning
│   └── icp.md              # Ideal customer profile
├── operations/
│   ├── metrics.md          # KPIs, auto-updated daily
│   └── customers.md        # Account data, behaviors
├── skills/
│   └── *.md                # Permanent rules and behaviors
└── tasks/
    ├── queue.json          # Scheduled autonomous tasks
    └── completed/          # Execution logs
```

## Initial Setup Flow

After installation, Claude will ask clarifying questions:

```bash
# Example questions Claude might ask:
# - "What's your product in one sentence?"
# - "What's your primary tech stack?"
# - "Which tools do you use? (Stripe, Postgres, etc.)"
# - "What's your ICP?"
```

Claude reads existing files (`.env`, `package.json`, README, etc.) and only asks what it can't find.

## Core Concepts

### 1. Index-First Navigation

`_brain/index.md` is the entry point. Claude reads this first every session:

```markdown
# Project Brain Index

## Quick Reference
- Product: [core/product.md]
- Stack: Node.js, Postgres, Stripe
- Last updated: 2026-06-04

## Priority Files
1. operations/metrics.md — current numbers
2. core/icp.md — who we serve
3. skills/code-style.md — how we write code

## Active Tasks
- Weekly report (runs Mondays)
- Churn detection (runs daily)
```

### 2. Skills (Permanent Memory)

Skills are rules that persist across sessions. Created automatically when you teach Claude something:

**Example:** "Always use functional components in React"

Claude creates `_brain/skills/react-patterns.md`:

```markdown
---
created: 2026-06-04
category: code-style
---

# React Patterns

## Component Style
- Always use functional components
- Prefer hooks over class methods
- Use TypeScript for props

## Example
```javascript
// ✅ Correct
const Button: React.FC<ButtonProps> = ({ label, onClick }) => {
  return <button onClick={onClick}>{label}</button>;
};

// ❌ Avoid
class Button extends React.Component { ... }
```
```

Skills are loaded at every session start.

### 3. Autonomous Tasks

Tasks run automatically with full project context:

**Edit `_brain/tasks/queue.json`:**

```json
{
  "tasks": [
    {
      "id": "weekly-metrics",
      "name": "Weekly Metrics Report",
      "description": "Pull revenue, churn, new signups from Stripe. Flag anomalies.",
      "schedule": "weekly",
      "day": "monday",
      "time": "09:00",
      "timeout_minutes": 10,
      "enabled": true
    },
    {
      "id": "churn-detection",
      "name": "At-Risk Customer Detection",
      "description": "Find customers with 30%+ usage drop. Check support history. Draft re-engagement.",
      "schedule": "daily",
      "time": "08:00",
      "batch_size": 20,
      "timeout_minutes": 5,
      "enabled": true
    },
    {
      "id": "onboarding-follow-up",
      "name": "New User Follow-up",
      "description": "Find signups from last 30 days with no activity. Visit company website. Write personalized email.",
      "schedule": "daily",
      "time": "10:00",
      "batch_size": 10,
      "enabled": true
    }
  ]
}
```

**Task Properties:**
- `schedule`: `"daily"`, `"weekly"`, `"monthly"`
- `batch_size`: Max items per run (prevents timeouts)
- `timeout_minutes`: Max execution time (default: 5)
- `day`: For weekly tasks (`"monday"`, `"tuesday"`, etc.)
- `time`: 24-hour format

Completed tasks log to `_brain/tasks/completed/YYYY-MM-DD_task-id.md`

### 4. Git Diff-Based Updates

Daily scheduler runs:

```bash
cd _brain && git diff HEAD~1..HEAD --name-only
```

Only changed files get re-indexed. Example:

```
# 1,000 files in project
# 3 changed today
# Claude only reads those 3
```

### 5. Credentials Management

Persistia reads existing credentials from:
- `.env` files
- Environment variables
- Git config
- Tool CLIs (Stripe, AWS, etc.)

No re-entry required. Uses what you already have.

## Common Usage Patterns

### Teaching New Skills

**User:** "Always format prices with 2 decimals and currency symbol"

Claude creates `_brain/skills/formatting.md`:

```markdown
# Formatting Rules

## Prices
Always display: `$X,XXX.XX`

```javascript
const formatPrice = (cents) => {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
  }).format(cents / 100);
};
```
```

### Scheduling Cross-Tool Tasks

**User:** "Every Monday, pull Stripe revenue and compare to Postgres usage metrics. Flag discrepancies."

Claude adds to `queue.json` and writes execution logic in `_brain/tasks/scripts/revenue-check.js`:

```javascript
const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);
const { Pool } = require('pg');

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
});

async function checkRevenueMatch() {
  const startDate = new Date();
  startDate.setDate(startDate.getDate() - 7);
  
  // Stripe revenue
  const charges = await stripe.charges.list({
    created: { gte: Math.floor(startDate / 1000) },
    limit: 100,
  });
  
  const stripeTotal = charges.data
    .filter(c => c.status === 'succeeded')
    .reduce((sum, c) => sum + c.amount, 0);
  
  // Postgres usage
  const result = await pool.query(`
    SELECT customer_id, SUM(usage_amount) as total
    FROM usage_events
    WHERE created_at >= $1
    GROUP BY customer_id
  `, [startDate]);
  
  // Compare and flag
  const discrepancies = [];
  // ... comparison logic
  
  return discrepancies;
}

module.exports = { checkRevenueMatch };
```

### Updating Core Knowledge

**User:** "We pivoted — now targeting dev tools, not marketing SaaS"

Claude updates `_brain/core/icp.md` and `_brain/core/product.md`, commits changes:

```bash
cd _brain
git add core/icp.md core/product.md
git commit -m "Update: pivot to dev tools market"
git push origin main  # If gh CLI configured
```

### Querying Operational Data

**User:** "Which customers churned last month?"

Claude reads `_brain/operations/customers.md` (auto-updated daily from Stripe/DB), answers immediately without API calls.

## Dashboard

Open in browser:

```bash
open _brain/dashboard.html
```

Auto-refreshes every 5 minutes. Shows:
- Recent metrics
- Active tasks status
- Last brain update
- Quick actions

## Troubleshooting

### Brain not updating

Check scheduler status:

**macOS:**
```bash
launchctl list | grep persistia
launchctl start com.persistia.update
```

**Linux:**
```bash
systemctl status persistia-update.timer
systemctl start persistia-update.service
```

### Task timing out

Increase `timeout_minutes` or reduce `batch_size`:

```json
{
  "id": "large-task",
  "timeout_minutes": 15,
  "batch_size": 5
}
```

### Memory getting stale

Force re-index:

```bash
cd _brain
rm -rf operations/
# Restart Claude Code session
```

### Git conflicts in brain repo

```bash
cd _brain
git fetch origin
git reset --hard origin/main
```

## Advanced Configuration

### Custom Update Frequency

Edit scheduler config:

**macOS** (`~/Library/LaunchAgents/com.persistia.update.plist`):
```xml
<key>StartInterval</key>
<integer>3600</integer>  <!-- seconds, 3600 = hourly -->
```

**Linux** (`~/.config/systemd/user/persistia-update.timer`):
```ini
[Timer]
OnCalendar=hourly
```

### Excluding Files from Indexing

Add to `.gitignore` in project root:

```
_brain/
node_modules/
.env
*.log
```

Persistia respects `.gitignore`.

### Multi-Agent Teams

Share brain repo with team:

```bash
cd _brain
git remote add origin git@github.com:yourteam/project-brain.git
git push -u origin main
```

Team members clone after installing Persistia:

```bash
rm -rf _brain
git clone git@github.com:yourteam/project-brain.git _brain
```

## Best Practices

1. **Teach once, benefit forever** — when you explain something, ask Claude to save it as a skill
2. **Let git diff do the work** — don't manually update `_brain/`, let the scheduler handle it
3. **Batch large operations** — use `batch_size` to prevent timeouts
4. **Review task logs** — check `_brain/tasks/completed/` weekly
5. **Commit brain changes** — treat `_brain/` like code, commit meaningful updates

## Integration Examples

### Stripe + Postgres Sync

```javascript
// _brain/tasks/scripts/sync-revenue.js
const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);
const { Pool } = require('pg');

async function syncRevenue() {
  const pool = new Pool({ connectionString: process.env.DATABASE_URL });
  
  const subscriptions = await stripe.subscriptions.list({ limit: 100 });
  
  for (const sub of subscriptions.data) {
    await pool.query(`
      INSERT INTO revenue (customer_id, amount, period_start)
      VALUES ($1, $2, $3)
      ON CONFLICT (customer_id, period_start) DO UPDATE
      SET amount = $2
    `, [sub.customer, sub.plan.amount, new Date(sub.current_period_start * 1000)]);
  }
}
```

### Slack Alert on Anomaly

```javascript
// _brain/tasks/scripts/alert-churn.js
const axios = require('axios');

async function alertIfChurnSpike(churnRate) {
  if (churnRate > 0.05) {
    await axios.post(process.env.SLACK_WEBHOOK_URL, {
      text: `⚠️ Churn spike detected: ${(churnRate * 100).toFixed(1)}%`,
    });
  }
}
```

## File Formats

All brain files are Markdown with optional YAML frontmatter:

```markdown
---
updated: 2026-06-04
category: operations
auto_update: true
---

# Metrics

## MRR
$12,450 (+8.2% MoM)

## Active Users
342 (-2 from last week)
```

Claude updates these automatically. Manual edits are respected.
