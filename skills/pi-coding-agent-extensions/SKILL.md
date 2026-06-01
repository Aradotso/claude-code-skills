---
name: pi-coding-agent-extensions
description: Expert guidance for creating and using Pi Coding Agent extensions to customize AI coding workflows with custom UI, safety controls, and multi-agent orchestration
triggers:
  - "customize pi coding agent"
  - "create pi extension"
  - "pi agent ui customization"
  - "multi-agent orchestration with pi"
  - "pi coding agent safety controls"
  - "pi agent communication"
  - "extend pi coding agent"
  - "pi agent team workflow"
---

# Pi Coding Agent Extensions

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## Overview

Pi Coding Agent is an open-source alternative to Claude Code that supports deep customization through TypeScript extensions. This project demonstrates advanced extension patterns including custom UI, safety auditing, agent-to-agent communication, and multi-agent orchestration.

Pi extensions can:
- Modify the terminal UI (footer, widgets, overlays)
- Add custom commands and tools
- Implement safety controls and access policies
- Orchestrate multiple agents (parallel teams, sequential chains)
- Enable inter-agent communication (local or networked)

## Prerequisites

All three are required:

```bash
# Install Bun (runtime and package manager)
curl -fsSL https://bun.sh/install | bash

# Install just (task runner)
brew install just  # macOS
# or: cargo install just

# Install Pi Coding Agent CLI
# See: https://github.com/mariozechner/pi-coding-agent
```

## Installation

```bash
# Clone the repository
git clone https://github.com/disler/pi-vs-claude-code.git
cd pi-vs-claude-code

# Install dependencies
bun install

# Copy environment template
cp .env.sample .env
```

## API Key Configuration

Pi requires API keys in the shell environment before launch. Edit `.env` with your keys:

```bash
# .env
OPENAI_API_KEY=your_key_here
ANTHROPIC_API_KEY=your_key_here
GEMINI_API_KEY=your_key_here
OPENROUTER_API_KEY=your_key_here
```

**Load keys into environment:**

```bash
# Option A: Manual source each session
source .env && pi

# Option B: Use just (auto-loads .env)
just pi

# Option C: Add alias to ~/.zshrc or ~/.bashrc
alias pi='source $(pwd)/.env && pi'
```

## Running Extensions

### Single Extension

```bash
pi -e extensions/minimal.ts
```

### Stack Multiple Extensions

```bash
pi -e extensions/minimal.ts -e extensions/cross-agent.ts -e extensions/damage-control.ts
```

### Using Just Recipes

```bash
# List all recipes
just

# Common recipes
just pi                          # Plain Pi
just ext-minimal                 # Minimal footer with context meter
just ext-tool-counter            # Rich footer with tool call stats
just ext-damage-control          # Safety auditing
just ext-agent-team              # Multi-agent orchestration
just ext-pi-pi                   # Meta-agent builder
just all                         # Open all extensions in terminals

# Custom stack
just open purpose-gate minimal tool-counter-widget
```

## Extension Patterns

### 1. UI Customization - Minimal Footer

```typescript
// extensions/minimal.ts
import type { Agent } from "@pi/agent";
import { Plugin } from "@pi/agent";

export const MinimalFooter: Plugin = {
  name: "minimal-footer",
  
  renderFooter(agent: Agent, width: number): string {
    const model = agent.currentModel || "unknown";
    const usage = agent.contextUsage || 0;
    const blocks = 10;
    const filled = Math.floor((usage / 100) * blocks);
    const empty = blocks - filled;
    
    const meter = "[" + "#".repeat(filled) + "-".repeat(empty) + "]";
    return `${model} ${meter} ${usage}%`;
  }
};
```

### 2. Custom Commands - Purpose Gate

```typescript
// extensions/purpose-gate.ts
import type { Agent } from "@pi/agent";
import { Plugin } from "@pi/agent";

let sessionPurpose: string | null = null;

export const PurposeGate: Plugin = {
  name: "purpose-gate",
  
  async onStartup(agent: Agent) {
    const response = await agent.prompt(
      "What is the purpose of this session?",
      { required: true }
    );
    sessionPurpose = response.trim();
  },
  
  async beforePrompt(agent: Agent, userPrompt: string): Promise<boolean> {
    if (!sessionPurpose) {
      agent.error("Session purpose not set. Use /purpose to set it.");
      return false;
    }
    return true;
  },
  
  commands: {
    purpose: {
      description: "View or change session purpose",
      async execute(agent: Agent, args: string[]) {
        if (args.length === 0) {
          agent.info(`Current purpose: ${sessionPurpose}`);
        } else {
          sessionPurpose = args.join(" ");
          agent.info(`Purpose updated: ${sessionPurpose}`);
        }
      }
    }
  }
};
```

### 3. Custom Tools - Agent Communication

```typescript
// extensions/coms.ts
import type { Agent, Tool } from "@pi/agent";
import { Plugin } from "@pi/agent";
import fs from "fs";
import path from "path";

const COMS_DIR = "/tmp/pi-coms";

export const Coms: Plugin = {
  name: "coms",
  
  async onStartup() {
    if (!fs.existsSync(COMS_DIR)) {
      fs.mkdirSync(COMS_DIR, { recursive: true });
    }
  },
  
  tools: [
    {
      name: "coms_send",
      description: "Send a message to another Pi agent",
      parameters: {
        type: "object",
        properties: {
          to: { type: "string", description: "Target agent name" },
          message: { type: "string", description: "Message content" }
        },
        required: ["to", "message"]
      },
      async execute(agent: Agent, args: { to: string; message: string }) {
        const msgFile = path.join(COMS_DIR, `${args.to}.json`);
        const messages = fs.existsSync(msgFile)
          ? JSON.parse(fs.readFileSync(msgFile, "utf-8"))
          : [];
        
        messages.push({
          from: agent.name || "anonymous",
          message: args.message,
          timestamp: Date.now()
        });
        
        fs.writeFileSync(msgFile, JSON.stringify(messages, null, 2));
        return { success: true };
      }
    },
    {
      name: "coms_get",
      description: "Get messages for this agent",
      parameters: { type: "object", properties: {} },
      async execute(agent: Agent) {
        const msgFile = path.join(COMS_DIR, `${agent.name || "anonymous"}.json`);
        if (!fs.existsSync(msgFile)) {
          return { messages: [] };
        }
        const messages = JSON.parse(fs.readFileSync(msgFile, "utf-8"));
        fs.unlinkSync(msgFile); // Clear after reading
        return { messages };
      }
    }
  ]
};
```

### 4. Safety Controls - Damage Control

```typescript
// extensions/damage-control.ts
import type { Agent, ToolCall } from "@pi/agent";
import { Plugin } from "@pi/agent";
import yaml from "yaml";
import fs from "fs";

interface DamageRules {
  blocked_patterns: string[];
  allowed_paths: string[];
}

let rules: DamageRules;

export const DamageControl: Plugin = {
  name: "damage-control",
  
  async onStartup() {
    const rulesPath = ".pi/damage-control-rules.yaml";
    if (fs.existsSync(rulesPath)) {
      rules = yaml.parse(fs.readFileSync(rulesPath, "utf-8"));
    } else {
      rules = {
        blocked_patterns: ["rm -rf", "sudo", "> /dev/"],
        allowed_paths: ["./", "/tmp/"]
      };
    }
  },
  
  async beforeToolCall(agent: Agent, toolCall: ToolCall): Promise<boolean> {
    if (toolCall.tool.name === "bash") {
      const command = toolCall.arguments.command as string;
      
      // Check blocked patterns
      for (const pattern of rules.blocked_patterns) {
        if (command.includes(pattern)) {
          agent.error(`BLOCKED: Command contains dangerous pattern: ${pattern}`);
          return false;
        }
      }
      
      // Check path restrictions
      const pathMatch = command.match(/(?:^|\s)(\/[^\s]+)/);
      if (pathMatch) {
        const cmdPath = pathMatch[1];
        const allowed = rules.allowed_paths.some(p => cmdPath.startsWith(p));
        if (!allowed) {
          agent.error(`BLOCKED: Path outside allowed directories: ${cmdPath}`);
          return false;
        }
      }
    }
    
    return true;
  }
};
```

### 5. Multi-Agent Orchestration - Agent Teams

```typescript
// extensions/agent-team.ts
import type { Agent } from "@pi/agent";
import { Plugin } from "@pi/agent";
import yaml from "yaml";
import fs from "fs";
import { spawn } from "child_process";

interface TeamConfig {
  agents: Array<{
    name: string;
    role: string;
    model: string;
    systemPrompt: string;
  }>;
}

let teamConfig: TeamConfig;

export const AgentTeam: Plugin = {
  name: "agent-team",
  
  async onStartup() {
    teamConfig = yaml.parse(
      fs.readFileSync(".pi/agents/teams.yaml", "utf-8")
    );
  },
  
  tools: [
    {
      name: "dispatch_agent",
      description: "Dispatch work to a specialist agent",
      parameters: {
        type: "object",
        properties: {
          agent: { 
            type: "string", 
            enum: teamConfig?.agents.map(a => a.name) || [],
            description: "Target agent name"
          },
          task: { type: "string", description: "Task description" }
        },
        required: ["agent", "task"]
      },
      async execute(agent: Agent, args: { agent: string; task: string }) {
        const targetAgent = teamConfig.agents.find(a => a.name === args.agent);
        if (!targetAgent) {
          return { error: "Agent not found" };
        }
        
        // Spawn Pi process with agent config
        return new Promise((resolve) => {
          const proc = spawn("pi", [
            "--model", targetAgent.model,
            "--system", targetAgent.systemPrompt,
            "--prompt", args.task,
            "--headless"
          ]);
          
          let output = "";
          proc.stdout.on("data", (data) => {
            output += data.toString();
          });
          
          proc.on("close", () => {
            resolve({ result: output });
          });
        });
      }
    }
  ]
};
```

### 6. Sequential Pipeline - Agent Chain

```typescript
// extensions/agent-chain.ts
import type { Agent } from "@pi/agent";
import { Plugin } from "@pi/agent";
import yaml from "yaml";
import fs from "fs";

interface ChainStep {
  agent: string;
  prompt: string;
  outputVar?: string;
}

export const AgentChain: Plugin = {
  name: "agent-chain",
  
  commands: {
    chain: {
      description: "Run a sequential agent pipeline",
      async execute(agent: Agent, args: string[]) {
        const chainName = args[0];
        const chainPath = `.pi/agents/agent-chain.yaml`;
        const chains = yaml.parse(fs.readFileSync(chainPath, "utf-8"));
        const chain = chains[chainName] as ChainStep[];
        
        if (!chain) {
          agent.error(`Chain '${chainName}' not found`);
          return;
        }
        
        const context: Record<string, string> = {};
        
        for (const step of chain) {
          agent.info(`Running step: ${step.agent}`);
          
          // Interpolate variables in prompt
          let prompt = step.prompt;
          for (const [key, value] of Object.entries(context)) {
            prompt = prompt.replace(`{{${key}}}`, value);
          }
          
          // Execute agent step
          const result = await executeAgentStep(step.agent, prompt);
          
          if (step.outputVar) {
            context[step.outputVar] = result;
          }
        }
        
        agent.info("Chain complete!");
      }
    }
  }
};

async function executeAgentStep(agentName: string, prompt: string): Promise<string> {
  // Implementation similar to dispatch_agent in agent-team
  return "step result";
}
```

## Agent Definitions

### Team Configuration

```yaml
# .pi/agents/teams.yaml
agents:
  - name: researcher
    role: Research and documentation expert
    model: gpt-4o
    systemPrompt: |
      You are a research specialist. Search documentation, analyze APIs,
      and provide comprehensive findings.
  
  - name: coder
    role: Implementation expert
    model: claude-opus-4
    systemPrompt: |
      You are a coding specialist. Write clean, efficient code based on
      research findings and requirements.
  
  - name: tester
    role: Quality assurance expert
    model: gpt-4o-mini
    systemPrompt: |
      You are a testing specialist. Write comprehensive tests and
      validate implementations.
```

### Chain Pipeline

```yaml
# .pi/agents/agent-chain.yaml
feature-implementation:
  - agent: researcher
    prompt: "Research best practices for: {{feature_description}}"
    outputVar: research
  
  - agent: coder
    prompt: |
      Implement the following feature:
      {{feature_description}}
      
      Research findings:
      {{research}}
    outputVar: implementation
  
  - agent: tester
    prompt: |
      Write tests for this implementation:
      {{implementation}}
```

## Safety Rules Configuration

```yaml
# .pi/damage-control-rules.yaml
blocked_patterns:
  - "rm -rf"
  - "sudo"
  - "> /dev/"
  - "mkfs"
  - "dd if="
  - ":(){ :|:& };:"

allowed_paths:
  - "./"
  - "/tmp/"
  - "/var/tmp/"

require_confirmation:
  - "npm install"
  - "bun install"
  - "git push"
```

## Networked Agent Communication

### Start Communication Server

```bash
# Local server
just coms-net-server

# LAN-accessible server (requires auth token)
export PI_COMS_NET_AUTH_TOKEN=your_secret_token
just coms-net-server-lan
```

### Connect Agents

```bash
# Agent 1
export PI_COMS_NET_URL=http://localhost:3456
export PI_COMS_NET_AUTH_TOKEN=your_secret_token
just coms1

# Agent 2 (different terminal)
export PI_COMS_NET_URL=http://localhost:3456
export PI_COMS_NET_AUTH_TOKEN=your_secret_token
just coms2
```

### Communication Tools

The `coms-net` extension provides:

- `coms_net_send` - Send message to another agent
- `coms_net_get` - Get pending messages
- `coms_net_await` - Block until message received
- `coms_net_list` - List connected agents

## Common Patterns

### Persistent Widget Above Editor

```typescript
import type { Agent, Widget } from "@pi/agent";

const widget: Widget = {
  id: "my-widget",
  position: "above-editor",
  
  render(agent: Agent, width: number): string {
    return "Widget content";
  }
};

export const MyPlugin: Plugin = {
  name: "my-plugin",
  
  async onStartup(agent: Agent) {
    agent.registerWidget(widget);
  }
};
```

### Tracking Tool Calls

```typescript
const toolCalls = new Map<string, number>();

export const ToolTracker: Plugin = {
  name: "tool-tracker",
  
  async afterToolCall(agent: Agent, toolCall: ToolCall, result: any) {
    const count = toolCalls.get(toolCall.tool.name) || 0;
    toolCalls.set(toolCall.tool.name, count + 1);
    
    // Update UI
    agent.updateWidget("tool-stats");
  }
};
```

### Custom Theme

```json
// .pi/themes/my-theme.json
{
  "name": "my-theme",
  "colors": {
    "primary": "#00ff00",
    "secondary": "#ff00ff",
    "success": "#00ffff",
    "error": "#ff0000",
    "warning": "#ffff00",
    "bg": "#1a1a1a",
    "fg": "#ffffff"
  }
}
```

## Troubleshooting

### Extensions Not Loading

```bash
# Verify extension syntax
bun run extensions/your-extension.ts

# Check Pi can find the file
pi -e extensions/your-extension.ts --debug
```

### API Keys Not Working

```bash
# Verify keys are in environment
echo $OPENAI_API_KEY

# Re-source .env
source .env

# Use just to auto-load
just ext-minimal
```

### Agent Communication Issues

```bash
# Local coms - check directory
ls -la /tmp/pi-coms/

# Network coms - verify server running
curl http://localhost:3456/health

# Check auth token matches
echo $PI_COMS_NET_AUTH_TOKEN
```

### Tool Calls Being Blocked

```bash
# Check damage control rules
cat .pi/damage-control-rules.yaml

# Use continue variant to see feedback
just ext-damage-control-continue
```

### Widget Not Rendering

```typescript
// Ensure widget is registered
async onStartup(agent: Agent) {
  agent.registerWidget(myWidget);
}

// Force widget update
agent.updateWidget("widget-id");

// Check render method returns string
render(agent: Agent, width: number): string {
  return "content";
}
```

## Resources

- [Pi Coding Agent GitHub](https://github.com/mariozechner/pi-coding-agent)
- [Pi Provider Docs](https://github.com/badlogic/pi-mono/blob/main/packages/coding-agent/docs/providers.md)
- [Project Video Demo](https://youtu.be/f8cfH5XX-XU)
- [Pi-to-Pi Communication Demo](https://youtu.be/PIdETjcXNIk)
