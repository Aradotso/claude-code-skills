---
name: claude-code-templates-cli
description: Install and manage AI agents, commands, MCPs, settings, and hooks for Claude Code development workflows
triggers:
  - install claude code templates
  - add new agent to claude code
  - browse available claude agents
  - set up mcp integration
  - configure claude code settings
  - add custom slash commands
  - install development team agents
  - check claude code health
---

# Claude Code Templates CLI

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

The `claude-code-templates` CLI provides a comprehensive collection of ready-to-use configurations for Anthropic's Claude Code, including AI agents, custom commands, MCPs (Model Context Protocol integrations), settings, hooks, and skills. It offers both interactive browsing and direct installation of components to enhance AI-powered development workflows.

## Installation

The CLI can be run directly with `npx` without installation:

```bash
npx claude-code-templates@latest
```

For frequent use, install globally:

```bash
npm install -g claude-code-templates
```

## Core Commands

### Interactive Installation

Launch the interactive browser to explore and install components:

```bash
npx claude-code-templates@latest
```

This opens a menu-driven interface to browse agents, commands, MCPs, settings, and hooks.

### Direct Component Installation

Install specific components using flags:

```bash
# Install an agent
npx claude-code-templates@latest --agent development-team/frontend-developer --yes

# Install a command
npx claude-code-templates@latest --command testing/generate-tests --yes

# Install an MCP integration
npx claude-code-templates@latest --mcp development/github-integration --yes

# Install a setting
npx claude-code-templates@latest --setting performance/mcp-timeouts --yes

# Install a hook
npx claude-code-templates@latest --hook git/pre-commit-validation --yes
```

The `--yes` flag skips confirmation prompts for automated workflows.

### Batch Installation

Install multiple components in a single command:

```bash
npx claude-code-templates@latest \
  --agent development-team/frontend-developer \
  --agent security/security-auditor \
  --command testing/generate-tests \
  --mcp development/github-integration \
  --yes
```

## Development Tools

### Analytics Dashboard

Monitor Claude Code sessions with real-time state detection:

```bash
npx claude-code-templates@latest --analytics
```

Opens a web interface showing:
- Active conversation state
- Performance metrics
- Token usage
- Session timeline

### Conversation Monitor

View Claude responses in real-time with mobile-optimized interface:

```bash
# Local access
npx claude-code-templates@latest --chats

# Remote access via Cloudflare Tunnel
npx claude-code-templates@latest --chats --tunnel
```

The tunnel option provides secure remote access without port forwarding.

### Health Check

Run comprehensive diagnostics:

```bash
npx claude-code-templates@latest --health-check
```

Checks:
- Claude Code installation
- Configuration validity
- MCP integrations
- File permissions
- Dependencies

### Plugin Dashboard

Manage installed plugins and view marketplaces:

```bash
npx claude-code-templates@latest --plugins
```

## Component Types

### Agents

Specialized AI personas for specific development tasks:

```bash
# Install a frontend developer agent
npx claude-code-templates@latest --agent development-team/frontend-developer --yes

# Install a security auditor
npx claude-code-templates@latest --agent security/security-auditor --yes

# Install a database architect
npx claude-code-templates@latest --agent data/database-architect --yes
```

Agents are installed to `.claude/agents/` in your project.

### Commands

Custom slash commands for Claude Code:

```bash
# Add test generation command
npx claude-code-templates@latest --command testing/generate-tests --yes

# Add bundle optimization command
npx claude-code-templates@latest --command performance/optimize-bundle --yes

# Add security check command
npx claude-code-templates@latest --command security/check-security --yes
```

Commands are installed to `.claude/commands/` and can be invoked with `/command-name`.

### MCPs (Model Context Protocol)

External service integrations:

```bash
# GitHub integration
npx claude-code-templates@latest --mcp development/github-integration --yes

# PostgreSQL integration
npx claude-code-templates@latest --mcp database/postgresql-integration --yes

# AWS integration
npx claude-code-templates@latest --mcp cloud/aws-integration --yes
```

MCPs require configuration with environment variables:

```bash
# Example: GitHub MCP configuration
export GITHUB_TOKEN=your_token_here
export GITHUB_OWNER=your_username
export GITHUB_REPO=your_repo
```

### Settings

Claude Code configuration presets:

```bash
# Optimize MCP timeouts
npx claude-code-templates@latest --setting performance/mcp-timeouts --yes

# Configure memory settings
npx claude-code-templates@latest --setting performance/memory-limits --yes
```

Settings are merged into `.claude/settings.json`.

### Hooks

Automation triggers for development workflows:

```bash
# Pre-commit validation
npx claude-code-templates@latest --hook git/pre-commit-validation --yes

# Post-completion actions
npx claude-code-templates@latest --hook workflow/post-completion --yes
```

## Configuration

The CLI respects Claude Code's standard configuration structure:

```
.claude/
├── agents/           # Installed agents
├── commands/         # Custom commands
├── mcps/            # MCP integrations
├── settings.json    # Global settings
├── hooks/           # Automation hooks
└── skills/          # Reusable capabilities
```

### Environment Variables

For MCP integrations requiring credentials:

```bash
# GitHub
export GITHUB_TOKEN=${GITHUB_TOKEN}

# PostgreSQL
export POSTGRES_HOST=${POSTGRES_HOST}
export POSTGRES_USER=${POSTGRES_USER}
export POSTGRES_PASSWORD=${POSTGRES_PASSWORD}

# AWS
export AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
export AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
```

## Common Patterns

### Setting Up a Development Stack

```bash
# Install a complete frontend development environment
npx claude-code-templates@latest \
  --agent development-team/frontend-developer \
  --agent performance/react-optimizer \
  --command testing/generate-tests \
  --command performance/optimize-bundle \
  --mcp development/github-integration \
  --setting performance/mcp-timeouts \
  --yes
```

### Adding Security Tools

```bash
# Security-focused setup
npx claude-code-templates@latest \
  --agent security/security-auditor \
  --command security/check-security \
  --command security/dependency-audit \
  --hook git/pre-commit-validation \
  --yes
```

### Database Development

```bash
# Database-focused setup
npx claude-code-templates@latest \
  --agent data/database-architect \
  --command database/optimize-queries \
  --mcp database/postgresql-integration \
  --yes
```

## Programmatic Usage

The CLI can be imported as a Node.js module:

```javascript
const { installComponent } = require('claude-code-templates');

// Install an agent programmatically
await installComponent({
  type: 'agent',
  name: 'development-team/frontend-developer',
  skipConfirmation: true
});

// Install multiple components
const components = [
  { type: 'agent', name: 'security/security-auditor' },
  { type: 'command', name: 'testing/generate-tests' },
  { type: 'mcp', name: 'development/github-integration' }
];

for (const component of components) {
  await installComponent({ ...component, skipConfirmation: true });
}
```

## Troubleshooting

### Components Not Appearing in Claude Code

Verify installation location:

```bash
ls -la .claude/agents/
ls -la .claude/commands/
ls -la .claude/mcps/
```

Ensure Claude Code is configured to read from the `.claude` directory.

### MCP Integration Failures

Run health check to diagnose MCP issues:

```bash
npx claude-code-templates@latest --health-check
```

Verify environment variables are set:

```bash
echo $GITHUB_TOKEN
echo $POSTGRES_HOST
```

### Permission Errors

Ensure write permissions to the `.claude` directory:

```bash
chmod -R u+w .claude/
```

### Tunnel Connection Issues

For remote conversation monitoring, ensure Cloudflare CLI is installed:

```bash
# Install cloudflared
brew install cloudflare/cloudflare/cloudflared

# Or download from https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/installation/
```

### Component Conflicts

If components conflict, remove existing ones:

```bash
rm .claude/agents/conflicting-agent.md
npx claude-code-templates@latest --agent new/agent --yes
```

## Web Interface

Browse all available components at [aitmpl.com](https://aitmpl.com):

- Filter by category (agents, commands, MCPs, settings, hooks)
- Search by keywords
- View component details and examples
- Copy installation commands
- Track installation statistics

Documentation available at [docs.aitmpl.com](https://docs.aitmpl.com).
