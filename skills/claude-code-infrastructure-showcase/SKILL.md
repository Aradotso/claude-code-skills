---
name: claude-code-infrastructure-showcase
description: Expert in Claude Code infrastructure patterns - auto-activating skills, hooks, agents, and dev docs systems
triggers:
  - "how do I set up Claude Code skills"
  - "make my skills activate automatically"
  - "create a hook for Claude Code"
  - "build a specialized agent"
  - "set up dev docs pattern"
  - "implement skill auto-activation"
  - "configure skill-rules.json"
  - "create modular skills with progressive disclosure"
---

# Claude Code Infrastructure Showcase

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

Expert guidance for implementing production-tested Claude Code infrastructure including auto-activating skills, hooks, agents, and dev docs systems extracted from 6 months of real-world TypeScript microservices development.

## What This Project Provides

The claude-code-infrastructure-showcase is a **reference library** (not a working application) providing:

- **Auto-activating skills** via hook-based triggers
- **Modular skill pattern** following 500-line rule with progressive disclosure
- **Specialized agents** for complex development tasks
- **Dev docs system** that preserves context across resets
- **Production-tested patterns** from real enterprise development

**Key Philosophy:** Copy what you need into your own projects. Everything is designed to be extracted and customized.

## Repository Structure

```
.claude/
├── skills/                          # Production skills
│   ├── backend-dev-guidelines/      # Node.js/Express/Prisma patterns
│   ├── frontend-dev-guidelines/     # React/TypeScript/MUI v7
│   ├── skill-developer/             # Meta-skill for creating skills
│   ├── route-tester/                # API route testing
│   ├── error-tracking/              # Sentry integration
│   └── skill-rules.json            # Skill activation config
├── hooks/                           # Automation hooks
│   ├── skill-activation-prompt.*    # Auto-suggest skills (ESSENTIAL)
│   ├── post-tool-use-tracker.sh     # Track file operations (ESSENTIAL)
│   ├── tsc-check.sh                 # TypeScript validation (optional)
│   └── trigger-build-resolver.sh    # Build error resolution (optional)
├── agents/                          # Specialized agents
│   ├── code-architecture-reviewer.md
│   ├── refactor-planner.md
│   └── ... 8 more agents
└── commands/                        # Slash commands
    └── dev-docs.md

dev/active/                          # Dev docs examples
```

## Core Concepts

### 1. Auto-Activation System

**Problem:** Skills don't activate automatically - you must remember to use them.

**Solution:** Hook + configuration system:

```json
// .claude/skills/skill-rules.json
{
  "skills": [
    {
      "name": "backend-dev-guidelines",
      "activationRules": {
        "pathPatterns": [
          "**/src/routes/**",
          "**/src/controllers/**",
          "**/src/services/**"
        ],
        "promptKeywords": [
          "api", "endpoint", "route", "controller",
          "service", "repository", "database"
        ],
        "autoActivate": true
      }
    }
  ]
}
```

**How it works:**
1. `skill-activation-prompt` hook runs on every user prompt
2. Analyzes prompt text and open file paths
3. Matches against `skill-rules.json` patterns
4. Auto-suggests relevant skills

### 2. Modular Skills (500-Line Rule)

Large skills hit context limits. Use progressive disclosure:

```
skill-name/
├── SKILL.md                    # <500 lines - overview + navigation
└── resources/
    ├── routing-patterns.md     # <500 lines each
    ├── testing-guide.md
    └── error-handling.md
```

**Example from backend-dev-guidelines:**

```markdown
<!-- SKILL.md -->
# Backend Development Guidelines

## Quick Navigation
- [Routing Patterns](resources/routing-patterns.md)
- [Controller Layer](resources/controllers.md)
- [Service Layer](resources/services.md)

## When to Load Resources
- Working on routes? → Load routing-patterns.md
- Writing business logic? → Load services.md
```

### 3. Dev Docs Pattern

Preserve context across Claude Code resets:

```
dev/active/[task-name]/
├── [task]-plan.md       # Strategic overview
├── [task]-context.md    # Key decisions & files
└── [task]-tasks.md      # Checklist format
```

## Installation & Setup

### Phase 1: Essential Hooks (15 minutes)

**Step 1: Copy hook files**

```bash
# Copy to your project
cp .claude/hooks/skill-activation-prompt.* your-project/.claude/hooks/
cp .claude/hooks/post-tool-use-tracker.sh your-project/.claude/hooks/
chmod +x your-project/.claude/hooks/*.sh
```

**Step 2: Install Node.js dependencies (for skill-activation-prompt)**

```bash
cd your-project/.claude/hooks
npm init -y
npm install fs path
```

**Step 3: Update settings.json**

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "command": "node",
        "args": [".claude/hooks/skill-activation-prompt.js"],
        "timeout": 5000
      }
    ],
    "PostToolUse": [
      {
        "command": ".claude/hooks/post-tool-use-tracker.sh",
        "timeout": 3000
      }
    ]
  }
}
```

### Phase 2: Add Your First Skill (10 minutes)

**Step 1: Choose a skill**

Available skills:
- `backend-dev-guidelines` - Express/Prisma/TypeScript APIs
- `frontend-dev-guidelines` - React/MUI/TypeScript
- `skill-developer` - Creating new skills
- `route-tester` - Testing authenticated routes
- `error-tracking` - Sentry integration

**Step 2: Copy skill directory**

```bash
# Example: backend skill
cp -r .claude/skills/backend-dev-guidelines your-project/.claude/skills/
```

**Step 3: Create/update skill-rules.json**

```bash
# Copy base config
cp .claude/skills/skill-rules.json your-project/.claude/skills/

# Edit to match YOUR project structure
```

**Step 4: Customize path patterns**

```json
{
  "skills": [
    {
      "name": "backend-dev-guidelines",
      "activationRules": {
        "pathPatterns": [
          // Customize these to YOUR project structure
          "**/api/routes/**",           // Your routes directory
          "**/api/controllers/**",       // Your controllers
          "**/services/**"               // Your services
        ],
        "promptKeywords": [
          "api", "endpoint", "route"
        ],
        "autoActivate": true
      }
    }
  ]
}
```

### Phase 3: Test Activation (5 minutes)

**Test 1: Path-based activation**

```bash
# Open a file matching your pathPatterns
code your-project/api/routes/users.ts

# Ask Claude a question - skill should auto-suggest
"How should I structure this route?"
```

**Test 2: Keyword-based activation**

```bash
# Ask with trigger keywords
"I need to create a new API endpoint for user registration"
# Should suggest backend-dev-guidelines
```

## Key Patterns & Examples

### Pattern 1: Creating a Modular Skill

**Structure:**

```bash
.claude/skills/my-skill/
├── SKILL.md                      # Main entry point
└── resources/
    ├── topic-1.md
    ├── topic-2.md
    └── topic-3.md
```

**SKILL.md template:**

```markdown
---
name: my-skill
description: Brief one-line description
triggers:
  - "trigger phrase 1"
  - "trigger phrase 2"
---

# My Skill

> Skill by [ara.so](https://ara.so)

## Overview
High-level guidance (keep under 500 lines)

## Quick Navigation
- [Topic 1](resources/topic-1.md) - Use when...
- [Topic 2](resources/topic-2.md) - Use when...

## When to Load Resources
**Working on X?** → Load topic-1.md
**Debugging Y?** → Load topic-2.md
```

### Pattern 2: Skill Rules Configuration

**Full example:**

```json
{
  "skills": [
    {
      "name": "backend-dev-guidelines",
      "activationRules": {
        "pathPatterns": [
          "**/src/routes/**",
          "**/src/controllers/**",
          "**/src/services/**",
          "**/src/repositories/**",
          "**/prisma/**"
        ],
        "promptKeywords": [
          "api", "endpoint", "route", "controller",
          "service", "repository", "database", "prisma",
          "validation", "error handling", "middleware"
        ],
        "autoActivate": true,
        "priority": 1
      }
    },
    {
      "name": "frontend-dev-guidelines",
      "activationRules": {
        "pathPatterns": [
          "**/src/components/**",
          "**/src/pages/**",
          "**/src/hooks/**",
          "**/src/contexts/**"
        ],
        "promptKeywords": [
          "component", "react", "mui", "datagrid",
          "form", "validation", "state", "hook"
        ],
        "autoActivate": true,
        "priority": 1
      }
    }
  ],
  "globalSettings": {
    "maxConcurrentSkills": 2,
    "skillSuggestionMode": "auto"
  }
}
```

### Pattern 3: Creating a Hook

**Example: Simple post-commit hook**

```bash
#!/bin/bash
# .claude/hooks/post-commit.sh

# Log commit info
echo "Commit completed: $(git log -1 --oneline)"

# Update dev docs
if [ -f "dev/active/current-task/tasks.md" ]; then
  echo "✓ Remember to update dev/active/current-task/tasks.md"
fi

exit 0
```

**Register in settings.json:**

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "command": ".claude/hooks/post-commit.sh",
        "timeout": 3000
      }
    ]
  }
}
```

### Pattern 4: Creating a Specialized Agent

**Template:**

```markdown
---
name: my-agent
description: Agent for [specific task]
---

# My Agent

## Purpose
[What this agent does]

## When to Use
- [Scenario 1]
- [Scenario 2]

## Capabilities
- [Capability 1]
- [Capability 2]

## Workflow

### Step 1: [Phase Name]
[Instructions]

### Step 2: [Phase Name]
[Instructions]

## Output Format
[Expected deliverables]

## Example Usage
[Concrete example]
```

**Real example from showcase:**

```markdown
---
name: code-architecture-reviewer
description: Reviews code for architectural consistency and best practices
---

# Code Architecture Reviewer

## Purpose
Analyze code against established architectural patterns and provide
actionable improvement recommendations.

## When to Use
- Before merging large features
- After major refactoring
- When onboarding new patterns

## Workflow

### Step 1: Load Context
- Read architecture documentation
- Review relevant skills (backend-dev-guidelines, etc.)
- Understand project structure

### Step 2: Analyze Code
- Check layer separation (routes → controllers → services)
- Verify error handling patterns
- Review naming conventions

### Step 3: Generate Report
- Inconsistencies found
- Recommended changes
- Priority ranking
```

### Pattern 5: Dev Docs Workflow

**Create with slash command:**

```bash
# In Claude Code
/dev-docs "implement user authentication"
```

**Generates:**

```markdown
# dev/active/user-authentication/

## user-authentication-plan.md
### Overview
Implement JWT-based authentication with refresh tokens

### Success Criteria
- Users can register/login
- Tokens expire after 15 minutes
- Refresh tokens work

## user-authentication-context.md
### Key Files
- src/routes/auth.ts
- src/services/authService.ts
- src/middleware/authenticate.ts

### Important Decisions
- Using JWT (not sessions) for scalability
- Refresh tokens stored in httpOnly cookies

## user-authentication-tasks.md
- [ ] Create auth routes
- [ ] Implement JWT generation
- [ ] Add middleware
- [ ] Write tests
```

## Configuration Reference

### settings.json Structure

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "command": "node",
        "args": [".claude/hooks/skill-activation-prompt.js"],
        "timeout": 5000
      }
    ],
    "PostToolUse": [
      {
        "command": ".claude/hooks/post-tool-use-tracker.sh",
        "timeout": 3000
      }
    ],
    "Stop": [
      {
        "command": ".claude/hooks/tsc-check.sh",
        "timeout": 10000,
        "continueOnError": true
      }
    ]
  },
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/project"]
    }
  }
}
```

### skill-rules.json Schema

```typescript
interface SkillRulesConfig {
  skills: SkillRule[];
  globalSettings?: {
    maxConcurrentSkills?: number;
    skillSuggestionMode?: 'auto' | 'manual';
  };
}

interface SkillRule {
  name: string;
  activationRules: {
    pathPatterns: string[];      // Glob patterns
    promptKeywords: string[];    // Trigger words
    autoActivate: boolean;
    priority?: number;           // Higher = preferred
  };
}
```

## Available Skills

### backend-dev-guidelines

**Purpose:** Node.js/Express/Prisma/TypeScript API development

**Resources:**
- `routing-patterns.md` - Route structure and conventions
- `controllers.md` - Controller layer patterns
- `services.md` - Business logic organization
- `repositories.md` - Data access layer
- `error-handling.md` - Error management
- `validation.md` - Input validation patterns
- `testing.md` - Testing strategies
- `authentication.md` - Auth patterns
- `database.md` - Prisma best practices
- `logging.md` - Logging and monitoring
- `middleware.md` - Custom middleware
- `api-design.md` - REST API conventions

**Activation:**
```json
{
  "pathPatterns": [
    "**/src/routes/**",
    "**/src/controllers/**",
    "**/src/services/**"
  ],
  "promptKeywords": ["api", "endpoint", "route", "service"]
}
```

### frontend-dev-guidelines

**Purpose:** React/TypeScript/MUI v7 development

**Resources:**
- `component-patterns.md` - Component structure
- `state-management.md` - State patterns
- `mui-patterns.md` - MUI v7 best practices
- `datagrid-patterns.md` - DataGrid usage
- `form-patterns.md` - Form handling
- `hooks-patterns.md` - Custom hooks
- `routing.md` - React Router patterns
- `testing.md` - Component testing
- `performance.md` - Optimization
- `accessibility.md` - A11y guidelines
- `error-handling.md` - Error boundaries

**Activation:**
```json
{
  "pathPatterns": [
    "**/src/components/**",
    "**/src/pages/**",
    "**/src/hooks/**"
  ],
  "promptKeywords": ["component", "react", "mui", "form"]
}
```

### skill-developer

**Purpose:** Meta-skill for creating new skills

**Resources:**
- `skill-structure.md` - Skill anatomy
- `progressive-disclosure.md` - 500-line rule
- `skill-rules.md` - Activation configuration
- `resource-organization.md` - Resource file patterns
- `testing-skills.md` - Skill testing
- `maintenance.md` - Updating skills
- `examples.md` - Real skill examples

**Use when:** Creating new skills or modifying existing ones

## Available Agents

### code-architecture-reviewer
Reviews code for architectural consistency

**Usage:**
```bash
@code-architecture-reviewer Review the new user service for architectural issues
```

### refactor-planner
Creates detailed refactoring strategies

**Usage:**
```bash
@refactor-planner Plan refactoring of the auth system to use dependency injection
```

### frontend-error-fixer
Debugs frontend errors with React DevTools context

**Usage:**
```bash
@frontend-error-fixer Fix the "Cannot read property 'map' of undefined" error in UserList
```

### documentation-architect
Generates comprehensive documentation

**Usage:**
```bash
@documentation-architect Generate API documentation for the user service
```

### auth-route-tester
Tests authenticated API endpoints

**Usage:**
```bash
@auth-route-tester Test POST /api/users/register with sample data
```

### auto-error-resolver
Automatically fixes TypeScript compilation errors

**Usage:**
```bash
@auto-error-resolver Fix type errors in src/services/userService.ts
```

## Common Workflows

### Workflow 1: Setting Up for a New Project

```bash
# 1. Copy essential hooks
mkdir -p .claude/hooks
cp showcase/.claude/hooks/skill-activation-prompt.* .claude/hooks/
cp showcase/.claude/hooks/post-tool-use-tracker.sh .claude/hooks/
chmod +x .claude/hooks/*.sh

# 2. Install dependencies
cd .claude/hooks && npm init -y && npm install fs path

# 3. Copy relevant skill
mkdir -p .claude/skills
cp -r showcase/.claude/skills/backend-dev-guidelines .claude/skills/

# 4. Create skill-rules.json
cat > .claude/skills/skill-rules.json << 'EOF'
{
  "skills": [
    {
      "name": "backend-dev-guidelines",
      "activationRules": {
        "pathPatterns": ["**/src/**"],
        "promptKeywords": ["api", "service"],
        "autoActivate": true
      }
    }
  ]
}
EOF

# 5. Update settings.json
cat > .claude/settings.json << 'EOF'
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "command": "node",
        "args": [".claude/hooks/skill-activation-prompt.js"],
        "timeout": 5000
      }
    ],
    "PostToolUse": [
      {
        "command": ".claude/hooks/post-tool-use-tracker.sh",
        "timeout": 3000
      }
    ]
  }
}
EOF
```

### Workflow 2: Creating a Custom Skill

```bash
# 1. Use skill-developer skill
# In Claude Code:
# "I want to create a new skill for GraphQL development"

# 2. Create structure
mkdir -p .claude/skills/graphql-guidelines/resources

# 3. Create SKILL.md
cat > .claude/skills/graphql-guidelines/SKILL.md << 'EOF'
---
name: graphql-guidelines
description: GraphQL API development patterns with Apollo Server
triggers:
  - "graphql schema"
  - "resolver implementation"
  - "graphql query"
---

# GraphQL Guidelines

> Skill by [ara.so](https://ara.so)

## Overview
Patterns for GraphQL API development

## Resources
- [Schema Design](resources/schema-design.md)
- [Resolver Patterns](resources/resolvers.md)
EOF

# 4. Add to skill-rules.json
# Edit .claude/skills/skill-rules.json to add activation rules
```

### Workflow 3: Using Dev Docs

```bash
# 1. Start new task
# In Claude Code:
/dev-docs "refactor authentication to use refresh tokens"

# 2. Claude creates:
# dev/active/auth-refactor/
#   auth-refactor-plan.md
#   auth-refactor-context.md
#   auth-refactor-tasks.md

# 3. Work on task, update tasks.md as you go

# 4. Before context reset:
/dev-docs-update

# 5. After reset, load context:
# "Load dev/active/auth-refactor/ and continue"
```

### Workflow 4: Adding a Hook

```bash
# 1. Create hook file
cat > .claude/hooks/pre-commit-check.sh << 'EOF'
#!/bin/bash
# Run linting before commits

echo "Running pre-commit checks..."

# Run eslint
npm run lint
if [ $? -ne 0 ]; then
  echo "❌ Linting failed"
  exit 1
fi

echo "✅ Pre-commit checks passed"
exit 0
EOF

# 2. Make executable
chmod +x .claude/hooks/pre-commit-check.sh

# 3. Add to settings.json
# Under "hooks" → "Stop" array
```

## Troubleshooting

### Skills Not Activating

**Problem:** Skills don't auto-suggest when expected

**Check:**
1. Verify hook is registered in settings.json
2. Check skill-rules.json path patterns match your structure
3. Test hook manually:
   ```bash
   node .claude/hooks/skill-activation-prompt.js
   ```
4. Review hook logs in `.claude/hooks/activation-log.json`

**Common issues:**
- Path patterns too specific (use broader globs)
- Keywords not matching (add more trigger words)
- Hook timeout too short (increase to 5000ms)

### Hooks Not Running

**Problem:** Hooks don't execute at all

**Check:**
1. Hook files are executable:
   ```bash
   chmod +x .claude/hooks/*.sh
   ```
2. settings.json syntax is valid (use JSON validator)
3. Hook paths are relative to project root
4. Node.js dependencies installed for .js hooks

**Debug:**
```bash
# Test hook directly
.claude/hooks/post-tool-use-tracker.sh

# Check for errors
echo $?  # Should be 0
```

### Context Limit Errors

**Problem:** "Context limit exceeded" when loading skill

**Solution:** Skill is too large, needs modularization

```bash
# 1. Check main SKILL.md line count
wc -l .claude/skills/my-skill/SKILL.md

# 2. If >500 lines, split into resources
mkdir -p .claude/skills/my-skill/resources

# 3. Move sections to resource files
# Keep only overview + navigation in SKILL.md

# 4. Update references
# Link to resources/topic.md instead of inline content
```

### Dev Docs Not Persisting

**Problem:** Context still lost after using dev docs

**Check:**
1. Files saved in `dev/active/[task-name]/`
2. All three files created (plan, context, tasks)
3. Files committed to git (not in .gitignore)

**Best practice:**
```bash
# Explicitly load dev docs after reset
"Load all files in dev/active/auth-refactor/ and summarize the current state"
```

### Hook Performance Issues

**Problem:** Hooks causing slowdowns

**Solutions:**

1. **Optimize JavaScript hooks:**
   ```javascript
   // Cache file reads
   const cache = new Map();
   
   function readFileWithCache(path) {
     if (cache.has(path)) return cache.get(path);
     const content = fs.readFileSync(path, 'utf8');
     cache.set(path, content);
     return content;
   }
   ```

2. **Set appropriate timeouts:**
   ```json
   {
     "timeout": 3000,  // 3s for simple hooks
     "timeout": 10000  // 10s for complex hooks
   }
   ```

3. **Use `continueOnError`:**
   ```json
   {
     "command": ".claude/hooks/heavy-check.sh",
     "continueOnError": true  // Don't block on failure
   }
   ```

## Environment Variables

Hooks and skills reference these environment vars:

```bash
# Project paths
PROJECT_ROOT=/path/to/project
CLAUDE_CONFIG_DIR=/path/to/project/.claude

# Skill configuration
SKILL_RULES_PATH=$CLAUDE_CONFIG_DIR/skills/skill-rules.json

# Logging
HOOK_LOG_LEVEL=info  # debug, info, warn, error
ACTIVATION_LOG=$CLAUDE_CONFIG_DIR/hooks/activation-log.json

# Optional: Custom tool paths
TSC_PATH=./node_modules/.bin/tsc
ESLINT_PATH=./node_modules/.bin/eslint
```

## Best Practices

### Skill Development

1. **Keep main SKILL.md under 500 lines**
   - Use resources/ for deep dives
   - Include navigation section
   - Explain when to load each resource

2. **Write clear activation rules**
   ```json
   {
     "pathPatterns": [
       "**/specific/path/**",  // Specific first
       "**/broad/path/**"      // Broad as fallback
     ]
   }
   ```

3. **Use real code examples**
   - From your actual project
   - Show before/after
   - Include error cases

4. **Version control skills**
   - Commit to git
   - Document changes
   - Tag releases

### Hook Development

1. **Keep hooks fast (<3 seconds)**
   - Cache expensive operations
   - Use timeouts appropriately
   - Fail gracefully

2. **Handle errors explicitly**
   ```bash
   #!/bin/bash
   set -e  # Exit on error
   
   if ! command -v node &> /dev/null; then
     echo "❌ Node.js not found"
     exit 1
   fi
   ```

3. **Log useful information**
   ```javascript
   console.error(JSON.stringify({
     timestamp: new Date().toISOString(),
     hook: 'skill-activation',
     trigger: 'pathMatch',
     skill: 'backend-dev-guidelines'
   }));
   ```

### Agent Usage

1. **Be specific in requests**
   ```bash
   # Good
   @refactor-planner Create plan to extract email sending logic into a service

   # Too vague
   @refactor-planner Improve the code
   ```

2. **Provide context**
   - Load relevant files first
   - Explain constraints
   - Mention related issues

3. **Review agent output**
   - Agents suggest, you decide
   - Verify recommendations
   - Adapt to your project

## Integration Checklist

```markdown
## Phase 1: Core Setup
- [ ] Copy skill-activation-prompt hook
- [ ] Copy post-tool-use-tracker hook
- [ ] Install Node.js dependencies in .claude/hooks
- [ ] Update settings.json with hooks
- [ ] Test hooks execute (check logs)

## Phase 2: First Skill
- [ ] Choose relevant skill
- [ ] Copy skill directory
- [ ] Create skill-rules.json
- [ ] Customize path patterns for your project
- [ ] Test skill activation (open matching file)

## Phase 3: Validation
- [ ] Skill auto-suggests on file open
- [ ] Skill auto-suggests on relevant prompt
- [ ] No performance degradation
- [ ] Hooks log properly
- [ ] Skills load resources correctly

## Phase 4: Enhancement (Optional)
- [ ] Add more skills as needed
- [ ] Copy useful agents
- [ ] Add slash commands
- [ ] Customize Stop hooks
- [ ] Set up dev docs pattern
```

## Advanced Usage

### Custom Skill Rules Logic

Extend skill-activation-prompt.js:

```javascript
// .claude/hooks/skill-activation-prompt.js

// Add custom activation logic
function shouldActivateSkill(skill, context) {
  const { filePath, promptText, timestamp } = context;
  
  // Time-based activation
  if (skill.activationRules.timeOfDay) {
    const hour = new Date(timestamp).getHours();
    if (hour < 9 || hour > 17) return false;
  }
  
  // Git branch-based activation
  if (skill.activationRules.branches) {
    const branch = getCurrentGitBranch();
    if (!skill.activationRules.branches.includes(branch)) {
      return false;
    }
  }
  
  // File size-based activation
  if (skill.activationRules.maxFileSize) {
    const size = fs.statSync(filePath).size;
    if (size > skill.activationRules.maxFileSize) {
      return false;
    }
  }
  
  return true;
}
```

### Conditional Resource Loading

In SKILL.md, add logic for resource loading:

```markdown
## Progressive Loading Strategy

**For small changes (<50 lines):**
Just use this main skill file.

**For new features:**
1. Load [architecture overview](resources/architecture.md)
2. Load relevant pattern resource

**For debugging:**
1. Load [error-handling](resources/error-handling.md)
2. Load [logging](resources/logging.md)

**For refactoring:**
Load ALL resources for comprehensive guidance.
```

### Multi-Project Skill Sharing

Share skills across projects:

```bash
# 1. Create shared skills repository
mkdir ~/claude-skills
cp -r project1/.claude/skills/* ~/claude-skills/

# 2. Symlink in each project
ln -s ~/claude-skills project2/.claude/skills

# 3. Use environment-specific skill-rules.json
# project2/.claude/skills/skill-rules.json
{
  "skills": [
    {
      "name": "backend-dev-guidelines",
      "activationRules": {
        "pathPatterns": [
          "**/project2-specific/path/**"
        ]
      }
    }
  ]
}
```

## Resources

- **Main Repository:** https://github.com/diet103/claude-code-infrastructure-showcase
- **Original Reddit Post:** [Claude Code is a Beast](https://www.reddit.com/r/ClaudeAI/comments/1oivjvm/claude_code_is_a_beast_tips_from_6_months_of/)
- **Skills Guide:** `.claude/skills/README.md`
- **Hooks Guide:** `.claude/hooks/README.md`
- **Agents Guide:** `.claude/agents/README.md`
- **Integration Guide:** `CLAUDE_INTEGRATION_GUIDE.md`

## License

MIT License - Use freely in personal or commercial projects.
