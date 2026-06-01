---
name: pi-coding-agent-extensions
description: Build and use custom extensions for Pi Coding Agent, the open-source alternative to Claude Code with UI customization, multi-agent orchestration, and safety controls
triggers:
  - customize pi coding agent ui
  - create pi agent extension
  - pi vs claude code comparison
  - multi-agent orchestration with pi
  - pi agent safety auditing
  - pi agent cross-agent integration
  - pi to pi agent communication
---

# Pi Coding Agent Extensions

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

Pi Coding Agent is an open-source agentic coding assistant that competes with Claude Code. This skill focuses on the **pi-vs-claude-code** repository, which provides 15+ production-ready extensions showcasing UI customization, multi-agent orchestration, safety auditing, and agent-to-agent communication.

## What This Project Does

The `pi-vs-claude-code` repository demonstrates how to:

- **Customize the Pi UI**: Replace the default footer/widgets with custom displays (context meters, tool counters, task trackers)
- **Multi-agent orchestration**: Run multiple Pi agents in parallel (teams, chains, subagents)
- **Safety controls**: Intercept dangerous commands with real-time auditing
- **Cross-agent integration**: Import commands/skills from Claude, Gemini, Codex directories
- **Agent-to-agent communication**: Enable Pi agents to message each other (local or networked)
- **Session management**: Replay session history, switch agent personas, cycle themes

## Installation

### Prerequisites

All three required:

```bash
# Install Bun (runtime & package manager)
curl -fsSL https://bun.sh/install | bash

# Install just (task runner)
brew install just  # macOS
# or: cargo install just

# Install Pi Coding Agent CLI
# See: https://github.com/mariozechner/pi-coding-agent
```

### Setup

```bash
# Clone the repository
git clone https://github.com/disler/pi-vs-claude-code.git
cd pi-vs-claude-code

# Install dependencies
bun install

# Configure API keys
cp .env.sample .env
# Edit .env and add your keys:
# OPENAI_API_KEY=sk-...
# ANTHROPIC_API_KEY=sk-ant-...
# GEMINI_API_KEY=...
# OPENROUTER_API_KEY=sk-or-...
```

### Loading Environment Variables

Pi doesn't auto-load `.env`. Choose one approach:

```bash
# Option A: Source manually each session
source .env && pi

# Option B: Add alias to ~/.zshrc or ~/.bashrc
alias pi='source $(pwd)/.env && pi'

# Option C: Use just (auto-loads .env)
just pi
just ext-minimal
```

## Running Extensions

### Single Extension

```bash
pi -e extensions/minimal.ts
```

### Stack Multiple Extensions

```bash
pi -e extensions/minimal.ts -e extensions/cross-agent.ts -e extensions/tool-counter.ts
```

### Using Just Recipes

```bash
# List all recipes
just

# Common recipes
just pi                          # Plain Pi, no extensions
just ext-minimal                 # Compact footer with context meter
just ext-tool-counter            # Rich two-line footer with tool statistics
just ext-purpose-gate            # Session intent prompt on startup
just ext-damage-control          # Safety auditing with command blocking
just ext-agent-team              # Multi-agent dispatcher with grid dashboard
just ext-pi-pi                   # Meta-agent that builds Pi agents
just ext-session-replay          # Scrollable timeline of session history
just all                         # Open every extension in separate terminals

# Open custom stack
just open purpose-gate minimal tool-counter-widget
```

## Key Extensions

### UI Customization

**minimal.ts** — Compact footer with model name and context usage:
```typescript
// Shows: gpt-4o-mini [###-------] 30%
pi -e extensions/minimal.ts
```

**pure-focus.ts** — Removes all footer/status UI:
```bash
just ext-pure-focus
```

**theme-cycler.ts** — Switch themes with keyboard shortcuts (Ctrl+X/Ctrl+Q) or `/theme` command:
```bash
just ext-theme-cycler
# In Pi: /theme monokai
```

### Tool Monitoring

**tool-counter.ts** — Two-line footer with per-tool call statistics:
```bash
just ext-tool-counter
# Line 1: model + context + tokens/cost
# Line 2: cwd/branch + tool tallies (e.g., bash:12 write_file:5)
```

**tool-counter-widget.ts** — Live widget above editor:
```bash
just ext-tool-counter-widget
# Shows real-time tool usage with color-coded backgrounds
```

### Multi-Agent Orchestration

**subagent-widget.ts** — Spawn background Pi agents:
```bash
just ext-subagent-widget

# In Pi session:
/sub implement user authentication with JWT
/sub write unit tests for the API endpoints
```

Each `/sub` command creates a headless Pi agent with streaming progress display.

**agent-team.ts** — Dispatcher-only orchestrator:
```bash
just ext-agent-team

# Define team in .pi/agents/teams.yaml:
# team-name: web-app-team
# agents:
#   - frontend-specialist
#   - backend-specialist
#   - devops-specialist

# Main agent delegates via dispatch_agent tool
```

**agent-chain.ts** — Sequential pipeline:
```bash
just ext-agent-chain

# Define pipeline in .pi/agents/agent-chain.yaml:
# name: full-stack-pipeline
# steps:
#   - agent: research-agent
#     prompt: "Research best practices for {topic}"
#   - agent: implementation-agent
#     prompt: "Implement based on: {previous_output}"
#   - agent: review-agent
#     prompt: "Review and suggest improvements"

# In Pi: /chain
# Select pipeline interactively
```

**pi-pi.ts** — Meta-agent that builds Pi agents using parallel research experts:
```bash
just ext-pi-pi

# Ask: "Create a Pi agent that specializes in React testing"
# Pi-pi spawns documentation experts, synthesizes findings, generates agent
```

### Safety Controls

**damage-control.ts** — Real-time command auditing (aborts on violation):
```bash
just ext-damage-control

# Configure rules in .pi/damage-control-rules.yaml:
# banned_patterns:
#   - pattern: "rm -rf /"
#     reason: "Catastrophic deletion prevented"
#   - pattern: ":(){ :|:& };:"
#     reason: "Fork bomb detected"
# 
# path_restrictions:
#   - pattern: "^/etc/"
#     allowed: false
#     reason: "System directory access denied"
```

**damage-control-continue.ts** — Same rules, but blocked calls return feedback instead of aborting:
```bash
just ext-damage-control-continue
# Agent receives: "Tool call blocked: rm -rf / - Catastrophic deletion prevented"
# Agent can adapt and try a safer approach
```

### Cross-Agent Integration

**cross-agent.ts** — Import commands/skills from other agent directories:
```bash
just ext-cross-agent

# Scans and registers from:
# - .claude/commands/
# - .gemini/skills/
# - .codex/agents/
# All become available in Pi session
```

**system-select.ts** — Switch agent personas:
```bash
just ext-system-select

# In Pi: /system
# Select from .pi/agents/, .claude/agents/, .gemini/agents/, .codex/agents/

# Example agent definition (.pi/agents/rust-expert.md):
# You are a Rust programming expert specializing in systems programming.
# Focus on memory safety, zero-cost abstractions, and idiomatic code.
```

### Session Management

**purpose-gate.ts** — Declare session intent on startup:
```bash
just ext-purpose-gate

# On startup, prompts:
# "What is the purpose of this session?"
# Blocks prompts until answered
# Shows persistent purpose widget
```

**tilldone.ts** — Task discipline system:
```bash
just ext-tilldone

# Define tasks before work starts
# Tracks completion state across steps
# Shows persistent task list in footer with live progress
```

**session-replay.ts** — Scrollable timeline overlay:
```bash
just ext-session-replay

# Press configured hotkey to view session history
# Navigate through past turns with arrow keys
```

## Agent-to-Agent Communication

### Local Communication (Same Machine)

**coms.ts** — Peer-to-peer via Unix sockets:
```bash
just local-coms

# Terminal 1
pi -e extensions/coms.ts
# In Pi: (agent automatically registers as "agent-1")

# Terminal 2
pi -e extensions/coms.ts
# In Pi: (registers as "agent-2")

# Agent 1 can send to Agent 2:
# Use coms_send tool: coms_send("agent-2", "Start the API server")

# Agent 2 receives:
# Use coms_get or coms_await tools
```

Available tools in `coms.ts`:
- `coms_list()` — List all registered agents
- `coms_send(target_agent, message)` — Send message to another agent
- `coms_get(count)` — Retrieve pending messages (non-blocking)
- `coms_await(timeout_seconds)` — Wait for next message (blocking)

### Networked Communication (Multi-Machine)

**coms-net.ts** — HTTP/SSE hub for LAN or remote Pi agents:

```bash
# Start hub server (local only)
just coms-net-server
# Runs on http://127.0.0.1:3141

# Start hub server (LAN-visible)
export PI_COMS_NET_AUTH_TOKEN="your-secret-token"
just coms-net-server-lan
# Runs on http://0.0.0.0:3141

# Connect client agents
PI_COMS_NET_URL=http://192.168.1.100:3141 \
PI_COMS_NET_AUTH_TOKEN="your-secret-token" \
just coms

# Or on different machines:
# Machine A:
just coms1  # Uses gpt-4o

# Machine B:
just coms2  # Uses claude-opus-4
```

Available tools in `coms-net.ts`:
- `coms_net_list()` — List all connected agents
- `coms_net_send(target_agent, message)` — Send to specific agent
- `coms_net_broadcast(message)` — Send to all agents
- `coms_net_get(count)` — Poll for messages
- `coms_net_await(timeout_seconds)` — Wait for next message

## Creating Custom Extensions

### Basic Extension Template

```typescript
// extensions/my-custom.ts
import type { PiEnv, PiExtension } from "@pi/core";

export default {
  name: "my-custom",
  
  init(env: PiEnv) {
    // Setup: register tools, hooks, widgets
    
    env.addTool({
      name: "my_tool",
      description: "Does something useful",
      parameters: {
        type: "object",
        properties: {
          param1: { type: "string", description: "First parameter" }
        },
        required: ["param1"]
      },
      handler: async (args) => {
        // Tool implementation
        return { result: "success" };
      }
    });
  },
  
  cleanup(env: PiEnv) {
    // Teardown logic
  }
} satisfies PiExtension;
```

### UI Widget Example

```typescript
// extensions/custom-widget.ts
import type { PiEnv, PiExtension } from "@pi/core";

export default {
  name: "custom-widget",
  
  init(env: PiEnv) {
    const widgetId = env.ui.addWidget({
      position: "above-editor",
      height: 3,
      render: () => {
        return [
          { text: "╔═══ Custom Widget ═══╗", style: { fg: "cyan" } },
          { text: "║ Status: Active      ║", style: { fg: "green" } },
          { text: "╚═════════════════════╝", style: { fg: "cyan" } }
        ];
      }
    });
    
    // Update widget every 5 seconds
    const interval = setInterval(() => {
      env.ui.updateWidget(widgetId);
    }, 5000);
    
    env.onCleanup(() => clearInterval(interval));
  }
} satisfies PiExtension;
```

### Footer Customization

```typescript
// extensions/custom-footer.ts
import type { PiEnv, PiExtension } from "@pi/core";

export default {
  name: "custom-footer",
  
  init(env: PiEnv) {
    env.ui.setFooter({
      render: (context) => {
        const model = context.currentModel || "unknown";
        const usage = Math.round((context.tokensUsed / context.tokenLimit) * 100);
        
        return [
          { text: `Model: ${model} | Usage: ${usage}%`, style: { fg: "white", bg: "blue" } }
        ];
      }
    });
  }
} satisfies PiExtension;
```

### Intercepting Tool Calls

```typescript
// extensions/tool-interceptor.ts
import type { PiEnv, PiExtension } from "@pi/core";

export default {
  name: "tool-interceptor",
  
  init(env: PiEnv) {
    env.onBeforeToolCall(async (toolName, args) => {
      console.log(`Tool called: ${toolName}`, args);
      
      // Block dangerous operations
      if (toolName === "bash" && args.command?.includes("rm -rf")) {
        return {
          blocked: true,
          reason: "Destructive command blocked for safety"
        };
      }
      
      // Allow by default
      return { blocked: false };
    });
    
    env.onAfterToolCall(async (toolName, args, result) => {
      console.log(`Tool completed: ${toolName}`, result);
    });
  }
} satisfies PiExtension;
```

## Configuration Files

### Agent Definitions

Create agent personas in `.pi/agents/`:

```markdown
<!-- .pi/agents/typescript-expert.md -->
You are a TypeScript expert specializing in type-safe development.

Core principles:
- Prefer strict type checking
- Use discriminated unions over enums
- Leverage const assertions
- Avoid `any` type

When reviewing code:
1. Check type safety
2. Identify potential runtime errors
3. Suggest idiomatic patterns
```

### Team Configuration

```yaml
# .pi/agents/teams.yaml
team-name: fullstack-team
agents:
  - frontend-specialist
  - backend-specialist
  - database-specialist
  - devops-specialist

dispatch-strategy: auto  # or: manual, round-robin
```

### Chain Pipeline

```yaml
# .pi/agents/agent-chain.yaml
name: feature-development
steps:
  - agent: requirements-analyst
    prompt: "Analyze requirements for: {task}"
    
  - agent: architect
    prompt: "Design architecture based on: {previous_output}"
    
  - agent: implementer
    prompt: "Implement the design: {previous_output}"
    
  - agent: tester
    prompt: "Write tests for: {previous_output}"
    
  - agent: reviewer
    prompt: "Review and suggest improvements: {previous_output}"
```

### Safety Rules

```yaml
# .pi/damage-control-rules.yaml
banned_patterns:
  - pattern: "rm -rf /"
    reason: "Catastrophic deletion prevented"
    
  - pattern: ":(){ :|:& };:"
    reason: "Fork bomb detected"
    
  - pattern: "chmod 777"
    reason: "Insecure permissions"
    
  - pattern: "curl .* | bash"
    reason: "Unsafe remote script execution"

path_restrictions:
  - pattern: "^/etc/"
    allowed: false
    reason: "System directory modification denied"
    
  - pattern: "^/var/log/"
    allowed: false
    reason: "Log tampering prevented"
    
  - pattern: "^~/.ssh/"
    allowed: false
    reason: "SSH key modification blocked"

allowed_commands:
  - git
  - npm
  - bun
  - node
  - python
  - cargo
```

### Custom Themes

```json
// .pi/themes/monokai.json
{
  "name": "monokai",
  "colors": {
    "primary": "#F92672",
    "secondary": "#66D9EF",
    "success": "#A6E22E",
    "warning": "#FD971F",
    "error": "#F92672",
    "info": "#66D9EF",
    "muted": "#75715E",
    "background": "#272822",
    "foreground": "#F8F8F2"
  }
}
```

## Common Patterns

### Multi-Agent Research Pattern

```bash
# Start pi-pi meta-agent
just ext-pi-pi

# Prompt:
"Research the best approach for implementing real-time collaboration in a React app"

# Pi-pi will:
# 1. Spawn parallel documentation experts
# 2. Each expert researches different aspects (WebSockets, CRDTs, state sync)
# 3. Synthesize findings into comprehensive analysis
# 4. Optionally generate a specialist agent for implementation
```

### Sequential Pipeline Pattern

```bash
# Define pipeline in .pi/agents/agent-chain.yaml
# Start chain extension
just ext-agent-chain

# In Pi: /chain
# Select "feature-development" pipeline
# Provide initial task description

# Pipeline executes:
# requirements-analyst → architect → implementer → tester → reviewer
# Each step feeds into the next
```

### Safety-First Development Pattern

```bash
# Start with damage control
just ext-damage-control-continue

# Configure strict rules in .pi/damage-control-rules.yaml
# Agent receives feedback on blocked operations
# Agent adapts to safer alternatives

# Example interaction:
# Agent tries: bash("rm -rf node_modules/")
# System blocks but continues: "Use a safer pattern: trash or specific file deletion"
# Agent adapts: bash("find node_modules -maxdepth 1 -type d -exec rm -rf {} +")
```

### Cross-Team Collaboration Pattern

```bash
# Terminal 1: Frontend team
PI_COMS_NET_URL=http://hub.local:3141 \
PI_COMS_NET_AUTH_TOKEN=$AUTH_TOKEN \
pi -e extensions/coms-net.ts -e extensions/agent-team.ts

# Terminal 2: Backend team
PI_COMS_NET_URL=http://hub.local:3141 \
PI_COMS_NET_AUTH_TOKEN=$AUTH_TOKEN \
pi -e extensions/coms-net.ts -e extensions/agent-team.ts

# Teams can exchange messages:
# Frontend: coms_net_send("backend-team", "API spec ready for review")
# Backend: coms_net_await(60)  # Wait for message
```

## Environment Variables

```bash
# API Keys (required)
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
GEMINI_API_KEY=...
OPENROUTER_API_KEY=sk-or-...

# Coms-Net Configuration (optional)
PI_COMS_NET_URL=http://127.0.0.1:3141
PI_COMS_NET_AUTH_TOKEN=your-secret-token

# Pi Configuration (optional)
PI_MODEL=gpt-4o-mini
PI_WORKSPACE=.pi
```

## Troubleshooting

### Extensions Not Loading

```bash
# Verify extension file exists
ls -la extensions/minimal.ts

# Check for syntax errors
bun run extensions/minimal.ts

# Ensure dependencies installed
bun install

# Try absolute path
pi -e $(pwd)/extensions/minimal.ts
```

### API Keys Not Recognized

```bash
# Verify .env file exists
cat .env

# Check environment is loaded
echo $OPENAI_API_KEY

# Use just (auto-loads .env)
just ext-minimal

# Or source manually
source .env && pi -e extensions/minimal.ts
```

### Coms-Net Connection Failures

```bash
# Verify server is running
curl http://127.0.0.1:3141/health

# Check auth token matches
echo $PI_COMS_NET_AUTH_TOKEN

# Test with explicit config
PI_COMS_NET_URL=http://127.0.0.1:3141 \
PI_COMS_NET_AUTH_TOKEN=test-token \
pi -e extensions/coms-net.ts

# View server logs
just coms-net-server
# (Look for connection/auth errors)
```

### Agent-Team Dispatch Failures

```bash
# Verify teams.yaml exists
cat .pi/agents/teams.yaml

# Check agent definitions exist
ls -la .pi/agents/

# Validate YAML syntax
bun run -e "console.log(require('yaml').parse(require('fs').readFileSync('.pi/agents/teams.yaml', 'utf8')))"

# Test with single agent first
# Create .pi/agents/test-agent.md
# Then try dispatch_agent("test-agent", "simple task")
```

### Tool Counter Not Updating

```bash
# Ensure extension is loaded
pi -e extensions/tool-counter.ts

# Verify tools are being called
# (Counter only updates when agent uses tools)

# Check for UI conflicts
# Don't mix tool-counter.ts with tool-counter-widget.ts

# Try minimal test
just ext-tool-counter
# Then: "List files in current directory"
# Should show bash:1 in footer
```

### Damage Control Too Restrictive

```bash
# Review rules
cat .pi/damage-control-rules.yaml

# Use continue variant for feedback
just ext-damage-control-continue

# Temporarily disable specific rules
# Edit .pi/damage-control-rules.yaml
# Comment out overly restrictive patterns

# Test rule matching
# Add debug logging in damage-control.ts
```

## Additional Resources

- **Pi Coding Agent**: https://github.com/mariozechner/pi-coding-agent
- **Pi Documentation**: https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent/docs
- **Video Tutorials**:
  - [Pi Coding Agent: The Only Claude Code Competitor](https://youtu.be/f8cfH5XX-XU)
  - [Pi to Pi: Two-Way Agent Orchestration](https://youtu.be/PIdETjcXNIk)
- **Provider Configuration**: https://github.com/badlogic/pi-mono/blob/main/packages/coding-agent/docs/providers.md
