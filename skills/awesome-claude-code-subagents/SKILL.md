---
name: awesome-claude-code-subagents
description: Collection of 130+ specialized Claude Code subagents for development tasks across languages, frameworks, infrastructure, and security
triggers:
  - install claude code subagents
  - show me available claude subagents
  - add a specialized agent for my project
  - what subagents are available for python
  - install the terraform expert subagent
  - browse claude code agent categories
  - set up language specialist agents
  - get help with subagent installation
---

# Awesome Claude Code Subagents

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## What It Is

A curated collection of 130+ specialized Claude Code subagents organized into 9 categories. Each subagent is a markdown file that configures Claude with expert knowledge in a specific domain (language, framework, infrastructure, security, etc.). Installing subagents enhances Claude's capabilities for targeted development tasks.

## Installation Methods

### Method 1: Individual Agent Installation

Copy specific agent files to your Claude agents directory:

```bash
# Global installation (available in all projects)
curl -o ~/.claude/agents/python-pro.md \
  https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/categories/02-language-specialists/python-pro.md

# Project-specific installation
mkdir -p .claude/agents
curl -o .claude/agents/terraform-engineer.md \
  https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/categories/03-infrastructure/terraform-engineer.md
```

### Method 2: Interactive Installer (Recommended)

```bash
# Clone and run interactive installer
git clone https://github.com/VoltAgent/awesome-claude-code-subagents.git
cd awesome-claude-code-subagents
chmod +x install-agents.sh
./install-agents.sh
```

The installer provides:
- Browse by category
- Search agents by keyword
- Install/uninstall globally or per-project
- View agent descriptions

### Method 3: Standalone Installer (No Clone)

```bash
# Download and run installer directly
curl -sO https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/install-agents.sh
chmod +x install-agents.sh
./install-agents.sh
```

### Method 4: Meta Agent Installer

Install the agent-installer subagent to manage others through Claude:

```bash
curl -s https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/categories/09-meta-orchestration/agent-installer.md \
  -o ~/.claude/agents/agent-installer.md
```

Then use natural language:
- "Show me available Python subagents"
- "Install the FastAPI developer agent globally"
- "What infrastructure agents are available?"

## Categories and Key Agents

### 01. Core Development (`categories/01-core-development/`)

Essential development subagents:

```bash
# API development
curl -o ~/.claude/agents/api-designer.md \
  https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/categories/01-core-development/api-designer.md

# Full-stack development
curl -o ~/.claude/agents/fullstack-developer.md \
  https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/categories/01-core-development/fullstack-developer.md
```

**Key agents**: api-designer, backend-developer, frontend-developer, fullstack-developer, microservices-architect, mobile-developer

### 02. Language Specialists (`categories/02-language-specialists/`)

Language and framework experts:

```bash
# Python specialist
curl -o ~/.claude/agents/python-pro.md \
  https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/categories/02-language-specialists/python-pro.md

# TypeScript specialist
curl -o ~/.claude/agents/typescript-pro.md \
  https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/categories/02-language-specialists/typescript-pro.md

# Framework specialists
curl -o ~/.claude/agents/nextjs-developer.md \
  https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/categories/02-language-specialists/nextjs-developer.md
```

**Key agents**: python-pro, typescript-pro, golang-pro, rust-engineer, nextjs-developer, react-specialist, django-developer, fastapi-developer, laravel-specialist, spring-boot-engineer

### 03. Infrastructure (`categories/03-infrastructure/`)

DevOps and cloud specialists:

```bash
# Terraform expert
curl -o ~/.claude/agents/terraform-engineer.md \
  https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/categories/03-infrastructure/terraform-engineer.md

# Kubernetes specialist
curl -o ~/.claude/agents/kubernetes-specialist.md \
  https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/categories/03-infrastructure/kubernetes-specialist.md
```

**Key agents**: terraform-engineer, kubernetes-specialist, docker-expert, devops-engineer, cloud-architect, azure-infra-engineer, sre-engineer

### 04. Quality & Security (`categories/04-quality-security/`)

Testing and security experts:

```bash
# Code reviewer
curl -o ~/.claude/agents/code-reviewer.md \
  https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/categories/04-quality-security/code-reviewer.md

# Security specialist
curl -o ~/.claude/agents/penetration-tester.md \
  https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/categories/04-quality-security/penetration-tester.md
```

**Key agents**: code-reviewer, security-engineer, penetration-tester, qa-engineer, debugger, compliance-auditor

## Usage Patterns

### Pattern 1: Project-Specific Setup

Install agents relevant to your tech stack:

```bash
# Next.js + TypeScript + PostgreSQL project
mkdir -p .claude/agents

# Frontend
curl -o .claude/agents/nextjs-developer.md \
  https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/categories/02-language-specialists/nextjs-developer.md

curl -o .claude/agents/typescript-pro.md \
  https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/categories/02-language-specialists/typescript-pro.md

# Database
curl -o .claude/agents/sql-pro.md \
  https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/categories/02-language-specialists/sql-pro.md

# Infrastructure
curl -o .claude/agents/docker-expert.md \
  https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/categories/03-infrastructure/docker-expert.md
```

### Pattern 2: Global Utilities

Install universal helpers globally:

```bash
# Code quality
curl -o ~/.claude/agents/code-reviewer.md \
  https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/categories/04-quality-security/code-reviewer.md

# Documentation
curl -o ~/.claude/agents/documentation-writer.md \
  https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/categories/05-documentation-content/documentation-writer.md

# Git workflows
curl -o ~/.claude/agents/git-specialist.md \
  https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/categories/06-data-integration/git-specialist.md
```

### Pattern 3: Batch Category Installation

Install entire categories with a script:

```bash
# Install all language specialists
CATEGORY="02-language-specialists"
BASE_URL="https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/categories"

mkdir -p ~/.claude/agents

for agent in python-pro typescript-pro golang-pro rust-engineer; do
  curl -o ~/.claude/agents/${agent}.md \
    ${BASE_URL}/${CATEGORY}/${agent}.md
done
```

## Working with Subagents

### Activating a Subagent

Once installed, reference agents in your prompts:

```
@python-pro help me optimize this data processing pipeline
```

```
@terraform-engineer review this AWS infrastructure code
```

```
@code-reviewer analyze this pull request for potential issues
```

### Combining Multiple Subagents

```
@frontend-developer create a React component for the dashboard
@typescript-pro ensure type safety
@accessibility-tester verify WCAG compliance
```

### Creating Custom Subagents

Use existing agents as templates:

```bash
# Copy an existing agent
cp ~/.claude/agents/python-pro.md ~/.claude/agents/my-custom-agent.md

# Edit the frontmatter and instructions
vim ~/.claude/agents/my-custom-agent.md
```

Basic agent structure:

```markdown
---
name: my-custom-agent
description: Custom agent for specific task
---

# My Custom Agent

You are an expert in [domain]. Your role is to [purpose].

## Key Responsibilities
- Responsibility 1
- Responsibility 2

## Expertise
- Area 1
- Area 2

## Best Practices
- Practice 1
- Practice 2
```

## Configuration

### Agent Directory Structure

```
~/.claude/                    # Global agents
  agents/
    python-pro.md
    terraform-engineer.md
    code-reviewer.md

project-root/                 # Project-specific agents
  .claude/
    agents/
      nextjs-developer.md
      sql-pro.md
```

### Environment Variables

Agents may reference environment variables for credentials:

```bash
# .env or shell profile
export OPENAI_API_KEY="your-key-here"
export ANTHROPIC_API_KEY="your-key-here"
export GITHUB_TOKEN="your-token-here"
```

## Troubleshooting

### Agent Not Found

**Issue**: Claude doesn't recognize the agent reference

```bash
# Verify agent exists
ls ~/.claude/agents/
ls .claude/agents/

# Check agent filename matches reference
# Reference: @python-pro
# File should be: python-pro.md (not python_pro.md or pythonpro.md)
```

### Agent Not Working as Expected

**Issue**: Agent behavior doesn't match description

1. **Check file integrity**:
```bash
# Ensure file downloaded completely
wc -l ~/.claude/agents/python-pro.md

# Re-download if needed
curl -o ~/.claude/agents/python-pro.md \
  https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/categories/02-language-specialists/python-pro.md
```

2. **Verify frontmatter format**:
```bash
# First few lines should be YAML
head -10 ~/.claude/agents/python-pro.md
```

3. **Check for conflicts**:
```bash
# Multiple agents with same name can conflict
find ~/.claude .claude -name "python-pro.md"
```

### Installation Script Errors

**Issue**: `install-agents.sh` fails

```bash
# Make script executable
chmod +x install-agents.sh

# Check dependencies
which curl  # Required for downloading agents

# Run with verbose output
bash -x ./install-agents.sh
```

### Rate Limiting

**Issue**: GitHub API rate limiting when downloading many agents

```bash
# Use authenticated requests (higher rate limit)
export GITHUB_TOKEN="your-personal-access-token"

# Or download repository once and install locally
git clone https://github.com/VoltAgent/awesome-claude-code-subagents.git
cd awesome-claude-code-subagents
./install-agents.sh
```

## Advanced Usage

### Dynamic Agent Loading

Load agents programmatically based on project detection:

```bash
#!/bin/bash
# auto-install-agents.sh

# Detect project type and install relevant agents
if [ -f "package.json" ]; then
  # Node.js project
  curl -o .claude/agents/node-specialist.md \
    https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/categories/02-language-specialists/node-specialist.md
  
  # Check for Next.js
  if grep -q "next" package.json; then
    curl -o .claude/agents/nextjs-developer.md \
      https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/categories/02-language-specialists/nextjs-developer.md
  fi
fi

if [ -f "requirements.txt" ] || [ -f "pyproject.toml" ]; then
  # Python project
  curl -o .claude/agents/python-pro.md \
    https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/categories/02-language-specialists/python-pro.md
fi

if [ -f "Cargo.toml" ]; then
  # Rust project
  curl -o .claude/agents/rust-engineer.md \
    https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/categories/02-language-specialists/rust-engineer.md
fi
```

### Agent Updates

Keep agents updated with latest improvements:

```bash
#!/bin/bash
# update-agents.sh

AGENTS_DIR="$HOME/.claude/agents"
BASE_URL="https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main"

for agent in "$AGENTS_DIR"/*.md; do
  agent_name=$(basename "$agent")
  
  # Find agent in repository structure
  # This is simplified - actual implementation would need category lookup
  curl -o "$agent" "${BASE_URL}/categories/*/${agent_name}" 2>/dev/null || \
    echo "Could not update $agent_name"
done
```

### Multi-Agent Workflows

Create workflow templates combining multiple agents:

```bash
# .claude/workflows/code-review.md
---
name: comprehensive-code-review
agents: [code-reviewer, security-engineer, accessibility-tester]
---

# Comprehensive Code Review Workflow

1. @code-reviewer: Analyze code quality and patterns
2. @security-engineer: Check for security vulnerabilities
3. @accessibility-tester: Verify accessibility compliance
4. Generate consolidated report
```

## Common Patterns by Use Case

### Backend API Development
```bash
curl -o .claude/agents/api-designer.md https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/categories/01-core-development/api-designer.md
curl -o .claude/agents/fastapi-developer.md https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/categories/02-language-specialists/fastapi-developer.md
curl -o .claude/agents/sql-pro.md https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/categories/02-language-specialists/sql-pro.md
```

### Cloud Infrastructure
```bash
curl -o .claude/agents/terraform-engineer.md https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/categories/03-infrastructure/terraform-engineer.md
curl -o .claude/agents/kubernetes-specialist.md https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/categories/03-infrastructure/kubernetes-specialist.md
curl -o .claude/agents/cloud-architect.md https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/categories/03-infrastructure/cloud-architect.md
```

### Mobile Development
```bash
curl -o .claude/agents/mobile-developer.md https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/categories/01-core-development/mobile-developer.md
curl -o .claude/agents/flutter-expert.md https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/categories/02-language-specialists/flutter-expert.md
curl -o .claude/agents/swift-expert.md https://raw.githubusercontent.com/VoltAgent/awesome-claude-code-subagents/main/categories/02-language-specialists/swift-expert.md
```
