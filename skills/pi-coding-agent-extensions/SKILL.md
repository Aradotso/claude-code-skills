---
name: pi-coding-agent-extensions
description: Build and use custom extensions for Pi Coding Agent, the open-source alternative to Claude Code
triggers:
  - "how do I extend pi coding agent"
  - "create a custom pi extension"
  - "add a widget to pi agent"
  - "build a pi coding agent extension"
  - "customize pi agent ui"
  - "make a pi agent tool"
  - "orchestrate multiple pi agents"
  - "pi agent communication between agents"
---

# Pi Coding Agent Extensions

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## Overview

Pi Coding Agent is an open-source terminal-based AI coding assistant similar to Claude Code. This project (`pi-vs-claude-code`) provides a collection of production-ready extensions that showcase Pi's customization capabilities: custom UI widgets, multi-agent orchestration, safety auditing, cross-agent integrations, and inter-agent communication.

Extensions are TypeScript files that hook into Pi's lifecycle events to add tools, modify UI, intercept commands, and coordinate with other agents.

## Prerequisites

All three are required:

```bash
# Install Bun (runtime & package manager)
curl -fsSL https://bun.sh/install | bash

# Install just (task runner)
brew install just  # macOS
# or cargo install just  # cross-platform

# Install Pi Coding Agent CLI
# See https://github.com/mariozechner/pi-coding-agent
```

## Installation

```bash
# Clone the extensions repository
git clone https://github.com/disler/pi-vs-claude-code.git
cd pi-vs-claude-code

# Install dependencies
bun install

# Copy environment template
cp .env.sample .env
# Edit .env and add your API keys:
# OPENAI_API_KEY=sk-...
# ANTHROPIC_API_KEY=sk-ant-...
# GEMINI_API_KEY=...
# OPENROUTER_API_KEY=...
```

## API Key Setup

Pi does NOT auto-load `.env` files. You must source keys before launching:

**Option 1 - Manual source:**
```bash
source .env && pi
```

**Option 2 - Shell alias (add to `~/.zshrc`):**
```bash
alias pi='source $(pwd)/.env && pi'
```

**Option 3 - Use just (automatic):**
```bash
just pi  # .env is loaded automatically
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

```bash
# List all available recipes
just

# Common recipes
just ext-minimal              # Compact footer with context meter
just ext-tool-counter         # Rich footer with tool usage stats
just ext-subagent-widget      # Spawn background agents with live widgets
just ext-agent-team           # Multi-agent orchestration dashboard
just ext-damage-control       # Safety auditing with path controls
just ext-pi-pi                # Meta-agent that builds Pi agents
just coms-net-server          # Start agent-to-agent communication hub
just coms                     # Connect to coms hub
just all                      # Open every extension in separate terminals
```

## Extension Architecture

Extensions export a `PiExtension` object with lifecycle hooks:

```typescript
import type { PiExtension, PiApp } from "pi";

export default {
  name: "my-extension",
  version: "1.0.0",

  // Called when extension loads
  async init(app: PiApp) {
    // Register tools, widgets, commands
  },

  // Called before agent processes a turn
  async beforeTurn(app: PiApp, turn: Turn) {
    // Intercept or modify the turn
  },

  // Called after agent completes a turn
  async afterTurn(app: PiApp, turn: Turn) {
    // Log, analyze, or trigger follow-up actions
  },

  // Called on shutdown
  async destroy(app: PiApp) {
    // Cleanup resources
  }
} satisfies PiExtension;
```

## Key Extension Examples

### Minimal Footer

Compact UI showing model name and context usage:

```typescript
import type { PiExtension, PiApp } from "pi";
import { Widget } from "pi/ui";

export default {
  name: "minimal",
  version: "1.0.0",

  async init(app: PiApp) {
    const footer = new Widget({
      position: "footer",
      render: () => {
        const model = app.session.model.name;
        const usage = app.session.contextUsage;
        const pct = Math.round(usage * 100);
        const filled = Math.round(usage * 10);
        const bar = "█".repeat(filled) + "░".repeat(10 - filled);
        
        return `${model} [${bar}] ${pct}%`;
      }
    });
    
    app.ui.addWidget(footer);
  }
} satisfies PiExtension;
```

### Custom Tool Registration

Add a new tool the agent can call:

```typescript
import type { PiExtension, PiApp } from "pi";

export default {
  name: "custom-tool",
  version: "1.0.0",

  async init(app: PiApp) {
    app.registerTool({
      name: "analyze_code_quality",
      description: "Analyze code quality metrics for a file",
      parameters: {
        type: "object",
        properties: {
          file_path: {
            type: "string",
            description: "Path to the file to analyze"
          }
        },
        required: ["file_path"]
      },
      handler: async (args: { file_path: string }) => {
        const content = await app.fs.readFile(args.file_path);
        // Run analysis logic
        return {
          lines: content.split("\n").length,
          complexity: calculateComplexity(content),
          issues: findIssues(content)
        };
      }
    });
  }
} satisfies PiExtension;

function calculateComplexity(code: string): number {
  // Simplified cyclomatic complexity
  const branches = (code.match(/if|while|for|case/g) || []).length;
  return branches + 1;
}

function findIssues(code: string): string[] {
  const issues: string[] = [];
  if (code.includes("eval(")) issues.push("Dangerous eval() usage");
  if (code.includes("TODO")) issues.push("Unfinished TODO items");
  return issues;
}
```

### Live Widget Above Editor

Create a persistent widget that updates in real-time:

```typescript
import type { PiExtension, PiApp, Turn } from "pi";
import { Widget } from "pi/ui";

const toolCounts = new Map<string, number>();

export default {
  name: "tool-counter-widget",
  version: "1.0.0",

  async init(app: PiApp) {
    const widget = new Widget({
      position: "above-editor",
      height: 3,
      render: () => {
        const entries = Array.from(toolCounts.entries())
          .sort((a, b) => b[1] - a[1])
          .slice(0, 5);
        
        return entries
          .map(([tool, count]) => `${tool}: ${count}`)
          .join(" | ") || "No tools used yet";
      }
    });
    
    app.ui.addWidget(widget);
  },

  async afterTurn(app: PiApp, turn: Turn) {
    // Count tool calls from this turn
    for (const call of turn.toolCalls || []) {
      toolCounts.set(call.name, (toolCounts.get(call.name) || 0) + 1);
    }
    app.ui.refresh();
  }
} satisfies PiExtension;
```

### Safety Auditing (Damage Control)

Intercept and block dangerous commands before execution:

```typescript
import type { PiExtension, PiApp, Turn, ToolCall } from "pi";
import { readFileSync } from "fs";
import { parse } from "yaml";

interface DamageControlRules {
  blocked_patterns: string[];
  allowed_paths: string[];
}

export default {
  name: "damage-control",
  version: "1.0.0",

  async beforeTurn(app: PiApp, turn: Turn) {
    const rules: DamageControlRules = parse(
      readFileSync(".pi/damage-control-rules.yaml", "utf-8")
    );

    for (const call of turn.toolCalls || []) {
      if (call.name === "bash") {
        const cmd = call.arguments.command;
        
        // Check blocked patterns
        for (const pattern of rules.blocked_patterns) {
          if (cmd.includes(pattern)) {
            throw new Error(
              `🚨 BLOCKED: Command contains dangerous pattern "${pattern}"`
            );
          }
        }
        
        // Check path restrictions for file operations
        if (cmd.match(/rm|mv|cp|>|>>/) && call.arguments.path) {
          const isAllowed = rules.allowed_paths.some(
            allowed => call.arguments.path.startsWith(allowed)
          );
          if (!isAllowed) {
            throw new Error(
              `🚨 BLOCKED: Path "${call.arguments.path}" is outside allowed directories`
            );
          }
        }
      }
    }
  }
} satisfies PiExtension;
```

Example `.pi/damage-control-rules.yaml`:

```yaml
blocked_patterns:
  - "rm -rf /"
  - "dd if="
  - ":(){ :|:& };:"
  - "curl | bash"
  - "wget | sh"

allowed_paths:
  - "/tmp"
  - "./src"
  - "./tests"
  - "./extensions"
```

## Multi-Agent Orchestration

### Subagent Spawning (`/sub`)

Offload tasks to background Pi agents:

```typescript
import type { PiExtension, PiApp } from "pi";
import { Widget } from "pi/ui";
import { spawn } from "child_process";

const subagents = new Map<string, any>();

export default {
  name: "subagent-widget",
  version: "1.0.0",

  async init(app: PiApp) {
    // Register /sub command
    app.registerCommand({
      name: "sub",
      description: "Spawn a background Pi agent for a task",
      handler: async (args: string[]) => {
        const task = args.join(" ");
        const id = `sub-${Date.now()}`;
        
        const widget = new Widget({
          position: "above-editor",
          height: 4,
          render: () => `🤖 ${id}\n${subagents.get(id)?.status || "Starting..."}`
        });
        
        app.ui.addWidget(widget);
        
        // Spawn Pi subprocess
        const proc = spawn("pi", ["-p", task], {
          env: process.env,
          stdio: ["ignore", "pipe", "pipe"]
        });
        
        subagents.set(id, { proc, status: "Running...", widget });
        
        proc.stdout.on("data", (data) => {
          subagents.get(id).status = data.toString().slice(-200);
          app.ui.refresh();
        });
        
        proc.on("close", (code) => {
          subagents.get(id).status = `✓ Complete (exit ${code})`;
          app.ui.refresh();
          setTimeout(() => {
            app.ui.removeWidget(widget);
            subagents.delete(id);
          }, 3000);
        });
      }
    });
  }
} satisfies PiExtension;
```

Usage:
```
/sub implement user authentication with bcrypt
```

### Agent Team Orchestration

Dispatcher pattern with specialist agents:

```typescript
import type { PiExtension, PiApp } from "pi";
import { readFileSync } from "fs";
import { parse } from "yaml";
import { execSync } from "child_process";

interface TeamConfig {
  agents: {
    [key: string]: {
      description: string;
      system_prompt: string;
      model?: string;
    }
  };
}

export default {
  name: "agent-team",
  version: "1.0.0",

  async init(app: PiApp) {
    const config: TeamConfig = parse(
      readFileSync(".pi/agents/teams.yaml", "utf-8")
    );

    app.registerTool({
      name: "dispatch_agent",
      description: "Delegate a task to a specialist agent",
      parameters: {
        type: "object",
        properties: {
          agent_name: {
            type: "string",
            enum: Object.keys(config.agents),
            description: "Which specialist agent to use"
          },
          task: {
            type: "string",
            description: "The task to delegate"
          }
        },
        required: ["agent_name", "task"]
      },
      handler: async (args: { agent_name: string; task: string }) => {
        const agent = config.agents[args.agent_name];
        if (!agent) throw new Error(`Unknown agent: ${args.agent_name}`);

        // Write temporary agent config
        const agentPath = `/tmp/pi-agent-${args.agent_name}.json`;
        writeFileSync(agentPath, JSON.stringify({
          systemPrompt: agent.system_prompt,
          model: agent.model || app.session.model.name
        }));

        // Execute Pi with agent config
        const result = execSync(
          `pi -c ${agentPath} -p "${args.task}"`,
          { encoding: "utf-8", env: process.env }
        );

        return {
          agent: args.agent_name,
          output: result
        };
      }
    });
  }
} satisfies PiExtension;
```

Example `.pi/agents/teams.yaml`:

```yaml
agents:
  backend:
    description: Node.js/TypeScript backend specialist
    system_prompt: |
      You are an expert backend engineer specializing in Node.js, TypeScript,
      Express, and PostgreSQL. Focus on API design, database schemas, and server architecture.
    model: gpt-4o

  frontend:
    description: React/Next.js frontend specialist
    system_prompt: |
      You are an expert frontend engineer specializing in React, Next.js, and Tailwind.
      Focus on component architecture, state management, and responsive design.
    model: claude-opus-4

  security:
    description: Security auditing specialist
    system_prompt: |
      You are a security expert. Audit code for vulnerabilities, suggest fixes,
      and ensure best practices for authentication, authorization, and data protection.
    model: gpt-4o
```

## Pi-to-Pi Communication

### Local Communication (Same Machine)

Unix socket-based peer-to-peer messaging:

```typescript
import type { PiExtension, PiApp } from "pi";
import { createServer, connect } from "net";
import { existsSync, mkdirSync, unlinkSync } from "fs";

const SOCKET_DIR = "/tmp/pi-coms";
const messages = new Map<string, string[]>();

export default {
  name: "coms",
  version: "1.0.0",

  async init(app: PiApp) {
    if (!existsSync(SOCKET_DIR)) mkdirSync(SOCKET_DIR);
    
    const socketPath = `${SOCKET_DIR}/${app.session.id}.sock`;
    
    // Create local server
    const server = createServer((socket) => {
      socket.on("data", (data) => {
        const msg = JSON.parse(data.toString());
        if (!messages.has(msg.from)) messages.set(msg.from, []);
        messages.get(msg.from)!.push(msg.content);
      });
    });
    
    server.listen(socketPath);

    // Register communication tools
    app.registerTool({
      name: "coms_list",
      description: "List all available Pi agents on this machine",
      parameters: { type: "object", properties: {} },
      handler: async () => {
        const agents = readdirSync(SOCKET_DIR)
          .filter(f => f.endsWith(".sock"))
          .map(f => f.replace(".sock", ""));
        return { agents };
      }
    });

    app.registerTool({
      name: "coms_send",
      description: "Send a message to another Pi agent",
      parameters: {
        type: "object",
        properties: {
          to: { type: "string", description: "Target agent session ID" },
          message: { type: "string", description: "Message content" }
        },
        required: ["to", "message"]
      },
      handler: async (args: { to: string; message: string }) => {
        const targetSocket = `${SOCKET_DIR}/${args.to}.sock`;
        const client = connect(targetSocket);
        client.write(JSON.stringify({
          from: app.session.id,
          content: args.message
        }));
        client.end();
        return { sent: true };
      }
    });

    app.registerTool({
      name: "coms_get",
      description: "Retrieve messages from other agents",
      parameters: { type: "object", properties: {} },
      handler: async () => {
        const all = Array.from(messages.entries()).map(([from, msgs]) => ({
          from,
          messages: msgs
        }));
        messages.clear();
        return { messages: all };
      }
    });
  },

  async destroy(app: PiApp) {
    const socketPath = `${SOCKET_DIR}/${app.session.id}.sock`;
    if (existsSync(socketPath)) unlinkSync(socketPath);
  }
} satisfies PiExtension;
```

### Network Communication (Cross-Machine)

HTTP/SSE hub for LAN or remote agent coordination:

**Start the hub server:**
```bash
# Local development (127.0.0.1)
just coms-net-server

# LAN-accessible (requires auth token)
export PI_COMS_NET_AUTH_TOKEN=your-secret-token
just coms-net-server-lan
```

**Connect agents:**
```bash
# Terminal 1
just coms1  # Uses gpt-4o

# Terminal 2 (different machine on same network)
just coms2  # Uses claude-opus-4
```

The extension (`extensions/coms-net.ts`) provides tools:
- `coms_net_list` - List connected agents
- `coms_net_send` - Send message to specific agent
- `coms_net_broadcast` - Broadcast to all agents
- `coms_net_get` - Retrieve pending messages

## Cross-Agent Integration

Load commands and skills from Claude Code, Gemini, Codex directories:

```typescript
import type { PiExtension, PiApp } from "pi";
import { readdirSync, readFileSync, existsSync } from "fs";

export default {
  name: "cross-agent",
  version: "1.0.0",

  async init(app: PiApp) {
    const agentDirs = [".claude", ".gemini", ".codex"];
    
    for (const dir of agentDirs) {
      if (!existsSync(dir)) continue;
      
      // Load skills
      const skillsDir = `${dir}/skills`;
      if (existsSync(skillsDir)) {
        for (const file of readdirSync(skillsDir)) {
          if (!file.endsWith(".md")) continue;
          const content = readFileSync(`${skillsDir}/${file}`, "utf-8");
          app.addContextDocument({
            name: `${dir}-skill-${file}`,
            content
          });
        }
      }
      
      // Load agents
      const agentsDir = `${dir}/agents`;
      if (existsSync(agentsDir)) {
        for (const file of readdirSync(agentsDir)) {
          if (!file.endsWith(".md")) continue;
          const agentName = file.replace(".md", "");
          const systemPrompt = readFileSync(`${agentsDir}/${file}`, "utf-8");
          
          app.registerCommand({
            name: `${dir.slice(1)}-${agentName}`,
            description: `Switch to ${agentName} agent from ${dir}`,
            handler: async () => {
              app.session.systemPrompt = systemPrompt;
              return `Switched to ${agentName}`;
            }
          });
        }
      }
    }
  }
} satisfies PiExtension;
```

## Creating Your Own Extension

### 1. Create Extension File

```bash
touch extensions/my-feature.ts
```

### 2. Implement Extension

```typescript
import type { PiExtension, PiApp } from "pi";

export default {
  name: "my-feature",
  version: "1.0.0",
  description: "What this extension does",

  async init(app: PiApp) {
    // Setup: register tools, widgets, commands
    console.log("Extension initialized");
  },

  async beforeTurn(app: PiApp, turn: Turn) {
    // Pre-process agent turns
  },

  async afterTurn(app: PiApp, turn: Turn) {
    // Post-process agent turns
  },

  async destroy(app: PiApp) {
    // Cleanup
  }
} satisfies PiExtension;
```

### 3. Add Just Recipe

Edit `justfile`:

```makefile
ext-my-feature:
    #!/usr/bin/env bash
    set -euo pipefail
    source .env
    pi -e extensions/my-feature.ts
```

### 4. Test Extension

```bash
just ext-my-feature
```

## Configuration Files

### `.pi/settings.json`

Pi workspace settings:

```json
{
  "model": "gpt-4o",
  "temperature": 0.7,
  "maxTokens": 4096,
  "systemPrompt": "You are a helpful coding assistant.",
  "theme": "default"
}
```

### `.pi/damage-control-rules.yaml`

Safety rules for damage-control extension:

```yaml
blocked_patterns:
  - "rm -rf /"
  - "> /dev/sda"
  - "dd if="
  - "mkfs"
  - "curl | bash"

allowed_paths:
  - "./src"
  - "./tests"
  - "./extensions"
  - "/tmp"

max_file_size_mb: 10
```

### `.pi/agents/teams.yaml`

Agent team definitions:

```yaml
agents:
  researcher:
    description: Research and documentation specialist
    model: gpt-4o
    system_prompt: |
      You are a research specialist. Your job is to find accurate, up-to-date
      information from documentation, APIs, and code examples.

  implementer:
    description: Code implementation specialist
    model: claude-opus-4
    system_prompt: |
      You are an implementation specialist. Given requirements and research,
      you write clean, tested, production-ready code.
```

## Common Patterns

### Persistent State Across Turns

```typescript
const state = {
  taskList: [] as string[],
  completedCount: 0
};

export default {
  async init(app: PiApp) {
    app.registerTool({
      name: "add_task",
      parameters: {
        type: "object",
        properties: {
          task: { type: "string" }
        },
        required: ["task"]
      },
      handler: async (args: { task: string }) => {
        state.taskList.push(args.task);
        return { total: state.taskList.length };
      }
    });
  },

  async afterTurn(app: PiApp, turn: Turn) {
    // State persists across turns
    console.log(`Tasks: ${state.taskList.length}`);
  }
} satisfies PiExtension;
```

### Dynamic Widget Updates

```typescript
const widget = new Widget({
  position: "footer",
  render: () => dynamicContent
});

let dynamicContent = "Initial";

// Update from anywhere
setInterval(() => {
  dynamicContent = new Date().toISOString();
  app.ui.refresh();
}, 1000);
```

### Command Registration

```typescript
app.registerCommand({
  name: "mycommand",
  description: "Does something useful",
  handler: async (args: string[]) => {
    // args is array of space-separated arguments
    const result = await doSomething(args);
    return result; // Displayed to user
  }
});
```

Usage in Pi:
```
/mycommand arg1 arg2 arg3
```

## Troubleshooting

### Extension Not Loading

**Check TypeScript syntax:**
```bash
bun run tsc --noEmit extensions/my-extension.ts
```

**Verify extension export:**
```typescript
// Must have default export satisfying PiExtension
export default { ... } satisfies PiExtension;
```

### API Keys Not Working

**Verify environment:**
```bash
echo $OPENAI_API_KEY  # Should print your key
```

**Check .env file:**
```bash
cat .env  # Verify keys are set
source .env  # Load into current shell
```

### Widget Not Appearing

**Check position:**
```typescript
// Valid positions: "footer", "above-editor", "sidebar"
const widget = new Widget({ position: "footer", ... });
```

**Call refresh after updates:**
```typescript
widget.content = "New content";
app.ui.refresh();
```

### Tool Not Being Called

**Verify description clarity:**
```typescript
app.registerTool({
  name: "my_tool",
  description: "Clear, specific description of WHEN to use this tool",  // Important!
  // ...
});
```

**Check parameter schema:**
```typescript
parameters: {
  type: "object",
  properties: {
    required_param: {
      type: "string",
      description: "What this parameter does"  // Must have description
    }
  },
  required: ["required_param"]  // Don't forget this
}
```

### Subagent Communication Failing

**Check socket directory:**
```bash
ls -la /tmp/pi-coms/  # Should show .sock files
```

**Verify session IDs:**
```typescript
// In extension
console.log("My session ID:", app.session.id);
```

**For coms-net, check server is running:**
```bash
curl http://localhost:3141/health
# Should return: {"status":"ok","agents":0}
```

## Advanced: Building Multi-Step Workflows

Combine extensions for complex workflows:

```bash
# Safety-audited team orchestration with live monitoring
pi -e extensions/damage-control.ts \
   -e extensions/agent-team.ts \
   -e extensions/tool-counter-widget.ts
```

Create a custom workflow recipe in `justfile`:

```makefile
workflow-full-stack:
    #!/usr/bin/env bash
    set -euo pipefail
    source .env
    pi \
      -e extensions/agent-team.ts \
      -e extensions/damage-control.ts \
      -e extensions/tool-counter.ts \
      -e extensions/cross-agent.ts \
      -p "Build a full-stack app with backend and frontend teams"
```

## Resources

- **Pi Coding Agent**: https://github.com/mariozechner/pi-coding-agent
- **Provider Docs**: https://github.com/badlogic/pi-mono/blob/main/packages/coding-agent/docs/providers.md
- **Extension Video Tutorial**: https://youtu.be/f8cfH5XX-XU
- **Pi-to-Pi Communication**: https://youtu.be/PIdETjcXNIk
- **Project Repository**: https://github.com/disler/pi-vs-claude-code

## Contributing

Extensions in this repository are examples. To contribute:

1. Create a new extension in `extensions/`
2. Add a just recipe in `justfile`
3. Document in a spec file under `specs/`
4. Submit a PR with working examples
