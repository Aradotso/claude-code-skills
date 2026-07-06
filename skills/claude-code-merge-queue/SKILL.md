---
name: claude-code-merge-queue
description: Local FIFO merge queue for parallel Claude Code agents with zero runtime dependencies
triggers:
  - set up merge queue for claude code agents
  - configure parallel agent worktrees
  - manage concurrent claude code sessions
  - serialize agent landings to main branch
  - prevent concurrent build conflicts
  - coordinate multiple claude agents
  - queue agent pushes to integration branch
  - isolate claude code worktrees
---

# Claude Code Merge Queue

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

A local merge queue that coordinates parallel Claude Code agents. Prevents race conditions when multiple isolated worktrees try to land changes simultaneously.

## What It Solves

When running multiple `claude --worktree` sessions:
- **Push races**: Multiple agents push to the same branch, causing rejected pushes and rebases
- **Build contention**: Concurrent builds overload the machine
- **Test conflicts**: Tests hitting shared resources (databases, ports) interfere with each other

Claude Code Merge Queue serializes landings through a FIFO queue and coordinates builds across all lanes.

## Installation

```bash
npm install --save-dev claude-code-merge-queue
```

Or with other package managers:
```bash
pnpm add -D claude-code-merge-queue
yarn add -D claude-code-merge-queue
bun add -d claude-code-merge-queue
```

## Quick Setup

```bash
npx claude-code-merge-queue init
```

This auto-configures:
- **`claude-code-merge-queue.config.mjs`** — detects integration branch and check command
- **`CLAUDE.md`** — instructions for agents to land automatically
- **`.claude/settings.json`** — WorktreeCreate hook registration
- **`.husky/pre-push`** — enforces queue usage
- **`package.json` scripts** — `land`, `sync`, `promote`, `preview` commands
- **Preflight safety** — catches stale branches before push

After init, **commit all changes** and start using:
```bash
claude --worktree feature-name
```

## Core Commands

### `claude-code-merge-queue hook worktree-create`

Claude Code WorktreeCreate hook. Automatically creates numbered lanes when you run `claude --worktree`:

```bash
# In .claude/settings.json (init does this):
{
  "hooks": {
    "WorktreeCreate": "claude-code-merge-queue hook worktree-create"
  }
}
```

Creates worktrees as `../project-lane-1`, `../project-lane-2`, etc.

### `claude-code-merge-queue land`

Rebases current lane onto integration branch and pushes through FIFO queue:

```bash
npm run land
# or directly:
claude-code-merge-queue land
```

**Serialization**: Only one lane pushes at a time, machine-wide.

**Checks run first**: `checkCommand` must pass before push proceeds.

**Auto-cleanup**: Removes already-landed sibling lane worktrees.

Agents run this themselves after their changes pass local tests.

### `claude-code-merge-queue sync`

Fast-forwards main checkout to latest and reinstalls dependencies if needed:

```bash
npm run sync
```

Run after landing to see changes in your main dev server.

### `claude-code-merge-queue promote`

Ships integration branch to production:

```bash
npm run promote
```

**Human-only command** — never in agent instructions.

### `claude-code-merge-queue preview`

Mirrors a lane's working tree (including uncommitted changes) onto main checkout:

```bash
npm run preview
# Then switch back:
npm run preview:restore
```

For quick inspection without rebuilding.

### `claude-code-merge-queue build-lock -- <cmd>`

Serializes heavy build commands across all lanes:

```typescript
// In package.json:
{
  "scripts": {
    "build": "claude-code-merge-queue build-lock -- vite build",
    "check": "claude-code-merge-queue build-lock -- npm run type-check && npm test"
  }
}
```

Prevents concurrent builds from overwhelming the machine.

### `claude-code-merge-queue port`

Returns lane's dev server port (derived from lane number):

```bash
PORT=$(claude-code-merge-queue port)
npm run dev -- --port $PORT
```

With `portBase: 3000`, lane-1 gets 3001, lane-2 gets 3002, etc.

### `claude-code-merge-queue prune`

Removes already-landed sibling worktrees immediately:

```bash
claude-code-merge-queue prune
```

`land` does this automatically, but this is useful for manual cleanup.

## Configuration

**`claude-code-merge-queue.config.mjs`** (ES module format):

```javascript
export default {
  // Lane naming
  branchPrefix: "lane/",           // Creates lane/1, lane/2
  worktreeSuffix: "-lane-",        // Creates ../project-lane-1
  
  // Port allocation
  portBase: 3000,                  // lane-1 → 3001, lane-2 → 3002
  
  // Branch strategy
  integrationBranch: "main",       // Where agents land
  productionBranch: null,          // Set for staging→prod model
  protectedBranches: [],           // Extra protected branches
  
  // Landing requirements
  checkCommand: "npm run check",   // Must pass before landing
  checksRequired: true,            // false = no checks (explicit opt-out)
  
  // Build coordination
  regenerableFiles: [              // Never block rebase on these
    "package-lock.json",
    "pnpm-lock.yaml"
  ],
  buildOutputDirs: [               // preview skips these
    "dist",
    "build",
    ".next"
  ],
  
  // Shared resources
  symlinks: [                      // Symlinked from main checkout
    ".env",
    ".env.local",
    "node_modules"
  ]
};
```

### Two-Stage Branch Model

For staging→production workflow:

```javascript
export default {
  integrationBranch: "staging",    // Agents land here
  productionBranch: "main",        // promote ships to here
  checkCommand: "npm run check"
};
```

```bash
# Agents land to staging:
npm run land

# You decide when to ship:
npm run promote  # staging → main
```

## Real-World Patterns

### TypeScript Project with Full Checks

```javascript
// claude-code-merge-queue.config.mjs
export default {
  branchPrefix: "lane/",
  worktreeSuffix: "-lane-",
  portBase: 3000,
  integrationBranch: "main",
  productionBranch: null,
  checkCommand: "npm run check:push",
  checksRequired: true,
  symlinks: [".env", "node_modules"],
  buildOutputDirs: ["dist"],
  regenerableFiles: ["package-lock.json"]
};
```

```json
// package.json
{
  "scripts": {
    "type-check": "tsc --noEmit",
    "lint": "eslint src",
    "test": "vitest run",
    "build": "claude-code-merge-queue build-lock -- tsc && vite build",
    "check:push": "npm run type-check && npm run lint && npm run test",
    "land": "claude-code-merge-queue land",
    "sync": "claude-code-merge-queue sync",
    "promote": "claude-code-merge-queue promote"
  }
}
```

### React App with Separate Staging

```javascript
// claude-code-merge-queue.config.mjs
export default {
  branchPrefix: "lane/",
  worktreeSuffix: "-lane-",
  portBase: 3000,
  integrationBranch: "staging",
  productionBranch: "main",
  checkCommand: "npm run check",
  checksRequired: true,
  symlinks: [".env.local", "node_modules"],
  buildOutputDirs: [".next", "out"],
  regenerableFiles: ["package-lock.json"]
};
```

### Monorepo with Shared Database Tests

```typescript
// examples/ephemeral-tmp-dir.example.ts
import { registerEphemeralResource } from 'claude-code-merge-queue/ephemeral';
import { spawn } from 'child_process';
import { mkdtemp, rm } from 'fs/promises';
import { tmpdir } from 'os';
import { join } from 'path';

// Each lane gets isolated database directory
const dbDir = await registerEphemeralResource({
  async create() {
    const dir = await mkdtemp(join(tmpdir(), 'test-db-'));
    
    // Start postgres in this directory
    const pg = spawn('postgres', ['-D', dir], {
      detached: true,
      stdio: 'ignore'
    });
    pg.unref();
    
    return { path: dir, pid: pg.pid };
  },
  async destroy(resource) {
    // Kill postgres
    try {
      process.kill(resource.pid);
    } catch {}
    
    // Remove directory
    await rm(resource.path, { recursive: true, force: true });
  }
});

// Use in tests
process.env.TEST_DB_PATH = dbDir.path;
```

## Pre-Push Hook

The hook enforces queue usage and runs checks:

```bash
# .husky/pre-push (init creates this)
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

claude-code-merge-queue hook pre-push "$@"
```

Blocks:
- Direct pushes to integration branch → "Use `npm run land` instead"
- Pushes to production branch → "Use `npm run promote` instead"
- Any push without passing `checkCommand`

## Emergency Override

For genuine emergencies (init creates this):

```bash
# Bypass queue (branch name required for safety):
CLAUDE_CODE_MERGE_QUEUE_EMERGENCY_PUSH=1 git push origin HEAD:main
```

This is a convention, not cryptographic enforcement.

## Troubleshooting

### "command not found: claude-code-merge-queue"

Lane is stale relative to origin. The preflight script catches this:

```bash
# Rebase onto latest integration branch:
git fetch origin
git rebase origin/main
```

### Stale lock preventing land

If a process crashes mid-land, the next land detects the PID is dead and reclaims:

```bash
# Just retry:
npm run land
```

### Tests fail in lane but pass locally

Likely shared resource conflict. Check for:
- Same port numbers across lanes
- Shared database without isolation
- Temp files in predictable locations

Use `claude-code-merge-queue port` for ports and `ephemeral.ts` for databases.

### Lane out of sync after rebase

```bash
# In lane worktree:
npm run sync  # Updates main checkout
git rebase origin/main
```

### Build artifacts from other lanes

```bash
# Clean and rebuild:
rm -rf dist build .next
npm run build
```

## Agent Instructions

`init` adds this to `CLAUDE.md`:

```markdown
## Landing Changes

When your changes are complete and tests pass:

1. Run `npm run land` to push through the merge queue
2. If it succeeds, your work is done
3. If checks fail, fix them and retry

Never use `git push` directly to main/staging.
```

This makes landings automatic without human intervention.

## Architecture Notes

### Queue Implementation

FIFO queue uses filesystem locks in `os.tmpdir()`:
- Atomic operations via exclusive file creation
- PID tagging for crash safety
- No stale locks — liveness check reclaims dead PIDs

### Worktree Structure

```
project/              # Main checkout
project-lane-1/       # Worktree for lane/1
project-lane-2/       # Worktree for lane/2
```

Symlinks for shared resources:
- `node_modules` → avoids duplicate installs
- `.env*` → shares secrets
- Custom paths via config

### Zero Runtime Dependencies

No `node_modules` needed in production. All locking and coordination uses:
- Built-in `fs` module
- Process ID checks
- Git's native worktree support

## Common Commands Quick Reference

```bash
# Initial setup
npx claude-code-merge-queue init

# Daily workflow
claude --worktree feature-x        # Start isolated lane
npm run land                       # Land after tests pass
npm run sync                       # Update main checkout

# Preview without build
npm run preview                    # Mirror lane to main
npm run preview:restore            # Switch back

# Release
npm run promote                    # Ship to production

# Utilities
claude-code-merge-queue port       # Get lane's port number
claude-code-merge-queue prune      # Clean landed lanes
```

## Integration with CI

On integration branch:

```yaml
# .github/workflows/ci.yml
name: CI
on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      - run: npm run check
```

On production branch (if using two-stage):

```yaml
# .github/workflows/deploy.yml
name: Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run build
      - run: ./deploy.sh
```

The queue handles local coordination; CI is your second checkpoint for landed code.
