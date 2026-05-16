---
name: claude-code-best-practice
description: Master agentic engineering with Claude Code through best practices for agents, commands, skills, workflows, and orchestration patterns
triggers:
  - "how do I structure Claude Code agents"
  - "show me best practices for agentic engineering"
  - "help me create Claude Code commands and skills"
  - "what are orchestration workflows in Claude Code"
  - "configure Claude Code for my project"
  - "set up MCP servers and hooks"
  - "optimize my Claude Code workspace"
  - "implement subagents and workflows"
---

# Claude Code Best Practice

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

Master the transition from "vibe coding" to agentic engineering with comprehensive best practices, implementation patterns, and real-world examples for Claude Code.

## Overview

This project provides battle-tested patterns for building sophisticated AI coding workflows using Claude Code's agent system. It covers subagents, commands, skills, orchestration workflows, MCP servers, hooks, and advanced features like agent teams and scheduled tasks.

## Core Concepts

### Project Structure

```
your-project/
├── .claude/
│   ├── agents/           # Subagent definitions
│   │   └── <name>.md
│   ├── commands/         # Custom slash commands
│   │   └── <name>.md
│   ├── skills/           # Skill packages
│   │   └── <name>/
│   │       └── SKILL.md
│   ├── hooks/            # Lifecycle hooks
│   │   └── <hook-name>.sh
│   ├── rules/            # Project-specific rules
│   │   └── <name>.md
│   └── settings.json     # Configuration
├── .mcp.json             # MCP server config
└── CLAUDE.md             # Project memory/context
```

### Subagents

**Purpose**: Specialized AI agents focused on specific domains.

**Location**: `.claude/agents/<name>.md`

**Structure**:
```markdown
---
name: backend-api
description: Expert in FastAPI, database design, and REST APIs
model: claude-3-7-sonnet-20250219
---

You are a backend API specialist...

## Capabilities
- Design RESTful APIs
- Optimize database queries
- Handle authentication patterns

## Guidelines
- Always validate input
- Use async/await patterns
- Include proper error handling
```

**Invocation**:
```bash
# Direct invocation
@backend-api help me design a user authentication endpoint

# From command
/api-review  # Can delegate to @backend-api
```

**Best Practices**:
- Keep agents focused (single responsibility)
- Define clear capabilities and constraints
- Use appropriate models (Sonnet for complex, Haiku for simple)
- Include domain-specific guidelines

### Commands

**Purpose**: Reusable workflows triggered by `/command-name`.

**Location**: `.claude/commands/<name>.md`

**Basic Command**:
```markdown
---
name: api-review
description: Review API design for REST best practices
---

Review the current API implementation:

1. Check endpoint naming conventions
2. Verify HTTP method usage
3. Validate response status codes
4. Check error handling patterns
5. Assess documentation completeness

Focus on files in `src/api/` and `src/routes/`.
```

**Orchestration Command** (delegates to agents):
```markdown
---
name: full-stack-review
description: Comprehensive review using specialized agents
---

Orchestrate a complete codebase review:

## Step 1: Backend Review
@backend-api review all API endpoints in `src/api/` for:
- Security vulnerabilities
- Performance issues
- Best practices compliance

## Step 2: Frontend Review
@frontend-expert analyze components in `src/components/` for:
- Accessibility issues
- Performance optimizations
- React best practices

## Step 3: Integration Check
@integration-agent verify:
- API contract matches frontend usage
- Error handling is consistent
- Loading states are handled

## Final Report
Synthesize findings into a prioritized action plan.
```

**With Parameters**:
```markdown
---
name: generate-crud
description: Generate CRUD endpoints for a model
---

Generate complete CRUD operations for: {model_name}

1. Create `src/models/{model_name}.py`
2. Create `src/routes/{model_name}.py`
3. Add database migrations
4. Generate OpenAPI documentation
5. Create integration tests

Use existing patterns from `src/models/user.py` as reference.
```

### Skills

**Purpose**: Installable expertise packages with tools and context.

**Location**: `.claude/skills/<skill-name>/SKILL.md`

**Example FastAPI Skill**:
```markdown
---
name: fastapi-expert
description: Expert FastAPI development patterns and best practices
triggers:
  - "create a FastAPI endpoint"
  - "help with FastAPI routing"
  - "optimize FastAPI performance"
---

# FastAPI Expert Skill

## Installation

```bash
pip install fastapi uvicorn sqlalchemy pydantic
```

## Basic Endpoint Pattern

```python
from fastapi import FastAPI, HTTPException, Depends
from pydantic import BaseModel
from typing import List

app = FastAPI()

class Item(BaseModel):
    id: int
    name: str
    description: str | None = None

@app.get("/items/{item_id}", response_model=Item)
async def get_item(item_id: int):
    item = await fetch_item(item_id)
    if not item:
        raise HTTPException(status_code=404, detail="Item not found")
    return item

@app.post("/items", response_model=Item, status_code=201)
async def create_item(item: Item):
    created = await save_item(item)
    return created
```

## Dependency Injection

```python
from fastapi import Depends
from sqlalchemy.orm import Session

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@app.get("/users")
async def get_users(db: Session = Depends(get_db)):
    return db.query(User).all()
```

## Background Tasks

```python
from fastapi import BackgroundTasks

def send_email(email: str, message: str):
    # Email sending logic
    pass

@app.post("/send-notification")
async def notify(email: str, background_tasks: BackgroundTasks):
    background_tasks.add_task(send_email, email, "Welcome!")
    return {"message": "Notification queued"}
```

## Best Practices
- Use Pydantic models for validation
- Leverage dependency injection
- Implement proper error handling
- Use async/await for I/O operations
- Document with OpenAPI metadata
```

### Workflows (Orchestration)

**Purpose**: Multi-step processes that coordinate multiple agents.

**Example Weather Orchestrator**:
```markdown
---
name: weather-orchestrator
description: Fetch and analyze weather data using specialized agents
---

# Weather Analysis Workflow

## Phase 1: Data Collection
@data-fetcher retrieve current weather data for {city} using:
- Temperature
- Humidity
- Wind speed
- Forecast

## Phase 2: Analysis
@data-analyst analyze the weather data for:
- Trends over next 48 hours
- Unusual patterns
- Alerts or warnings

## Phase 3: Recommendations
@recommendation-engine generate actionable advice:
- Outdoor activity suitability
- Clothing recommendations
- Travel considerations

## Phase 4: Report
Compile findings into a user-friendly summary with:
- Current conditions
- 48-hour forecast
- Personalized recommendations
```

**Invocation**:
```bash
/weather-orchestrator city="San Francisco"
```

### MCP Servers

**Purpose**: Extend Claude with external tools and data sources.

**Configuration** (`.mcp.json`):
```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/workspace"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "brave-search": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-brave-search"],
      "env": {
        "BRAVE_API_KEY": "${BRAVE_API_KEY}"
      }
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "POSTGRES_CONNECTION_STRING": "${DATABASE_URL}"
      }
    }
  }
}
```

**Usage in Agents**:
```markdown
Use the GitHub MCP server to:
- List open pull requests
- Create issues
- Search repositories
- Analyze commit history
```

### Hooks

**Purpose**: Execute scripts at key lifecycle points.

**Available Hooks**:
- `BeforeChat` - Before processing user message
- `AfterChat` - After generating response
- `BeforeToolCall` - Before executing tool
- `AfterToolCall` - After tool execution
- `BeforeFileEdit` - Before modifying files
- `AfterFileEdit` - After file changes
- `WorktreeCreate` - When creating worktree
- `WorktreeRemove` - When removing worktree

**Example** (`.claude/hooks/BeforeFileEdit.sh`):
```bash
#!/bin/bash
# Backup files before editing

FILE_PATH="$1"
BACKUP_DIR=".claude/backups"

mkdir -p "$BACKUP_DIR"
timestamp=$(date +%Y%m%d_%H%M%S)
cp "$FILE_PATH" "$BACKUP_DIR/$(basename "$FILE_PATH").$timestamp.bak"

echo "Backed up $FILE_PATH"
```

Make executable:
```bash
chmod +x .claude/hooks/BeforeFileEdit.sh
```

### Settings

**Configuration** (`.claude/settings.json`):
```json
{
  "model": "claude-3-7-sonnet-20250219",
  "maxTokens": 8000,
  "temperature": 0.7,
  "statusline": {
    "left": ["model", "tokens"],
    "right": ["time", "cost"]
  },
  "permissions": {
    "read": ["**/*"],
    "write": ["src/**", "tests/**"],
    "execute": ["scripts/**/*.sh"]
  },
  "autoMode": {
    "enabled": false,
    "confirmThreshold": 10
  },
  "outputStyle": "markdown",
  "fastMode": false,
  "isolation": "none"
}
```

**Global Settings** (`~/.claude/settings.json`):
```json
{
  "defaultModel": "claude-3-7-sonnet-20250219",
  "apiKey": "${ANTHROPIC_API_KEY}",
  "theme": "dark",
  "telemetry": false
}
```

### Memory (CLAUDE.md)

**Purpose**: Persistent project context and preferences.

**Example**:
```markdown
# Project: E-commerce API

## Tech Stack
- FastAPI 0.104
- PostgreSQL 15
- Redis for caching
- Stripe for payments

## Architecture Decisions
- Use repository pattern for data access
- Implement CQRS for orders
- JWT for authentication
- Event-driven for notifications

## Code Style
- Use async/await throughout
- Type hints required
- Max line length: 100
- Sort imports with isort

## Testing Strategy
- Pytest for all tests
- Minimum 80% coverage
- Integration tests for API endpoints
- Mock external services

## Common Commands
- `make dev` - Start development server
- `make test` - Run test suite
- `make migrate` - Apply database migrations
- `make lint` - Run linters

## Never Do
- Commit secrets to repository
- Skip migration files
- Use print() for logging
- Deploy without tests passing
```

## Advanced Features

### Agent Teams

**Enable** (environment variable):
```bash
export CLAUDE_AGENT_TEAMS=true
claude
```

**Team Configuration**:
```markdown
# In .claude/agents/team-lead.md
---
name: team-lead
description: Coordinates multiple specialized agents
team:
  - backend-api
  - frontend-expert
  - database-admin
  - security-auditor
---

I coordinate specialized agents to solve complex problems.

When given a task, I:
1. Break it into domain-specific subtasks
2. Delegate to appropriate specialists
3. Synthesize their outputs
4. Ensure consistency across domains
```

### Scheduled Tasks

**Create Scheduled Command** (`.claude/commands/daily-health-check.md`):
```markdown
---
name: daily-health-check
description: Daily codebase health assessment
schedule: "0 9 * * *"  # 9 AM daily
---

Perform daily health check:

1. Run test suite and report failures
2. Check for dependency vulnerabilities
3. Analyze code quality metrics
4. Review open pull requests
5. Check CI/CD pipeline status

Email summary to: ${TEAM_EMAIL}
```

**Manual Scheduling**:
```bash
/schedule daily-health-check "0 9 * * *"
```

### Worktrees (Git Isolation)

**Create Isolated Environment**:
```bash
claude --worktree feature/new-api
```

**In Command**:
```markdown
---
name: safe-refactor
description: Refactor in isolated worktree
isolation: worktree
---

Safely refactor {component} by:
1. Creating clean worktree
2. Running comprehensive tests
3. Applying changes incrementally
4. Validating each step
```

### Ultrareview

**Start Deep Code Review**:
```bash
/ultrareview src/
```

**Track Progress**:
```bash
/ultrareview --status
```

**In settings.json**:
```json
{
  "ultrareview": {
    "depth": "thorough",
    "focus": ["security", "performance", "maintainability"],
    "excludePatterns": ["*.test.ts", "*.spec.ts"]
  }
}
```

## Real-World Patterns

### Full-Stack Feature Development

**Command** (`.claude/commands/new-feature.md`):
```markdown
---
name: new-feature
description: Scaffold complete full-stack feature
---

Create feature: {feature_name}

## Backend (@backend-api)
1. Create data model in `src/models/{feature_name}.py`
2. Create API routes in `src/routes/{feature_name}.py`
3. Add database migration
4. Write integration tests

## Frontend (@frontend-expert)
1. Create React components in `src/components/{feature_name}/`
2. Add API client methods
3. Implement form validation
4. Add unit tests

## Integration (@integration-agent)
1. Verify API contract
2. Test error scenarios
3. Validate loading states
4. Check accessibility

## Documentation
1. Update OpenAPI spec
2. Add component Storybook stories
3. Update README.md
```

### Security Audit Workflow

**Command** (`.claude/commands/security-audit.md`):
```markdown
---
name: security-audit
description: Comprehensive security review
---

## Phase 1: Dependency Check
@security-auditor scan dependencies:
```bash
npm audit
pip-audit
```

## Phase 2: Code Analysis
@security-auditor review for:
- SQL injection vulnerabilities
- XSS attack vectors
- Authentication bypass risks
- Exposed secrets

## Phase 3: Configuration Review
@security-auditor check:
- CORS policies
- CSP headers
- Environment variable usage
- API rate limiting

## Phase 4: Report
Generate security report with:
- Critical findings (immediate action)
- Medium priority issues
- Recommendations
- Remediation steps
```

### Database Migration Pattern

**Agent** (`.claude/agents/database-admin.md`):
```markdown
---
name: database-admin
description: Database schema and migration expert
model: claude-3-7-sonnet-20250219
---

Expert in database design and migrations.

## Migration Template

```python
"""
Migration: {description}
Created: {timestamp}
"""

from alembic import op
import sqlalchemy as sa

def upgrade():
    op.create_table(
        'table_name',
        sa.Column('id', sa.Integer, primary_key=True),
        sa.Column('created_at', sa.DateTime, nullable=False),
    )
    
    # Add indexes
    op.create_index('ix_table_name_id', 'table_name', ['id'])

def downgrade():
    op.drop_index('ix_table_name_id')
    op.drop_table('table_name')
```

## Safety Checks
- Always provide downgrade path
- Test migration on copy of production data
- Add indexes for foreign keys
- Use transactions where possible
```

## Troubleshooting

### Agent Not Found
```bash
# List available agents
ls .claude/agents/

# Verify agent syntax
@agent-name --help
```

### Command Fails
```bash
# Check command definition
cat .claude/commands/command-name.md

# View command output
/command-name --verbose
```

### MCP Server Issues
```bash
# Test MCP server
npx @modelcontextprotocol/server-github --help

# Check environment variables
echo $GITHUB_TOKEN

# Validate .mcp.json
jq . .mcp.json
```

### Permission Errors
```json
{
  "permissions": {
    "read": ["**/*"],
    "write": ["src/**", "!src/config/**"],
    "execute": ["scripts/**/*.sh"]
  }
}
```

### Hook Not Executing
```bash
# Verify executable permission
chmod +x .claude/hooks/BeforeFileEdit.sh

# Test hook directly
.claude/hooks/BeforeFileEdit.sh "test.txt"

# Check hook output
cat ~/.claude/logs/hooks.log
```

## Best Practices Summary

1. **Agents**: Keep focused, use appropriate models, define clear boundaries
2. **Commands**: Document parameters, handle errors, provide examples
3. **Skills**: Include real code, cover common patterns, troubleshoot
4. **Workflows**: Break into phases, coordinate agents, validate steps
5. **MCP**: Secure credentials, test servers, handle failures
6. **Hooks**: Keep fast, handle errors, log actions
7. **Settings**: Version control, document choices, test changes
8. **Memory**: Keep updated, be specific, include decisions

## Resources

- [Official Documentation](https://code.claude.com/docs)
- [Best Practices Guide](https://github.com/shanraisshan/claude-code-best-practice/tree/main/best-practice)
- [Implementation Examples](https://github.com/shanraisshan/claude-code-best-practice/tree/main/implementation)
- [Community Patterns](https://github.com/shanraisshan/claude-code-best-practice)
