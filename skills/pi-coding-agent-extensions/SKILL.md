---
name: pi-coding-agent-extensions
description: Build and use Pi Coding Agent extensions to customize UI, orchestration, safety, and multi-agent workflows
triggers:
  - customize pi coding agent
  - create pi extension
  - pi agent orchestration
  - multi-agent pi workflow
  - pi coding agent safety rules
  - build custom pi ui
  - pi to pi communication
  - pi agent theme customization
---

# Pi Coding Agent Extensions

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

Pi Coding Agent is an open-source agentic coding assistant that supports deep customization through TypeScript extensions. This skill covers building extensions for UI customization, multi-agent orchestration, safety auditing, and agent-to-agent communication.

## What This Project Does

`pi-vs-cc` is a collection of production-ready Pi Coding Agent extensions demonstrating:
- Custom UI widgets and themes
- Multi-agent orchestration patterns (subagents, teams, chains)
- Real-time safety auditing and access control
- Cross-agent command loading from Claude/Gemini/Codex directories
- Pi-to-Pi communication over Unix sockets or HTTP/SSE

## Prerequisites

Install all three requirements:

```bash
# Install Bun (required runtime)
curl -fsSL https://bun.sh/install | bash

# Install just task runner
brew install just

# Install Pi Coding Agent
# Follow instructions at: https://github.com/mariozechner/pi-coding-agent
```

## Installation

```bash
git clone https://github.com/disler/pi-vs-claude-code.git
cd pi-vs-claude-code
bun install
```

## API Key Configuration

Pi requires environment variables to be set **before** launching. Create `.env` from template:

```bash
cp .env.sample .env
```

Edit `.env` and add your keys:

```bash
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
GEMINI_API_KEY=AI...
OPENROUTER_API_KEY=sk-or-...
```

Source environment before running Pi:

```bash
source .env && pi
```

Or use `just` recipes which auto-load `.env`:

```bash
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
pi -e extensions/minimal.ts -e extensions/cross-agent.ts
```

### Using Just Recipes

List all available recipes:

```bash
just
```

Common recipes:

```bash
just ext-minimal              # Compact footer with context meter
just ext-tool-counter         # Rich stats footer
just ext-subagent-widget      # Background subagent spawner
just ext-agent-team           # Multi-agent orchestration
just ext-damage-control       # Safety auditing
just ext-coms-net            # Networked Pi-to-Pi communication
```

## Extension Architecture

Extensions are TypeScript modules that export an `extension` object implementing the `PiExtension` interface:

```typescript
import type { PiExtension } from "pi-coding-agent";

export const extension: PiExtension = {
  name: "my-extension",
  
  setup(pi) {
    // Called once on startup
    // Register tools, commands, widgets
  },
  
  onTurnStart(turn) {
    // Called before agent processes user input
  },
  
  onToolCall(call) {
    // Intercept/modify tool calls
    return call; // or return modified call
  },
  
  onToolResult(result) {
    // Process tool execution results
    return result;
  },
  
  onTurnEnd(turn) {
    // Called after agent completes turn
  },
  
  footer() {
    // Return custom footer content
    return "Custom Status";
  }
};
```

## Key Extension Patterns

### Custom Footer UI

```typescript
import type { PiExtension } from "pi-coding-agent";
import { terminal } from "pi-coding-agent";

export const extension: PiExtension = {
  name: "custom-footer",
  
  footer(pi) {
    const modelName = pi.model?.name || "unknown";
    const usage = pi.context.usage;
    const pct = Math.round((usage.current / usage.total) * 100);
    
    return terminal.green(`${modelName} `) + 
           terminal.dim(`[${pct}%]`);
  }
};
```

### Registering Custom Tools

```typescript
import type { PiExtension } from "pi-coding-agent";

export const extension: PiExtension = {
  name: "custom-tools",
  
  setup(pi) {
    pi.registerTool({
      name: "analyze_project",
      description: "Analyze project structure and dependencies",
      parameters: {
        type: "object",
        properties: {
          path: { type: "string", description: "Project path" }
        },
        required: ["path"]
      },
      async execute(params) {
        const { path } = params;
        // Implementation
        return { status: "analyzed", files: 42 };
      }
    });
  }
};
```

### Creating Widgets

```typescript
import type { PiExtension } from "pi-coding-agent";
import { terminal } from "pi-coding-agent";

export const extension: PiExtension = {
  name: "status-widget",
  
  setup(pi) {
    let taskCount = 0;
    
    const widget = pi.createWidget({
      position: "above-editor",
      render() {
        return terminal.bgBlue(` Tasks: ${taskCount} `);
      }
    });
    
    pi.registerCommand("task", async (args) => {
      taskCount++;
      widget.update();
      return "Task registered";
    });
  }
};
```

### Safety Auditing

```typescript
import type { PiExtension, ToolCall } from "pi-coding-agent";
import { terminal } from "pi-coding-agent";

const DANGEROUS_PATTERNS = [
  /rm\s+-rf\s+\//,
  /sudo\s+rm/,
  /:\s*\(\s*\)\s*\{\s*:\s*\|\s*:/  // fork bomb
];

export const extension: PiExtension = {
  name: "safety-audit",
  
  onToolCall(call: ToolCall) {
    if (call.name === "bash_command") {
      const command = call.params.command;
      
      for (const pattern of DANGEROUS_PATTERNS) {
        if (pattern.test(command)) {
          console.error(terminal.red(
            `🚨 BLOCKED: Dangerous command detected: ${command}`
          ));
          throw new Error("Safety violation");
        }
      }
    }
    
    return call;
  }
};
```

### Multi-Agent Orchestration

```typescript
import type { PiExtension } from "pi-coding-agent";
import { spawn } from "child_process";

export const extension: PiExtension = {
  name: "subagent-spawner",
  
  setup(pi) {
    pi.registerCommand("sub", async (args) => {
      const task = args.join(" ");
      
      const subagent = spawn("pi", [
        "--model", "gpt-4o",
        "--prompt", task,
        "--headless"
      ]);
      
      const widget = pi.createWidget({
        position: "above-editor",
        render() {
          return `🤖 Subagent: ${task}`;
        }
      });
      
      subagent.stdout.on("data", (data) => {
        widget.update();
      });
      
      return "Subagent spawned";
    });
  }
};
```

## Pi-to-Pi Communication

### Local Communication (Unix Sockets)

Use the `coms` extension for same-machine agent communication:

```bash
# Terminal 1: Start coordinator agent
just local-coms

# Terminal 2: Start worker agent
just local-coms
```

In agent 1:

```
/send agent-2 "analyze the database schema"
```

In agent 2:

```
/get
```

### Network Communication (HTTP/SSE)

Start a communication hub server:

```bash
# Local only
just coms-net-server

# LAN-accessible (requires PI_COMS_NET_AUTH_TOKEN in .env)
just coms-net-server-lan
```

Connect agents:

```bash
# Terminal 1
just coms1

# Terminal 2
just coms2
```

Tools available in networked mode:

```typescript
// List all connected agents
coms_net_list()

// Send message to agent
coms_net_send({
  to: "agent-id",
  message: "Task description",
  priority: "high"
})

// Get pending messages
coms_net_get({
  filter_sender: "specific-agent-id"  // optional
})

// Wait for specific message
coms_net_await({
  from: "agent-id",
  timeout: 30000
})
```

## Agent Team Orchestration

Create team definition in `.pi/agents/teams.yaml`:

```yaml
teams:
  - name: fullstack-team
    dispatcher_only: true
    agents:
      - name: frontend-specialist
        system_prompt: "Expert in React, TypeScript, Tailwind CSS"
        model: gpt-4o
      
      - name: backend-specialist
        system_prompt: "Expert in Node.js, PostgreSQL, REST APIs"
        model: claude-opus-4
      
      - name: devops-specialist
        system_prompt: "Expert in Docker, CI/CD, cloud deployment"
        model: gemini-2.0-flash-exp
```

Run team orchestrator:

```bash
just ext-agent-team
```

The dispatcher agent has access to `dispatch_agent(agent_name, task)` tool.

## Agent Chain Pipelines

Create chain definition in `.pi/agents/agent-chain.yaml`:

```yaml
chains:
  - name: feature-pipeline
    steps:
      - agent: analyst
        prompt_template: "Analyze requirements: {input}"
      
      - agent: architect
        prompt_template: "Design system for: {previous_output}"
      
      - agent: implementer
        prompt_template: "Implement: {previous_output}"
      
      - agent: tester
        prompt_template: "Test and validate: {previous_output}"
```

Run chain:

```bash
just ext-agent-chain
# Use /chain command to select and run pipeline
```

## Cross-Agent Integration

Load commands from other agent frameworks:

```bash
just ext-cross-agent
```

This scans:
- `.claude/` - Claude Code commands/skills
- `.gemini/` - Gemini commands
- `.codex/` - Codex commands

Commands become available with their original prefixes in Pi.

## Custom Themes

Create theme in `.pi/themes/custom.json`:

```json
{
  "name": "Custom Theme",
  "colors": {
    "primary": "#00ff00",
    "secondary": "#0088ff",
    "success": "#00ff88",
    "warning": "#ffaa00",
    "error": "#ff0044",
    "dim": "#888888"
  }
}
```

Use theme cycler:

```bash
just ext-theme-cycler
# Press Ctrl+X to cycle themes
# Use /theme command to select specific theme
```

## Damage Control Rules

Define safety rules in `.pi/damage-control-rules.yaml`:

```yaml
blocked_commands:
  - pattern: "rm -rf /"
    reason: "Dangerous recursive delete"
  
  - pattern: "sudo rm"
    reason: "Elevated permissions delete"
  
  - pattern: "> /dev/sda"
    reason: "Direct disk write"

allowed_paths:
  - "/workspace"
  - "/tmp"
  - "$HOME/projects"

blocked_paths:
  - "/etc"
  - "/var"
  - "/usr"
  - "$HOME/.ssh"
```

Run with safety:

```bash
just ext-damage-control
```

## Task Discipline System

The `tilldone` extension enforces structured workflows:

```bash
just ext-tilldone
```

Define tasks before starting:

```
/task Setup authentication system
/task Implement user registration
/task Add password reset flow
```

System tracks completion and shows progress in footer.

## Session Replay

View scrollable history of entire session:

```bash
just ext-session-replay
# Press Ctrl+H to toggle timeline overlay
```

## Troubleshooting

### Extensions Not Loading

Verify extension exports `extension` object:

```typescript
export const extension: PiExtension = { /* ... */ };
```

### Environment Variables Not Working

Ensure `.env` is sourced before running Pi:

```bash
source .env && pi -e extensions/my-extension.ts
```

Or use `just` which auto-loads `.env`.

### Tool Registration Fails

Check tool schema matches expected format:

```typescript
pi.registerTool({
  name: "valid_tool_name",  // no spaces, kebab-case
  description: "Clear description",
  parameters: {
    type: "object",
    properties: { /* ... */ },
    required: []
  },
  async execute(params) { /* ... */ }
});
```

### Widget Not Updating

Call `widget.update()` after state changes:

```typescript
const widget = pi.createWidget({
  position: "above-editor",
  render() { return content; }
});

// After updating state:
widget.update();
```

### Subagent Communication Timeout

For `coms-net`, ensure server is running and `PI_COMS_NET_URL` is set:

```bash
# In .env
PI_COMS_NET_URL=http://localhost:3456
PI_COMS_NET_AUTH_TOKEN=your-secret-token

# Start server first
just coms-net-server

# Then connect agents
just coms
```

### Safety Rules Too Strict

Adjust `.pi/damage-control-rules.yaml` or use `damage-control-continue` extension which provides feedback instead of blocking:

```bash
just ext-damage-control-continue
```

## Common Patterns

### Persistent State Across Turns

```typescript
let state = { count: 0 };

export const extension: PiExtension = {
  name: "stateful",
  
  onTurnEnd() {
    state.count++;
  },
  
  footer() {
    return `Turns: ${state.count}`;
  }
};
```

### Async Tool with Progress

```typescript
pi.registerTool({
  name: "long_task",
  description: "Execute long-running task",
  parameters: { /* ... */ },
  async execute(params) {
    const widget = pi.createWidget({
      position: "above-editor",
      render() { return "Processing..." }
    });
    
    for (let i = 0; i < 10; i++) {
      await new Promise(r => setTimeout(r, 1000));
      widget.update();
    }
    
    widget.remove();
    return { status: "complete" };
  }
});
```

### Conditional Tool Interception

```typescript
onToolCall(call) {
  if (call.name === "bash_command" && 
      call.params.command.includes("deploy")) {
    // Require confirmation for deployments
    console.log("⚠️  Deployment detected - require review");
  }
  return call;
}
```

## Reference

- [Pi Coding Agent Documentation](https://github.com/mariozechner/pi-coding-agent)
- [Extension API Types](https://github.com/badlogic/pi-mono/blob/main/packages/coding-agent/src/extension.ts)
- [Built-in Tools Reference](./TOOLS.md)
- [Theme Color Tokens](./THEME.md)
- [Video Tutorials](https://youtu.be/f8cfH5XX-XU)
