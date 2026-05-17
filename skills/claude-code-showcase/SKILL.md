---
name: claude-code-showcase
description: Configure Claude Code projects with hooks, skills, agents, commands, and GitHub Actions for automated workflows and AI-powered development.
triggers:
  - "set up Claude Code configuration"
  - "create a skill for Claude"
  - "add hooks to my project"
  - "configure MCP servers"
  - "set up automated PR reviews"
  - "create a custom agent"
  - "add slash commands"
  - "configure GitHub Actions for Claude"
---

# Claude Code Project Configuration

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## What This Project Does

`claude-code-showcase` provides a comprehensive reference for configuring Claude Code projects with:

- **Skills** - Domain knowledge documents that teach Claude your patterns (testing, GraphQL, UI components, etc.)
- **Agents** - Specialized AI assistants that run automatically (code reviewers, ticket handlers)
- **Hooks** - Scripts that run at key points (auto-format, run tests, block main branch edits)
- **Commands** - Custom slash commands for common workflows (`/onboard`, `/pr-review`, `/ticket`)
- **MCP Servers** - External integrations (JIRA, GitHub, Slack, databases)
- **GitHub Actions** - Scheduled automation (docs sync, quality reviews, dependency audits)

## Installation & Setup

### 1. Initialize Claude Code Directory

```bash
mkdir -p .claude/{agents,commands,hooks,skills,rules}
touch .claude/settings.json
touch CLAUDE.md
```

### 2. Add Project Memory (CLAUDE.md)

Create `CLAUDE.md` in your project root:

```markdown
# My Project

## Stack
- React 18, TypeScript 5.x, Node.js 20+
- GraphQL API (Apollo Client/Server)
- Testing: Jest + React Testing Library

## Key Commands
- `npm test` - Run tests
- `npm run lint` - ESLint
- `npm run typecheck` - TypeScript validation
- `npm run build` - Production build

## Code Style
- TypeScript strict mode enabled
- Prefer interfaces over types
- No `any` - use `unknown` instead
- Components must handle loading/error states

## Key Directories
- `src/components/` - React components
- `src/api/` - GraphQL schema and resolvers
- `src/hooks/` - Custom React hooks
- `tests/` - Test files (colocated with source)
```

### 3. Configure Settings & Hooks

Create `.claude/settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "[ \"$(git branch --show-current)\" != \"main\" ] || { echo '{\"block\": true, \"message\": \"Cannot edit files on main branch. Create a feature branch first.\"}' >&2; exit 2; }",
            "timeout": 5
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write {{file}}",
            "timeout": 10,
            "suppressOutput": true
          }
        ]
      },
      {
        "matcher": "Edit.*\\.test\\.(ts|tsx|js|jsx)",
        "hooks": [
          {
            "type": "command",
            "command": "npm test -- --findRelatedTests {{file}} --passWithNoTests",
            "timeout": 60
          }
        ]
      }
    ],
    "UserPromptSubmit": [
      {
        "matcher": ".*",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/skill-eval.sh",
            "timeout": 5
          }
        ]
      }
    ]
  },
  "allowedCommands": [
    "npm",
    "npx",
    "git",
    "node",
    "cat",
    "grep",
    "find"
  ],
  "environment": {
    "NODE_ENV": "development"
  }
}
```

## Creating Skills

Skills are domain knowledge documents that teach Claude your patterns and conventions.

### Skill File Structure

```
.claude/skills/
├── README.md
├── testing-patterns/
│   └── SKILL.md
├── graphql-schema/
│   └── SKILL.md
└── core-components/
    └── SKILL.md
```

### Example: Testing Patterns Skill

`.claude/skills/testing-patterns/SKILL.md`:

```markdown
---
name: testing-patterns
description: Jest and React Testing Library patterns for this project. Use when writing tests, creating mocks, or following TDD workflow.
---

# Testing Patterns

## Test Structure

Use the AAA pattern (Arrange, Act, Assert):

\`\`\`typescript
import { render, screen, waitFor } from '@testing-library/react';
import { userEvent } from '@testing-library/user-event';

describe('LoginForm', () => {
  it('submits credentials when form is valid', async () => {
    // Arrange
    const mockOnSubmit = jest.fn();
    render(<LoginForm onSubmit={mockOnSubmit} />);
    
    // Act
    await userEvent.type(screen.getByLabelText(/email/i), 'user@example.com');
    await userEvent.type(screen.getByLabelText(/password/i), 'password123');
    await userEvent.click(screen.getByRole('button', { name: /sign in/i }));
    
    // Assert
    await waitFor(() => {
      expect(mockOnSubmit).toHaveBeenCalledWith({
        email: 'user@example.com',
        password: 'password123'
      });
    });
  });
});
\`\`\`

## Mock Factories

Create factory functions for reusable mocks:

\`\`\`typescript
// tests/factories/user.ts
export const getMockUser = (overrides = {}) => ({
  id: '123',
  email: 'test@example.com',
  name: 'Test User',
  role: 'user',
  ...overrides
});
\`\`\`

## GraphQL Mocking

Use MSW for GraphQL mocks:

\`\`\`typescript
import { graphql } from 'msw';
import { setupServer } from 'msw/node';

const server = setupServer(
  graphql.query('GetUser', (req, res, ctx) => {
    return res(
      ctx.data({
        user: getMockUser()
      })
    );
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
\`\`\`

## Coverage Requirements

- All components must have tests
- Minimum 80% coverage for new code
- Test happy path + error states + loading states
```

### Example: GraphQL Schema Skill

`.claude/skills/graphql-schema/SKILL.md`:

```markdown
---
name: graphql-schema
description: GraphQL schema design patterns and resolver conventions for this project. Use when creating types, queries, mutations, or resolvers.
---

# GraphQL Schema Patterns

## Type Definitions

\`\`\`graphql
type User {
  id: ID!
  email: String!
  name: String!
  createdAt: DateTime!
  updatedAt: DateTime!
}

input CreateUserInput {
  email: String!
  name: String!
  password: String!
}

type CreateUserPayload {
  user: User
  errors: [UserError!]
}
\`\`\`

## Resolver Pattern

\`\`\`typescript
// src/api/resolvers/user.ts
import { GraphQLError } from 'graphql';

export const userResolvers = {
  Query: {
    user: async (_, { id }, { dataSources, user }) => {
      if (!user) {
        throw new GraphQLError('Not authenticated', {
          extensions: { code: 'UNAUTHENTICATED' }
        });
      }
      
      return dataSources.userAPI.getUser(id);
    }
  },
  
  Mutation: {
    createUser: async (_, { input }, { dataSources }) => {
      try {
        const user = await dataSources.userAPI.createUser(input);
        return { user, errors: [] };
      } catch (error) {
        return {
          user: null,
          errors: [{ message: error.message, field: 'email' }]
        };
      }
    }
  }
};
\`\`\`

## Error Handling

- Use `GraphQLError` with extension codes
- Return errors in payload for mutations
- Common codes: `UNAUTHENTICATED`, `FORBIDDEN`, `BAD_USER_INPUT`, `INTERNAL_SERVER_ERROR`
```

## Creating Agents

Agents are specialized AI assistants that run automatically or on demand.

### Example: Code Review Agent

`.claude/agents/code-reviewer.md`:

```markdown
# Code Review Agent

You are a code review agent. After code changes are made, perform a thorough review.

## Review Checklist

### TypeScript
- [ ] Strict mode compliance (no `any`, proper null checks)
- [ ] Proper type imports/exports
- [ ] Generic types used appropriately

### Error Handling
- [ ] API calls wrapped in try/catch
- [ ] User-facing errors handled gracefully
- [ ] Loading states implemented
- [ ] Error boundaries for React components

### Testing
- [ ] Tests added for new features
- [ ] Edge cases covered
- [ ] Mocks use factory pattern
- [ ] Tests follow AAA pattern

### GraphQL
- [ ] Mutations return payload with errors
- [ ] Queries handle authentication
- [ ] Proper error codes used

### Performance
- [ ] No unnecessary re-renders
- [ ] API calls properly memoized
- [ ] Images optimized

## Review Process

1. Read changed files
2. Check against each category
3. Provide specific line-level feedback
4. Suggest improvements with code examples
5. Highlight what was done well

Be constructive and specific. Reference exact lines and provide working alternatives.
```

## Creating Commands

Commands are custom slash commands accessible via `/command-name`.

### Example: PR Review Command

`.claude/commands/pr-review.md`:

```markdown
# PR Review Command

**Usage:** `/pr-review [pr-number]`

**Description:** Perform a comprehensive PR review with detailed feedback.

## Process

1. Fetch PR details: `gh pr view {{pr-number}} --json title,body,files`
2. Read all changed files
3. Review against project standards (see `.claude/agents/code-reviewer.md`)
4. Check for:
   - Breaking changes
   - Migration requirements
   - Test coverage
   - Documentation updates
5. Generate review summary with:
   - Overview of changes
   - Issues found (categorized by severity)
   - Suggestions for improvement
   - Approval recommendation

## Output Format

```markdown
## PR Review: {{title}}

### Summary
Brief overview of what changed

### Issues Found

#### 🔴 Critical
- Issue with line reference and fix

#### 🟡 Suggestions
- Improvement ideas

### Test Coverage
- Coverage percentage
- Missing test cases

### Documentation
- Docs that need updating

### Recommendation
✅ Approve with minor changes / ❌ Request changes
```
```

### Example: Ticket Integration Command

`.claude/commands/ticket.md`:

```markdown
# Ticket Command

**Usage:** `/ticket <ticket-id>`

**Description:** Read ticket, implement feature, update ticket status.

## Requirements

- JIRA MCP server configured in `.mcp.json`
- Git branch naming: `feature/TICKET-123-description`

## Workflow

1. Read ticket details via MCP: `jira_get_issue`
2. Analyze requirements and acceptance criteria
3. Check current branch or create new one
4. Implement changes following skills (testing-patterns, graphql-schema, etc.)
5. Run tests and validation
6. Update ticket with progress: `jira_update_issue`
7. Prompt user to create PR

## Implementation Steps

- Parse acceptance criteria into tasks
- For each task:
  - Implement code
  - Write tests
  - Verify functionality
- Add ticket link to commit messages
- Update ticket status to "In Review"
```

## MCP Server Configuration

MCP servers connect Claude to external tools (JIRA, GitHub, databases, etc.).

### .mcp.json Format

Create `.mcp.json` in project root:

```json
{
  "mcpServers": {
    "jira": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-jira"],
      "env": {
        "JIRA_HOST": "${JIRA_HOST}",
        "JIRA_EMAIL": "${JIRA_EMAIL}",
        "JIRA_API_TOKEN": "${JIRA_API_TOKEN}"
      }
    },
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "postgres": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-postgres"],
      "env": {
        "DATABASE_URL": "${DATABASE_URL}"
      }
    }
  }
}
```

Set environment variables in `.env` (gitignored):

```bash
JIRA_HOST=yourcompany.atlassian.net
JIRA_EMAIL=you@company.com
JIRA_API_TOKEN=your_token_here
GITHUB_TOKEN=ghp_your_token_here
DATABASE_URL=postgresql://user:pass@localhost:5432/db
```

## Skill Evaluation Hooks

Automatically suggest relevant skills based on user prompts.

### Create Skill Evaluation Script

`.claude/hooks/skill-eval.sh`:

```bash
#!/bin/bash

PROMPT="$1"
SKILL_DIR=".claude/skills"

# Extract skill keywords from prompt
if echo "$PROMPT" | grep -qi "test\|spec\|jest\|mock"; then
  echo '{"feedback": "💡 Suggested skill: testing-patterns - activate with @testing-patterns"}'
  exit 0
fi

if echo "$PROMPT" | grep -qi "graphql\|query\|mutation\|resolver"; then
  echo '{"feedback": "💡 Suggested skill: graphql-schema - activate with @graphql-schema"}'
  exit 0
fi

if echo "$PROMPT" | grep -qi "component\|ui\|button\|form"; then
  echo '{"feedback": "💡 Suggested skill: core-components - activate with @core-components"}'
  exit 0
fi
```

Make executable:

```bash
chmod +x .claude/hooks/skill-eval.sh
```

### Advanced: Node.js Skill Matcher

`.claude/hooks/skill-eval.js`:

```javascript
#!/usr/bin/env node

const fs = require('fs');
const path = require('path');

const prompt = process.argv[2] || '';
const skillsDir = path.join(__dirname, '../skills');
const rulesPath = path.join(__dirname, 'skill-rules.json');

const rules = JSON.parse(fs.readFileSync(rulesPath, 'utf8'));

const matches = rules
  .filter(rule => {
    const keywordMatch = rule.keywords.some(kw => 
      prompt.toLowerCase().includes(kw.toLowerCase())
    );
    const pathMatch = rule.paths?.some(p => prompt.includes(p)) ?? false;
    return keywordMatch || pathMatch;
  })
  .map(r => r.skill);

if (matches.length > 0) {
  const suggestions = matches.map(s => `@${s}`).join(', ');
  console.log(JSON.stringify({
    feedback: `💡 Suggested skills: ${suggestions}`
  }));
}
```

`.claude/hooks/skill-rules.json`:

```json
[
  {
    "skill": "testing-patterns",
    "keywords": ["test", "spec", "mock", "jest", "coverage"],
    "paths": [".test.", ".spec.", "/__tests__/"]
  },
  {
    "skill": "graphql-schema",
    "keywords": ["graphql", "query", "mutation", "resolver", "schema"],
    "paths": ["/api/", "/graphql/", ".graphql"]
  },
  {
    "skill": "core-components",
    "keywords": ["component", "ui", "button", "form", "input"],
    "paths": ["/components/", "/ui/"]
  }
]
```

## GitHub Actions Integration

### PR Review Automation

`.github/workflows/pr-claude-code-review.yml`:

```yaml
name: Claude Code PR Review

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run Claude Code Review
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          npx @anthropic/claude-code review \
            --pr ${{ github.event.pull_request.number }} \
            --agent .claude/agents/code-reviewer.md \
            --post-comment
```

### Scheduled Documentation Sync

`.github/workflows/scheduled-claude-code-docs-sync.yml`:

```yaml
name: Monthly Docs Sync

on:
  schedule:
    - cron: '0 0 1 * *' # First day of month
  workflow_dispatch:

jobs:
  docs-sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      - name: Sync Documentation
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          npx @anthropic/claude-code task \
            "Review commits from the last 30 days and update documentation to reflect any API changes, new features, or updated patterns. Focus on README.md, API docs, and CLAUDE.md." \
            --auto-commit \
            --commit-message "docs: monthly sync via Claude Code"
      
      - name: Create PR
        uses: peter-evans/create-pull-request@v5
        with:
          title: "docs: monthly documentation sync"
          body: "Automated documentation updates based on recent changes"
          branch: docs/monthly-sync
```

### Weekly Code Quality Review

`.github/workflows/scheduled-claude-code-quality.yml`:

```yaml
name: Weekly Code Quality

on:
  schedule:
    - cron: '0 0 * * 1' # Every Monday
  workflow_dispatch:

jobs:
  quality-review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Random Directory Review
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          DIR=$(find src -type d | shuf -n 1)
          npx @anthropic/claude-code task \
            "Review all files in $DIR for code quality issues: TypeScript strict mode violations, missing error handling, test coverage gaps, performance issues. Fix any issues found." \
            --auto-commit \
            --commit-message "refactor: code quality improvements in $DIR"
```

## Hook Patterns & Best Practices

### Auto-Format on Save

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write {{file}}",
            "suppressOutput": true
          }
        ]
      }
    ]
  }
}
```

### Run Tests on Test File Changes

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit.*\\.test\\.(ts|tsx)",
        "hooks": [
          {
            "type": "command",
            "command": "npm test -- --findRelatedTests {{file}} --passWithNoTests",
            "timeout": 60
          }
        ]
      }
    ]
  }
}
```

### TypeScript Type Checking

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit.*\\.(ts|tsx)",
        "hooks": [
          {
            "type": "command",
            "command": "npx tsc --noEmit --pretty false {{file}}",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

### Block Main Branch Edits

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "[ \"$(git branch --show-current)\" != \"main\" ] || { echo '{\"block\": true, \"message\": \"Create a feature branch first\"}' >&2; exit 2; }",
            "timeout": 5
          }
        ]
      }
    ]
  }
}
```

## Common Patterns

### Multi-Skill Activation

```
@testing-patterns @graphql-schema Create a new GraphQL mutation for updating user profiles with comprehensive tests
```

### Agent + Command Workflow

```
/ticket PROJ-123
```

This activates the ticket agent which:
1. Reads ticket via MCP
2. Implements feature using relevant skills
3. Writes tests
4. Updates ticket status

### Progressive Skill Building

Start with core skills:
1. `testing-patterns` - How you write tests
2. `code-style` - Formatting and conventions
3. `error-handling` - How you handle errors

Add domain skills:
4. `graphql-schema` - API patterns
5. `core-components` - UI component patterns
6. `state-management` - Redux/Zustand patterns

Add workflow skills:
7. `deployment` - How to deploy
8. `database-migrations` - Schema changes

## Troubleshooting

### Hooks Not Running

1. Check hook syntax in `settings.json`
2. Verify command is in `allowedCommands`
3. Check timeout values (default: 30s)
4. Test command manually: `.claude/hooks/script.sh "test prompt"`

### Skills Not Activating

1. Verify frontmatter format (YAML with `---`)
2. Check `description` field includes trigger keywords
3. Use explicit activation: `@skill-name`
4. Review skill-eval hook logs

### MCP Server Connection Issues

1. Verify environment variables are set
2. Test MCP server independently: `npx @anthropic/mcp-jira test`
3. Check `.mcp.json` syntax
4. Review Claude Code logs for connection errors

### GitHub Actions Failing

1. Verify `ANTHROPIC_API_KEY` secret is set
2. Check Claude Code CLI is available: `npx @anthropic/claude-code --version`
3. Test locally first before pushing to CI
4. Review action logs for specific errors

## Resources

- [Official Claude Code Documentation](https://docs.anthropic.com/claude-code)
- [MCP Server Protocol](https://modelcontextprotocol.io)
- [Example Skills Repository](.claude/skills/)
- [GitHub Actions Examples](.github/workflows/)
