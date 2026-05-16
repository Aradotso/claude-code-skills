---
name: oh-my-claudecode-multi-agent-orchestration
description: Multi-agent orchestration for Claude Code with Team mode, autopilot, deep interview, and CLI workers
triggers:
  - "set up oh-my-claudecode multi-agent system"
  - "orchestrate multiple Claude agents using omc"
  - "run team mode with specialized agents"
  - "use autopilot for autonomous development"
  - "start deep interview for requirements gathering"
  - "spawn codex or gemini CLI workers"
---

# oh-my-claudecode Multi-Agent Orchestration

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

oh-my-claudecode (OMC) is a teams-first multi-agent orchestration framework for Claude Code. It enables parallel execution across specialized agents, autonomous development workflows, and intelligent requirements gathering with zero configuration required.

## Installation

### Plugin Installation (Recommended)

Install via Claude Code marketplace (one command at a time):

```bash
/plugin marketplace add https://github.com/Yeachan-Heo/oh-my-claudecode
/plugin install oh-my-claudecode
```

Then run setup:

```bash
/setup
```

### NPM Installation (CLI Runtime)

```bash
npm i -g oh-my-claude-sisyphus@latest
```

Then run setup:

```bash
omc setup
```

**Plugin Directory Mode**: If using `--plugin-dir`, add `--plugin-dir-mode` to setup:

```bash
omc setup --plugin-dir-mode
# or
export OMC_PLUGIN_ROOT=/path/to/plugins
omc setup
```

## Core Concepts

### Execution Surfaces

OMC operates on two surfaces:

1. **In-session skills** (`/...`) - Run inside Claude Code after plugin install
2. **Terminal CLI** (`omc ...`) - Run from shell after npm install

### Enable Native Teams

Add to `~/.claude/settings.json`:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

## Team Mode (Canonical Orchestration)

Team is the primary multi-agent surface. It runs as a staged pipeline:

```
team-plan → team-prd → team-exec → team-verify → team-fix (loop)
```

### In-Session Team Usage

```bash
# Fix TypeScript errors with 3 executor agents
/team 3:executor "fix all TypeScript errors"

# Review security with 2 reviewer agents
/team 2:reviewer "audit authentication flow"

# General development task
/team 4:developer "implement user profile feature"
```

### CLI Team Workers (tmux)

Spawn real CLI workers in tmux panes:

```bash
# 2 Codex CLI workers for code review
omc team 2:codex "review auth module for security issues"

# 2 Gemini CLI workers for UI design
omc team 2:gemini "redesign UI components for accessibility"

# 1 Claude CLI worker
omc team 1:claude "implement the payment flow"

# Check worker status
omc team status auth-review

# Shutdown workers
omc team shutdown auth-review
```

**Requirements**: Requires `codex`/`gemini` CLIs installed and active tmux session.

## Deep Interview (Requirements Gathering)

Use Socratic questioning to clarify requirements before building:

```bash
/deep-interview "I want to build a task management app"
```

The interview:
- Uses weighted dimensions to measure clarity
- Exposes hidden assumptions
- Generates clear requirements document
- Prevents premature execution

### Deep Interview with Autoresearch

```bash
/deep-interview --autoresearch "build an e-commerce platform"
```

Combines requirements gathering with automated research.

## Autopilot (Autonomous Execution)

Single-agent autonomous development:

```bash
# In-session
/autopilot "build a REST API for managing tasks"

# Natural language shortcut
autopilot: build a REST API for managing tasks
```

Autopilot characteristics:
- Single lead agent (not multi-agent like Team)
- Autonomous decision-making
- Persistent execution until verified complete
- Automatic error recovery

## Multi-Model Advisory (CCG)

Combine Codex and Gemini expertise with Claude synthesis:

```bash
/ccg Review this PR — architecture (Codex) and UI components (Gemini)
```

This routes through `/ask codex` and `/ask gemini`, then Claude synthesizes the combined feedback.

### Direct Provider Queries

```bash
# Ask Codex specifically
/ask codex "review this authentication implementation"

# Ask Gemini specifically
/ask gemini "suggest improvements to this component design"
```

## Configuration

### Settings Location

- Global: `~/.claude/settings.json`
- Project: `.claude/settings.json`

### Key Environment Variables

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1",
    "OMC_PLUGIN_ROOT": "/path/to/plugins",
    "ANTHROPIC_API_KEY": "${ANTHROPIC_API_KEY}",
    "OPENAI_API_KEY": "${OPENAI_API_KEY}",
    "GEMINI_API_KEY": "${GEMINI_API_KEY}"
  }
}
```

## TypeScript/JavaScript Usage Patterns

### Custom Agent Configuration

```typescript
// agents/custom-reviewer.ts
export interface AgentConfig {
  role: string;
  capabilities: string[];
  model: string;
}

export const customReviewer: AgentConfig = {
  role: "security-reviewer",
  capabilities: [
    "security-audit",
    "vulnerability-detection",
    "code-review"
  ],
  model: "claude-sonnet-4"
};
```

### Team Workflow Integration

```typescript
// workflows/deployment-pipeline.ts
import { TeamOrchestrator } from 'oh-my-claude-sisyphus';

async function runDeploymentPipeline() {
  const orchestrator = new TeamOrchestrator({
    agents: 3,
    role: 'executor',
    task: 'run full deployment pipeline'
  });

  await orchestrator.execute();
  
  const results = await orchestrator.verify();
  if (!results.allPassed) {
    await orchestrator.fix(results.failures);
  }
}
```

### Skill Extension Pattern

```typescript
// skills/custom-workflow.ts
export const customWorkflowSkill = {
  name: 'custom-workflow',
  description: 'Custom development workflow',
  
  async execute(context: SkillContext) {
    // 1. Requirements gathering
    await context.runSkill('/deep-interview', {
      prompt: context.userInput
    });
    
    // 2. Team execution
    await context.runSkill('/team', {
      agents: 3,
      role: 'executor',
      task: context.clarifiedRequirements
    });
    
    // 3. Verification
    await context.runSkill('/verify', {
      criteria: context.acceptanceCriteria
    });
  }
};
```

## Common Workflows

### Full Feature Development

```bash
# 1. Clarify requirements
/deep-interview "add user authentication"

# 2. Execute with team
/team 4:developer "implement OAuth2 authentication based on requirements doc"

# 3. Review with specialized agents
/team 2:reviewer "security audit authentication implementation"
```

### Code Review Pipeline

```bash
# Use CLI workers for multi-model review
omc team 1:codex "review code architecture and design patterns"
omc team 1:gemini "review user experience and component structure"

# Or use tri-model advisory
/ccg Complete code review covering architecture, security, and UX
```

### Refactoring Large Codebase

```bash
# Deep interview for strategy
/deep-interview "refactor monolith to microservices"

# Parallel execution with team
/team 5:executor "implement microservices architecture per plan"

# Continuous verification
/team 2:validator "verify service boundaries and API contracts"
```

### Research-Driven Development

```bash
# Start with autoresearch
/deep-interview --autoresearch "build real-time collaborative editor"

# Execute with findings
/autopilot "implement collaborative editor using CRDTs based on research"
```

## CLI Reference

### Setup Commands

```bash
# Basic setup
omc setup

# Setup with plugin directory mode
omc setup --plugin-dir-mode

# Force reinstall
omc setup --force
```

### Team Commands

```bash
# Spawn team workers
omc team <N>:<role> "<task>"

# Worker status
omc team status <task-id>

# Shutdown workers
omc team shutdown <task-id>

# List all active teams
omc team list
```

### Diagnostic Commands

```bash
# Run diagnostics
/omc-doctor

# Check plugin status
/plugin list

# Update marketplace plugin
/plugin marketplace update omc
```

## Troubleshooting

### Teams Not Working

**Issue**: Team commands fail or warn about teams being disabled.

**Solution**: Enable experimental teams in settings:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

### CLI Workers Not Spawning

**Issue**: `omc team N:codex` fails.

**Solutions**:
1. Verify tmux is running: `tmux ls`
2. Check CLI tools installed: `which codex` / `which gemini`
3. Ensure API keys configured in environment

### Plugin Not Found After Install

**Issue**: `/setup` or skills not available.

**Solutions**:

```bash
# Clear plugin cache
/omc-doctor

# Reinstall plugin
/plugin marketplace update omc
/setup
```

### Duplicate Skills/Agents

**Issue**: Skills appear twice or setup duplicates resources.

**Solution**: Use `--plugin-dir-mode` when running from custom plugin directory:

```bash
export OMC_PLUGIN_ROOT=/path/to/plugins
omc setup --plugin-dir-mode
```

### npm Warning: deprecated prebuild-install

**Issue**: Warning during `npm i -g oh-my-claude-sisyphus`.

**Status**: Known upstream dependency warning from `better-sqlite3 → prebuild-install@7.1.3`. Tracked in issue #2913. Does not indicate install failure.

### Package Name Confusion

**Issue**: Repository is `oh-my-claudecode` but npm package is different.

**Clarification**: 
- Repository/plugin: `oh-my-claudecode`
- npm package: `oh-my-claude-sisyphus`

Always use `oh-my-claude-sisyphus` for npm commands:

```bash
npm i -g oh-my-claude-sisyphus@latest
```

## Advanced Patterns

### Combining Multiple Orchestration Modes

```typescript
// Hybrid workflow: deep interview → team → autopilot cleanup
async function hybridWorkflow(task: string) {
  // Phase 1: Requirements
  const requirements = await runDeepInterview(task);
  
  // Phase 2: Core implementation with team
  await runTeam({
    agents: 4,
    role: 'executor',
    task: requirements.implementation
  });
  
  // Phase 3: Polish with autopilot
  await runAutopilot({
    task: 'refine and optimize implementation',
    context: requirements
  });
}
```

### Custom Provider Integration

```typescript
// Add custom provider to advisory workflow
export async function consultProviders(
  question: string,
  providers: string[]
) {
  const responses = await Promise.all(
    providers.map(p => askProvider(p, question))
  );
  
  return synthesizeResponses(responses);
}

// Usage
const feedback = await consultProviders(
  "Review this architecture",
  ["codex", "gemini", "claude"]
);
```

### Persistent Task Management

```typescript
// Track long-running tasks across sessions
interface PersistentTask {
  id: string;
  status: 'planning' | 'executing' | 'verifying' | 'complete';
  checkpoint: any;
}

export async function resumeTask(taskId: string) {
  const task = await loadTask(taskId);
  
  switch (task.status) {
    case 'planning':
      await runTeamPlan(task);
      break;
    case 'executing':
      await runTeamExec(task);
      break;
    case 'verifying':
      await runTeamVerify(task);
      break;
  }
}
```

## Best Practices

1. **Start with Deep Interview** for unclear requirements
2. **Use Team mode** for coordinated multi-agent work
3. **Use CLI workers** (`omc team N:codex/gemini`) for provider-specific tasks
4. **Enable native teams** in settings before using `/team`
5. **Use CCG** for mixed backend/frontend review work
6. **Keep plugin updated** with `/plugin marketplace update omc`
7. **Use `--plugin-dir-mode`** if running with custom plugin directory
8. **Set environment variables** properly for API access
9. **Run `/omc-doctor`** when troubleshooting
10. **Check tmux session** before spawning CLI workers

## Resources

- Homepage: https://oh-my-claudecode.dev
- Documentation: https://yeachan-heo.github.io/oh-my-claudecode-website
- Repository: https://github.com/Yeachan-Heo/oh-my-claudecode
- Discord: https://discord.gg/PUwSMR9XNk
- npm Package: https://www.npmjs.com/package/oh-my-claude-sisyphus
- Migration Guide: https://github.com/Yeachan-Heo/oh-my-claudecode/blob/main/docs/MIGRATION.md
