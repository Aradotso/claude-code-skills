---
name: awesome-claude-code-toolkit
description: Comprehensive toolkit for Claude Code with 135+ agents, 176+ plugins, 42 commands, 20 hooks, 15 rules, and 400K+ skills via SkillKit
triggers:
  - install claude code toolkit
  - add claude code plugins
  - set up claude code hooks
  - configure claude code agents
  - use claude code skills
  - extend claude code functionality
  - customize claude code workflow
  - manage claude code ecosystem
---

# Awesome Claude Code Toolkit

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

The most comprehensive toolkit for Claude Code, providing 135 agents, 35 curated skills (+400,000 via SkillKit), 42 commands, 176+ plugins, 20 hooks, 15 rules, 7 templates, 14 MCP configs, 26 companion apps, and 52+ ecosystem entries.

## Installation

### Via Plugin Marketplace (Recommended)

```bash
/plugin marketplace add rohitg00/awesome-claude-code-toolkit
```

### Manual Clone

```bash
git clone https://github.com/rohitg00/awesome-claude-code-toolkit.git ~/.claude/plugins/claude-code-toolkit
```

### One-Liner Install

```bash
curl -fsSL https://raw.githubusercontent.com/rohitg00/awesome-claude-code-toolkit/main/setup/install.sh | bash
```

## Core Components

### Plugins (176+)

The toolkit includes production-ready plugins across multiple domains:

#### Featured Plugins

**pro-workflow** - Battle-tested workflows with self-correcting memory:
```bash
/plugin marketplace add rohitg00/pro-workflow
```

**gstack** - Garry Tan's opinionated Claude Code setup:
```bash
/plugin marketplace add garrytan/gstack
```

**agento-patronum** - Security layer for sensitive files:
```bash
git clone https://github.com/emaarco/agento-patronum.git ~/.claude/plugins/agento-patronum
```

#### Domain-Specific Plugins

**AWS Cost Optimization**:
```bash
/plugin marketplace add prajapatimehul/aws-cost-saver
```

**Code Quality & Reviews**:
```bash
# Brooks-lint: Classic engineering book principles
/plugin marketplace add hyhmrright/brooks-lint

# Bouncer: Independent quality gate with Gemini
/plugin marketplace add buildingopen/bouncer
```

**Project Management**:
```bash
# CCPM: GitHub Issues + Git worktrees
/plugin marketplace add automazeio/ccpm

# Fractal: Recursive project management
/plugin marketplace add rmolines/fractal
```

**Cost Control**:
```bash
# Cost optimizer skill
npx skills add Sagargupta16/claude-cost-optimizer

# Usage analyzer
npm install -g ccusage
ccusage analyze
```

### Agents (135)

The toolkit provides specialized agents for different roles:

#### Software Development Lifecycle

**great_cto** - Full SDLC pipeline with 7 agents:
```javascript
// Agents included:
// - tech-lead: Architecture decisions
// - senior-dev: Code implementation
// - qa-engineer: Testing strategy
// - security-officer: Security audits
// - devops: CI/CD and infrastructure
// - l3-support: Production issues
// - project-auditor: Compliance checks

// Usage via plugin
/plugin marketplace add avelikiy/great_cto
```

#### Custom Agent Configuration

```yaml
# agents/backend-architect.yml
name: backend-architect
role: Backend Service Architect
capabilities:
  - API design with OpenAPI specs
  - Database schema design
  - Microservice decomposition
  - Performance optimization
context:
  - Always consider scalability
  - Follow REST/GraphQL best practices
  - Include authentication patterns
```

### Skills (35 Curated + 400K via SkillKit)

#### Installing Skills

```bash
# Add individual skill
npx skills add <username>/<skill-name>

# Cost optimizer with templates
npx skills add Sagargupta16/claude-cost-optimizer

# Search SkillKit marketplace
npx skills search "database migration"
```

#### Creating Custom Skills

```markdown
---
name: my-custom-skill
description: Custom skill for specific workflow
triggers:
  - perform custom workflow
  - run custom task
---

# My Custom Skill

## Usage

When user requests this skill:
1. Analyze requirements
2. Execute steps
3. Validate results
```

### Commands (42)

#### Built-in Toolkit Commands

```bash
# Plugin management
/plugin marketplace search <query>
/plugin marketplace add <owner>/<repo>
/plugin list
/plugin remove <name>

# Skill management
/skill add <name>
/skill list
/skill remove <name>

# Agent management
/agent activate <name>
/agent list
/agent deactivate <name>

# Project scaffolding
npx claude-scaffold init
npx claude-scaffold update --all

# Cost analysis
ccusage analyze
ccusage report --monthly
ccusage diagnose --top 10
```

#### Skills Janitor Commands

```bash
# Audit skills
/skills-audit

# Deduplicate
/skills-dedupe

# Check validity
/skills-check

# Fix common issues
/skills-fix

# Usage tracking
/skills-usage
```

### Hooks (20)

Hooks execute scripts at key lifecycle points:

#### Hook Types

```javascript
// pre-approve.js - Validation before approval
module.exports = async (context) => {
  const { files, command } = context;
  
  // Block sensitive file access
  const sensitivePatterns = ['.env', 'id_rsa', 'credentials'];
  const blocked = files.filter(f => 
    sensitivePatterns.some(p => f.includes(p))
  );
  
  if (blocked.length > 0) {
    throw new Error(`Blocked access to: ${blocked.join(', ')}`);
  }
};
```

```javascript
// post-task.js - Archive after completion
module.exports = async (context) => {
  const { task, result, metadata } = context;
  
  // Log to memory system
  await context.memory.store({
    task: task.description,
    result: result.summary,
    timestamp: Date.now(),
    cost: metadata.tokens
  });
};
```

#### Agento Patronum Security Hooks

```bash
# Install security hooks
git clone https://github.com/emaarco/agento-patronum.git ~/.claude/hooks/security

# Configure protected patterns
cat > ~/.claude/hooks/security/config.json << 'EOF'
{
  "patterns": {
    "credentials": [".env", ".env.*", "*.pem", "id_rsa*"],
    "cloud": ["~/.aws/*", "~/.kube/config"],
    "secrets": ["secret*", "password*", "token*"]
  }
}
EOF
```

### Rules (15)

Rules guide Claude Code behavior:

```markdown
# rules/cost-optimization.md

## Cost Optimization Rules

1. **Minimize Context**: Only include relevant files
2. **Cache Prompts**: Use consistent system prompts
3. **Batch Operations**: Group similar tasks
4. **Incremental Changes**: Avoid full rewrites
5. **Smart Diffs**: Use targeted edits

## Implementation

- Activate cost-mode skill for 30-60% savings
- Use Haiku for routine tasks
- Reserve Opus for complex architecture
```

```markdown
# rules/security-first.md

## Security Rules

1. Never commit secrets to version control
2. Use environment variables: `process.env.API_KEY`
3. Validate all user input
4. Apply principle of least privilege
5. Enable audit logging for sensitive operations
```

### Templates (7)

#### Project Templates

```bash
# Initialize with template
/template apply node-api

# Available templates:
# - node-api: Express REST API
# - react-app: React + TypeScript
# - python-cli: Python CLI tool
# - fullstack: Next.js + tRPC
# - microservice: Docker + K8s
# - ml-pipeline: MLOps setup
# - mobile-app: React Native
```

#### Custom Template Structure

```
templates/
├── node-api/
│   ├── template.yml
│   ├── src/
│   │   ├── index.ts
│   │   ├── routes/
│   │   └── middleware/
│   ├── tests/
│   ├── .env.example
│   └── README.md
```

### MCP Configs (14)

Model Context Protocol server configurations:

#### Claude Context (Semantic Search)

```json
{
  "mcpServers": {
    "claude-context": {
      "command": "npx",
      "args": ["-y", "@zilliz/claude-context"],
      "env": {
        "MILVUS_URI": "${MILVUS_URI}",
        "OPENAI_API_KEY": "${OPENAI_API_KEY}"
      }
    }
  }
}
```

#### Claude Code as MCP Server

```json
{
  "mcpServers": {
    "claude-code": {
      "command": "npx",
      "args": ["claude-code-mcp"],
      "env": {
        "ANTHROPIC_API_KEY": "${ANTHROPIC_API_KEY}"
      }
    }
  }
}
```

#### Custom MCP Configuration

```json
{
  "mcpServers": {
    "custom-tools": {
      "command": "node",
      "args": ["./mcp-servers/custom/index.js"],
      "env": {
        "DATABASE_URL": "${DATABASE_URL}",
        "REDIS_URL": "${REDIS_URL}"
      }
    }
  }
}
```

## Common Workflows

### Complete Setup Flow

```bash
# 1. Install toolkit
/plugin marketplace add rohitg00/awesome-claude-code-toolkit

# 2. Install essential plugins
/plugin marketplace add rohitg00/pro-workflow
/plugin marketplace add buildingopen/bouncer

# 3. Set up security hooks
git clone https://github.com/emaarco/agento-patronum.git ~/.claude/hooks/security

# 4. Add cost optimization
npx skills add Sagargupta16/claude-cost-optimizer

# 5. Configure MCP servers
# Edit ~/.claude/config.json with MCP settings

# 6. Initialize project scaffolding
npx claude-scaffold init
```

### Cost Optimization Workflow

```bash
# 1. Analyze current usage
ccusage analyze --last 30d

# 2. Enable cost-mode skill
/skill activate claude-cost-optimizer

# 3. Set up budget hooks
cat > ~/.claude/hooks/pre-approve.js << 'EOF'
module.exports = async (context) => {
  const budget = 10.0; // USD per day
  const spent = await context.usage.dailySpend();
  if (spent > budget) {
    throw new Error(`Daily budget exceeded: $${spent}`);
  }
};
EOF

# 4. Run diagnostics
ccusage diagnose --top 10
```

### Code Quality Workflow

```bash
# 1. Install quality gates
/plugin marketplace add buildingopen/bouncer
/plugin marketplace add hyhmrright/brooks-lint

# 2. Configure automatic audits
/skill activate bouncer-quick-audit

# 3. Run pre-commit hook
cat > .git/hooks/pre-commit << 'EOF'
#!/bin/bash
npx bouncer audit --quick
npx brooks-lint check --staged
EOF
chmod +x .git/hooks/pre-commit
```

### Multi-Agent Project Management

```bash
# 1. Install project manager
/plugin marketplace add automazeio/ccpm

# 2. Create epic from issues
ccpm epic-start "Feature: User Authentication"

# 3. Activate agents
/agent activate backend-architect
/agent activate security-officer
/agent activate qa-engineer

# 4. Parallel execution
ccpm issue-analyze --parallel

# 5. Merge results
ccpm epic-merge
```

## Configuration Examples

### Settings.json Integration

```json
{
  "claude": {
    "plugins": {
      "autoload": [
        "awesome-claude-code-toolkit",
        "pro-workflow",
        "agento-patronum"
      ]
    },
    "skills": {
      "autoActivate": true,
      "costMode": "balanced"
    },
    "hooks": {
      "enabled": true,
      "timeout": 5000
    },
    "agents": {
      "maxConcurrent": 3,
      "defaultModel": "claude-3-5-sonnet"
    }
  }
}
```

### Memory Configuration

```javascript
// memory/config.js
module.exports = {
  storage: {
    type: 'sqlite',
    path: '~/.claude/memory/sessions.db'
  },
  compression: {
    enabled: true,
    model: 'claude-3-haiku',
    targetRatio: 0.3
  },
  search: {
    algorithm: 'hybrid', // BM25 + vector
    topK: 5,
    minScore: 0.7
  },
  retention: {
    maxAge: '90d',
    maxSize: '1GB'
  }
};
```

## Advanced Patterns

### Custom Agent System

```javascript
// agents/custom-agent.js
class CustomAgent {
  constructor(config) {
    this.name = config.name;
    this.role = config.role;
    this.tools = config.tools;
  }
  
  async execute(task) {
    // Pre-task validation
    await this.validate(task);
    
    // Execute with error handling
    try {
      const result = await this.performTask(task);
      await this.postProcess(result);
      return result;
    } catch (error) {
      return this.handleError(error, task);
    }
  }
  
  async validate(task) {
    // Check prerequisites
    if (!task.requirements) {
      throw new Error('Task requirements missing');
    }
    
    // Verify permissions
    const hasPermission = await this.checkPermissions(task);
    if (!hasPermission) {
      throw new Error('Insufficient permissions');
    }
  }
}

module.exports = CustomAgent;
```

### Hook Chain Pattern

```javascript
// hooks/chain.js
const hooks = [
  require('./validate-security'),
  require('./check-budget'),
  require('./analyze-impact'),
  require('./log-metrics')
];

module.exports = async (context) => {
  for (const hook of hooks) {
    await hook(context);
  }
};
```

### Skill Composition

```markdown
---
name: fullstack-deploy
description: Complete fullstack deployment workflow
requires:
  - backend-architect
  - frontend-builder
  - devops-engineer
---

# Fullstack Deploy

Orchestrates multiple skills for complete deployment:

1. **Backend**: Build and test API
2. **Frontend**: Build and optimize assets
3. **DevOps**: Deploy infrastructure
4. **QA**: Run smoke tests
5. **Monitor**: Set up observability
```

## Troubleshooting

### Plugin Not Loading

```bash
# Check plugin directory
ls -la ~/.claude/plugins/

# Verify plugin.json exists
cat ~/.claude/plugins/claude-code-toolkit/plugin.json

# Reload plugins
/plugin reload

# Check logs
tail -f ~/.claude/logs/plugins.log
```

### Hook Execution Failures

```bash
# Test hook manually
node ~/.claude/hooks/pre-approve.js

# Enable debug mode
export CLAUDE_DEBUG=hooks
claude code

# Check hook permissions
chmod +x ~/.claude/hooks/*.js
```

### Skill Not Activating

```bash
# List installed skills
/skill list

# Check skill triggers
cat ~/.claude/skills/*/SKILL.md | grep "triggers:"

# Force activation
/skill activate <name> --force

# Clear skill cache
rm -rf ~/.claude/cache/skills/
```

### MCP Server Issues

```bash
# Test MCP server directly
npx @zilliz/claude-context --test

# Check environment variables
echo $MILVUS_URI
echo $OPENAI_API_KEY

# View MCP logs
tail -f ~/.claude/logs/mcp.log

# Restart MCP servers
/mcp restart
```

### Cost Tracking Issues

```bash
# Verify JSONL files exist
ls -la ~/.claude/projects/*.jsonl

# Run diagnostic
ccusage diagnose --verbose

# Check for corrupted entries
ccusage validate --fix

# Export for manual analysis
ccusage export --format csv > usage.csv
```

## Best Practices

### 1. Layered Configuration

```
~/.claude/
├── config.json           # Global settings
├── plugins/              # Installed plugins
├── hooks/                # Lifecycle hooks
│   ├── global/          # Apply to all projects
│   └── project/         # Project-specific
├── skills/              # Installed skills
├── agents/              # Agent definitions
└── memory/              # Session memory
```

### 2. Cost Management

- Use Haiku for routine tasks (80% of work)
- Reserve Sonnet for complex logic
- Use Opus only for critical architecture decisions
- Enable prompt caching for repeated contexts
- Monitor with `ccusage` daily

### 3. Security Hardening

- Always use `agento-patronum` hooks
- Never commit `.env` files
- Use environment variable references
- Enable audit logging
- Regular security reviews with `bouncer`

### 4. Quality Gates

- Pre-commit: Lint + format check
- Pre-approve: Security + cost validation
- Post-task: Quality audit with Bouncer
- Pre-merge: Brooks-lint review

### 5. Memory Management

- Compress sessions with AI summarization
- Regular cleanup of old sessions
- Index for fast retrieval
- Tag by project/feature for context

## Resources

- **Homepage**: https://skillkit.sh
- **Repository**: https://github.com/rohitg00/awesome-claude-code-toolkit
- **Documentation**: https://github.com/rohitg00/awesome-claude-code-toolkit#readme
- **Issues**: https://github.com/rohitg00/awesome-claude-code-toolkit/issues
- **License**: Apache-2.0
- **Stars**: 1,700+ (16 stars/day)

## Related Tools

- **SkillKit**: 400K+ skills marketplace
- **Claude Mem**: Automatic session memory
- **CCPM**: GitHub Issues project management
- **Chief**: Autonomous loop runner
- **AgentLint**: Repo AI-readiness checker
