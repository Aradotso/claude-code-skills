---
name: antigravity-for-claude-code
description: Run Antigravity CLI (Gemini) as a sub-agent in Claude Code for intelligent model routing, bulk work delegation, and cross-model verification.
triggers:
  - delegate this to antigravity
  - use gemini for this bulk task
  - review this with antigravity
  - research this topic with web search
  - run antigravity delegate
  - check cloud run logs
  - use agy for this
  - cross-model verification
---

# Antigravity for Claude Code

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## Overview

Antigravity for Claude Code is a plugin that enables intelligent model routing across the SDLC by delegating bulk work to the Antigravity CLI (agy/Gemini) while keeping Claude as the conductor for judgment, architecture, and verification.

**Core principle**: Claude owns requirements, architecture, and verification; Gemini/agy handles scaffolding, implementation, test generation, and bulk operations at lower cost.

**Measured savings**: −27% to −64% cost reduction on large tasks while maintaining equal quality through cross-model verification.

## Installation

```bash
# In Claude Code
/plugin marketplace add yuting0624/antigravity-for-claude-code
/plugin install antigravity@antigravity-for-claude-code
/antigravity:setup
```

**Prerequisites**:
- [Antigravity CLI](https://antigravity.google/docs/cli-using) installed and authenticated
- Claude Code running
- Verify setup: `agy models` should list available models

**Platform support**: macOS, Linux, WSL (recommended). Native Windows has known issues with headless mode.

## Key Commands

### Slash Commands

```bash
# Health check - verify agy installed and authenticated
/antigravity:setup

# Delegate a task to agy with cost discipline
/antigravity:delegate [--tier flash|pro] <task>

# Cross-model code review
/antigravity:review [--adversarial]

# Deep research with web search and citation verification
/antigravity:research <topic>

# Debug Cloud Run services with log analysis
/antigravity:cloud-run-debug [--service <name>] [--region <region>] [--project <id>] [--since 1h] [--apply]

# Background job management
/antigravity:status [id]
/antigravity:result <id>
/antigravity:cancel <id>
```

### Direct Script Usage

```bash
# One-shot delegation (plain text output)
scripts/agy-delegate.sh --tier flash "Summarize this changelog in 3 bullets"

# Multi-file agentic work with workspace
scripts/agy-delegate.sh --tier pro --dir ./src "List every TODO with file:line"

# Bulk read with digest-only reply (biggest cost savings)
scripts/agy-delegate.sh --digest --dir . "Map the auth flow end to end"

# Write task: auto-apply FILE edits (agy >= 1.1.0)
scripts/agy-delegate.sh --mode accept-edits --dir ./app "Implement feature X per SPEC.md"

# Live web/Google search (requires --yolo in headless)
scripts/agy-delegate.sh --tier pro --yolo "Search latest React 19 features. Include URLs and dates."

# Vertex AI Search over internal data
scripts/agy-delegate.sh --tier pro --yolo "List available Vertex AI Search engines (list_engines)."

# Cross-model review from stdin
cat code-changes.diff | scripts/agy-delegate.sh -

# Background job pattern
ID=$(scripts/agy-job.sh start --tier pro --dir . "Generate comprehensive test suite")
# ... do other work ...
scripts/agy-job.sh result "$ID"
```

## Tier Configuration

| Tier | Model | Use Case |
|------|-------|----------|
| `flash` (default) | Gemini 3.5 Flash (High) | Most bulk work |
| `flash-lo` | Gemini 3.5 Flash (Low) | Trivial tasks, cheapest option |
| `pro` | Gemini 3.1 Pro (High) | Complex reasoning, cross-checks |

Configure tiers via plugin options:
- `default_model`: Override default model
- `tier_flash`: Custom model for flash tier
- `tier_flash_lo`: Custom model for flash-lo tier
- `tier_pro`: Custom model for pro tier

Or use environment variables:
```bash
export CLAUDE_PLUGIN_OPTION_tier_pro="claude-3-5-sonnet-20241022"
```

## Delegation Patterns

### Pattern 1: Bulk Implementation

```bash
# Claude: Design the architecture and write the spec
# Then delegate implementation to agy
/antigravity:delegate --tier flash --dir ./src Implement the UserService class according to DESIGN.md. Include error handling and logging.

# Claude: Review the generated code and verify correctness
```

### Pattern 2: Test Generation

```bash
# Generate comprehensive tests for existing code
scripts/agy-delegate.sh --tier flash --dir ./src "Generate unit tests for all functions in auth.ts. Use Jest. Cover edge cases and error paths."
```

### Pattern 3: Cross-Model Review

```bash
# After making changes, get independent verification
/antigravity:review

# For more critical code, use adversarial mode
/antigravity:review --adversarial
```

### Pattern 4: Research with Citation Verification

```bash
# Claude orchestrates, agy does web search, Claude verifies
/antigravity:research "Best practices for handling webhook retries in distributed systems"

# Result includes verified citations from ≥2 sources
```

### Pattern 5: Bulk Refactoring

```bash
# Delegate large-scale mechanical changes
scripts/agy-delegate.sh --mode accept-edits --dir ./app "Migrate all class components to functional components with hooks. Preserve all functionality."

# Verify changes in git diff before committing
```

## Cost Discipline Best Practices

The plugin's cost savings come from following these patterns:

### 1. Delegate Above Break-Even

```bash
# GOOD: Bulk, parallel, or repetitive work
/antigravity:delegate Generate integration tests for all API endpoints

# BAD: Tiny tasks (overhead > savings)
/antigravity:delegate Fix this typo in line 42
```

### 2. Use Digests for Bulk Reads

```bash
# GOOD: Digest-only output collapses cache_read cost
scripts/agy-delegate.sh --digest --dir ./src "Analyze all database queries and summarize optimization opportunities"

# BAD: Full output causes cache_read bloat
scripts/agy-delegate.sh --dir ./src "List every database query with full context"
```

The wrapper warns when replies exceed `digest_warn_chars` (default 8000).

### 3. Batch Operations

```bash
# GOOD: One large delegation
/antigravity:delegate --dir ./src Analyze all components, identify performance issues, generate optimization recommendations with code examples

# BAD: Multiple round-trips
/antigravity:delegate List all components
/antigravity:delegate Analyze each component for performance
/antigravity:delegate Generate recommendations
```

### 4. Review Diffs, Not Full Files

```bash
# GOOD: Review only what changed
/antigravity:review

# BAD: Re-reading entire codebase
/antigravity:delegate Review all files in ./src for bugs
```

### Measure Session Cost

```bash
# Analyze cost breakdown for a Claude Code session
scripts/measure-session.py <session-id>

# Compare cost for a specific task
scripts/agy-cost-compare.sh "Generate test suite for user module"
```

## Configuration

Plugin options (set in `.claude-plugin/claude.json` or via `/plugin options`):

```json
{
  "default_tier": "flash",
  "default_model": "",
  "tier_flash": "",
  "tier_flash_lo": "",
  "tier_pro": "",
  "timeout_seconds": 120,
  "job_timeout_seconds": 600,
  "digest_warn_chars": 8000,
  "coding_policy": "default",
  "delegation_nudge": true,
  "auto_inject_policy": true
}
```

**Key options**:
- `timeout_seconds`: Sync delegation timeout (default 120)
- `job_timeout_seconds`: Background job timeout (default 600)
- `digest_warn_chars`: Warn when replies exceed this size
- `delegation_nudge`: Proactive suggestions for bulk work
- `auto_inject_policy`: Auto-inject cost-aware routing policy

## Real-World Examples

### Example 1: Migration Task

```bash
# Context: Migrating from REST to GraphQL
/antigravity:delegate --tier pro --dir ./api Review all REST endpoints in routes/ and generate equivalent GraphQL schema definitions with resolvers. Include type definitions and input validation.

# agy generates schema.graphql + resolvers
# Claude reviews for correctness and API contract preservation
```

### Example 2: Documentation Generation

```bash
# Generate comprehensive API docs
scripts/agy-delegate.sh --tier flash --dir ./src "Generate API documentation for all exported functions. Include parameter descriptions, return types, examples, and edge cases."

# Output: Markdown docs ready for review
```

### Example 3: Cloud Run Debugging

```bash
# Service returning 500s in production
/antigravity:cloud-run-debug --service my-api --region us-central1 --since 2h

# agy: Fetches logs, identifies root cause
# Claude: Verifies diagnosis, suggests fix
# --apply flag: Applies fix to new branch
```

### Example 4: Security Audit

```bash
# Cross-model security review
/antigravity:review --adversarial

# Independent Gemini analysis + Claude verification
# Catches issues single-model review might miss
```

### Example 5: Background Long-Running Task

```bash
# Start long task without blocking
ID=$(scripts/agy-job.sh start --tier pro --dir . "Analyze all dependencies, identify outdated packages, security vulnerabilities, and breaking changes in upgrades. Generate migration plan.")

# Continue other work...

# Check status
/antigravity:status $ID

# Retrieve result when ready
/antigravity:result $ID
```

## Write Operations

For tasks that modify files:

```bash
# Preferred: accept-edits mode (agy >= 1.1.0)
# File edits only, no terminal/tool access
scripts/agy-delegate.sh --mode accept-edits --dir ./app "Refactor authentication to use OAuth2"

# When tools needed: --yolo grants full access
scripts/agy-delegate.sh --yolo --dir ./app "Search docs and implement feature X"

# ALWAYS run write tasks on a dedicated branch
git checkout -b agy-refactor
scripts/agy-delegate.sh --mode accept-edits --dir . "Task description"
git diff # Review changes
git commit -m "Applied agy refactor"
```

**Guardrails for writes**:
- Run on dedicated branch/worktree
- Verify files actually changed (agy may write to scratch dir if permissions missing)
- Review diff before merging
- Long write tasks → use background jobs (avoid sync timeout)

## Troubleshooting

### Authentication Issues

```bash
# Verify agy authenticated
agy models

# Should list available models
# If not, re-authenticate:
agy auth login
```

### Timeout Issues

```bash
# Increase timeout for long tasks
export CLAUDE_PLUGIN_OPTION_timeout_seconds=300

# Or use background jobs
scripts/agy-job.sh start --tier pro --dir . "long task"
```

### WSL Performance

```bash
# SLOW: Repo on Windows mount
cd /mnt/c/Users/user/project

# FAST: Repo on WSL filesystem
cd ~/project
```

The wrapper warns about Windows mount performance issues.

### Write Tasks Not Applying

Check:
1. File permissions: agy needs write access
2. agy version: `--mode accept-edits` requires >= 1.1.0
3. Verify files changed: `git status`
4. Review agy log: `~/.config/agy/logs/*.log`

See [docs/TROUBLESHOOTING.md](https://github.com/yuting0624/antigravity-for-claude-code/blob/main/docs/TROUBLESHOOTING.md) for detailed diagnostics.

### Windows Native (Git Bash/MSYS)

Known issue: `agy -p` can hang without ConPTY. Solutions:
1. Use WSL (recommended)
2. Ensure `timeout`/`gtimeout` on PATH for hang guard
3. Wrapper returns TIMEOUT (exit 12) instead of hanging

## Integration with Claude Code Workflow

### Proactive Delegation

With `delegation_nudge` enabled, Claude receives hints for bulk-appropriate tasks:

```
User: "Generate tests for all components"
Claude: [recognizes bulk pattern] → suggests /antigravity:delegate
```

### Session Policy Auto-Injection

`SessionStart` hook automatically injects cost-aware routing policy when `auto_inject_policy: true`:

```yaml
# Automatically available in context:
- Delegate bulk/parallel/repetitive work to agy
- Keep Claude for judgment, architecture, verification
- Use digests for large reads
- Batch operations when possible
```

### Subagent File Writing

The `antigravity-delegate` subagent handles file writes on Gemini side:
- Claude designs what to write
- Subagent executes on Gemini
- Claude verifies result
- Zero tokens spent on file content generation

## Advanced Patterns

### Multi-Stage Pipeline

```bash
# Stage 1: Research (agy)
RESEARCH_ID=$(scripts/agy-job.sh start --tier pro --yolo "Research best practices for rate limiting in Node.js APIs")

# Stage 2: Design (Claude)
# ... review research results, design architecture ...

# Stage 3: Implementation (agy)
/antigravity:delegate --tier flash --dir ./src Implement rate limiting middleware per DESIGN.md

# Stage 4: Review (cross-model)
/antigravity:review

# Stage 5: Test generation (agy)
scripts/agy-delegate.sh --tier flash --dir ./src "Generate rate limiter tests covering edge cases"
```

### Fan-Out Pattern

```bash
# One delegation spawns internal subagents
scripts/agy-delegate.sh --tier pro --dir . "Audit entire codebase: security, performance, maintainability. Use separate subagents for each concern."

# agy spawns subagents via define_subagent
# Each leaves readable trajectory
# Audit with: agy-trace
```

### Cost Measurement Loop

```bash
# Before optimization
BEFORE=$(scripts/measure-session.py session-1 | grep "COST-WEIGHTED" | awk '{print $2}')

# Apply delegation strategy
/antigravity:delegate [task]

# After optimization
AFTER=$(scripts/measure-session.py session-2 | grep "COST-WEIGHTED" | awk '{print $2}')

# Compare
echo "Savings: $((BEFORE - AFTER)) tokens"
```

## Environment Variables

```bash
# Plugin options via env vars
export CLAUDE_PLUGIN_OPTION_default_tier="pro"
export CLAUDE_PLUGIN_OPTION_timeout_seconds=180
export CLAUDE_PLUGIN_OPTION_delegation_nudge=false

# Custom model per tier
export CLAUDE_PLUGIN_OPTION_tier_flash="gemini-3.5-flash-002"
export CLAUDE_PLUGIN_OPTION_tier_pro="claude-3-5-sonnet-20241022"

# Pricing (for cost measurement)
# Edit prices.json for accurate cost estimation
```

## Key Concepts

**Conductor vs Executor**: Claude conducts (judges, designs, verifies), Gemini executes (implements, searches, generates).

**Break-Even Point**: Delegation overhead vs savings. Bulk work above ~500 tokens is typically worth delegating.

**Digest Contract**: `--digest` appends instruction to return summary not dump. Biggest cost lever.

**Cross-Model Verification**: Different model provides independent check. Catches issues single model misses.

**Background Jobs**: Fire-and-collect pattern for interactive sessions. Use sync delegation in one-shot `claude -p`.

**Cost Discipline**: Not automatic—requires following patterns (delegate above break-even, use digests, batch operations, review diffs).

## Links

- [GitHub Repository](https://github.com/yuting0624/antigravity-for-claude-code)
- [Antigravity CLI Docs](https://antigravity.google/docs/cli-using)
- [Troubleshooting Guide](https://github.com/yuting0624/antigravity-for-claude-code/blob/main/docs/TROUBLESHOOTING.md)
- [A/B Test Results](https://github.com/yuting0624/antigravity-for-claude-code/blob/main/docs/AB-RESULTS.md)
