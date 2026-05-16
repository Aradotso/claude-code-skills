---
name: awesome-claude-skills
description: Curated list of Claude Skills, resources, and tools for customizing Claude AI workflows and agentic coding
triggers:
  - "find Claude skills for"
  - "browse available Claude skills"
  - "how do I create a Claude skill"
  - "install Claude skills"
  - "what skills are available for Claude"
  - "customize Claude with skills"
  - "build a custom Claude skill"
  - "search for Claude Code skills"
---

# Awesome Claude Skills

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

This skill provides comprehensive knowledge of the Claude Skills ecosystem, including official and community skills, creation guidelines, installation methods, and best practices for customizing Claude AI workflows.

## What Are Claude Skills?

Claude Skills teach Claude how to perform tasks in a repeatable way. They are specialized folders containing instructions, scripts, and resources that Claude dynamically discovers and loads when relevant to tasks.

### Progressive Disclosure Architecture

Skills use efficient loading:
1. **Metadata loading** (~100 tokens): Claude scans available Skills to identify relevant matches
2. **Full instructions** (<5k tokens): Load when Claude determines the Skill applies
3. **Bundled resources**: Files and executable code load only as needed

## Installation Methods

### Claude.ai Web Interface

1. Navigate to [Settings > Capabilities](https://claude.ai/settings/capabilities)
2. Enable Skills toggle
3. Browse available skills or upload custom skills
4. For Team/Enterprise: Admin must enable Skills organization-wide first

### Claude Code CLI

```bash
# Install skills from marketplace
/plugin marketplace add anthropics/skills

# Install from local directory
/plugin add /path/to/skill-directory

# Install community skill collections
/plugin marketplace add obra/superpowers-marketplace
```

### Claude API

```python
import anthropic
import os

client = anthropic.Client(api_key=os.environ.get("ANTHROPIC_API_KEY"))

# List available skills
response = client.skills.list()

# Use skills in conversation
message = client.messages.create(
    model="claude-3-5-sonnet-20241022",
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "Create a Word document with quarterly report"}
    ],
    skills=["docx"]
)
```

## Skill Structure

### Basic Skill Format

```
my-skill/
├── SKILL.md          # Main skill file with frontmatter
├── scripts/          # Optional executable scripts
│   └── helper.py
└── resources/        # Optional supporting files
    └── template.json
```

### SKILL.md Template

```yaml
---
name: my-custom-skill
description: Brief description for skill discovery (keep concise)
triggers:
  - "phrase users might say"
  - "another trigger phrase"
---

# Detailed Instructions

Claude will read these instructions when the skill is activated.

## Usage
Explain how to use this skill...

## Examples
Provide clear examples...
```

## Creating Skills

### Using skill-creator (Recommended)

```
# In Claude Code or Claude.ai
User: "Use the skill-creator to help me build a skill for [your task]"

# Follow interactive prompts to generate skill structure
```

### Manual Creation Example

```yaml
---
name: api-documentation-generator
description: Generate comprehensive API documentation from code
triggers:
  - "generate API docs"
  - "document my API endpoints"
  - "create API reference"
---

# API Documentation Generator

This skill helps create comprehensive API documentation from code.

## Usage

1. Provide the API code or endpoint definitions
2. Specify the documentation format (OpenAPI, Markdown, etc.)
3. Claude generates structured documentation

## Example

For a Flask API:

```python
from flask import Flask, jsonify, request

app = Flask(__name__)

@app.route('/api/users', methods=['GET'])
def get_users():
    """Retrieve all users"""
    return jsonify({"users": []})

@app.route('/api/users/<int:user_id>', methods=['GET'])
def get_user(user_id):
    """Retrieve a specific user by ID"""
    return jsonify({"user_id": user_id})
```

Generated documentation includes:
- Endpoint descriptions
- HTTP methods
- Parameters
- Response formats
- Example requests/responses
```

## Key Official Skills

### Document Manipulation

```python
# docx skill - Word document handling
# Example usage through Claude
"Create a Word document with the following sections: Introduction, Methods, Results"

# pdf skill - PDF operations
"Extract tables from this PDF and convert to Excel format"

# xlsx skill - Excel spreadsheet handling
"Create a spreadsheet with sales data and generate pivot tables"

# pptx skill - PowerPoint presentations
"Create a 10-slide presentation about quarterly results with charts"
```

### Development Skills

```python
# frontend-design skill
"Create a landing page with bold design choices, avoid generic AI aesthetics"

# web-artifacts-builder skill
"Build an interactive dashboard using React, Tailwind, and shadcn/ui"

# mcp-builder skill
"Create an MCP server that integrates with the GitHub API"

# webapp-testing skill
"Test my local webapp at http://localhost:3000 using Playwright"
```

### Creative Skills

```python
# algorithmic-art skill
"Create generative art using p5.js with flow fields and particle systems"

# canvas-design skill
"Design a poster for a tech conference in PNG format"

# slack-gif-creator skill
"Create an animated GIF for Slack showing a progress bar animation"
```

## Community Skills

### Installing Superpowers Collection

```bash
# Install the comprehensive skills library
/plugin marketplace add obra/superpowers-marketplace

# Available commands after installation
/brainstorm          # Interactive brainstorming
/write-plan         # Create detailed plans
/execute-plan       # Execute planned tasks
/skills-search      # Search available skills
```

### Popular Community Skills

```bash
# iOS Simulator automation
/plugin add https://github.com/conorluddy/ios-simulator-skill

# Web fuzzing with ffuf
/plugin add https://github.com/jthack/ffuf_claude_skill

# Browser automation with Playwright
/plugin add https://github.com/lackeyjb/playwright-skill

# D3.js visualizations
/plugin add https://github.com/chrisvoncsefalvay/claude-d3js-skill

# Scientific computing
/plugin add https://github.com/K-Dense-AI/claude-scientific-skills

# Security auditing
/plugin add https://github.com/trailofbits/skills

# Expo app development
/plugin add https://github.com/expo/skills
```

## Best Practices

### Skill Design

1. **Keep descriptions concise** - Frontmatter description used for discovery
2. **Use clear triggers** - Natural phrases users would say
3. **Provide working examples** - Show real code, not placeholders
4. **Document dependencies** - List required packages/tools
5. **Version with git tags** - Use semantic versioning

### Security Considerations

```yaml
# ✅ Good - Use environment variables
api_key: ${API_KEY}
database_url: ${DATABASE_URL}

# ❌ Bad - Never hardcode secrets
api_key: "sk-1234567890abcdef"
```

### Testing Skills

```bash
# Test locally before sharing
/plugin add /path/to/my-skill

# Verify activation
"Use my-skill to [trigger phrase]"

# Check skill loading
# Skills should load metadata first, then full instructions when relevant
```

## Configuration

### Skills in Claude Desktop

Add to `claude_desktop_config.json`:

```json
{
  "skills": {
    "enabled": true,
    "paths": [
      "/path/to/custom/skills",
      "~/.claude/skills"
    ]
  }
}
```

### Skills in Projects

Add to `.claude/config.json`:

```json
{
  "skills": [
    "frontend-design",
    "webapp-testing",
    "custom-skill"
  ],
  "skill_paths": [
    "./skills"
  ]
}
```

## Common Patterns

### Conditional Skill Activation

```yaml
---
name: python-testing-skill
description: Generate Python tests with pytest
triggers:
  - "write tests for"
  - "create pytest tests"
---

# Detect when to activate
This skill activates when:
- Python files are present
- User requests test creation
- pytest is mentioned or available
```

### Multi-File Skills

```
complex-skill/
├── SKILL.md
├── scripts/
│   ├── analyzer.py
│   ├── generator.py
│   └── validator.py
└── resources/
    ├── templates/
    │   ├── template1.json
    │   └── template2.json
    └── config.yaml
```

Reference in SKILL.md:

```markdown
## Available Scripts

- `scripts/analyzer.py` - Analyzes input data
- `scripts/generator.py` - Generates output
- `scripts/validator.py` - Validates results

## Templates

Located in `resources/templates/`:
- `template1.json` - For API responses
- `template2.json` - For configuration files
```

## Troubleshooting

### Skill Not Activating

```bash
# Check if skill is loaded
# In Claude Code:
/plugin list

# Verify skill metadata is correct
# Check SKILL.md frontmatter syntax

# Ensure triggers match user intent
# Add more natural trigger phrases
```

### Skill Conflicts

```yaml
# If multiple skills trigger, be specific
# Use unique, descriptive names
name: project-specific-testing  # ✅ Good
name: testing                   # ❌ Too generic
```

### Performance Issues

```markdown
# Keep metadata small (<100 tokens in frontmatter)
# Progressive disclosure: Don't load everything at once
# Split large skills into focused sub-skills
```

## Resources

- **Repository**: https://github.com/travisvn/awesome-claude-skills
- **Official Skills**: https://github.com/anthropics/skills
- **API Documentation**: https://platform.claude.com/docs/en/api/beta/skills
- **Community Collections**: 
  - https://github.com/obra/superpowers
  - https://github.com/obra/superpowers-skills
- **Skill Creator Tool**: Built-in `skill-creator` skill
- **Conversion Tool**: https://github.com/yusufkaraaslan/Skill_Seekers
