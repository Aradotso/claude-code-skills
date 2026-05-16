---
name: claude-skills-library-expert
description: Expert in using and contributing to the claude-skills library — 268+ AI coding agent skills and plugins for Claude Code, Codex, Gemini CLI, Cursor, and more.
triggers:
  - "how do I install claude skills"
  - "show me how to use claude code plugins"
  - "convert skills to cursor format"
  - "create a new skill for claude code"
  - "install marketing skills for gemini cli"
  - "how do I use personas with orchestration"
  - "troubleshoot skill installation"
  - "convert all skills to aider format"
---

# Claude Skills Library Expert

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

Expert in using, installing, and contributing to the `alirezarezvani/claude-skills` repository — the most comprehensive open-source library of AI coding agent skills with 268+ production-ready skills across 9 domains (engineering, marketing, product, compliance, C-level advisory, etc.).

## What This Project Does

The claude-skills library provides modular instruction packages (skills) that give AI coding agents domain expertise they don't have out of the box. Each skill includes:

- **SKILL.md** — structured instructions, workflows, decision frameworks
- **Python tools** — 373 CLI scripts (stdlib-only, zero pip installs)
- **Reference docs** — templates, checklists, domain-specific knowledge

**Multi-tool support:** Works natively with Claude Code, OpenAI Codex, Gemini CLI, and converts to 9+ other tools (Cursor, Aider, Windsurf, Kilo Code, OpenCode, Augment, Antigravity, Hermes Agent).

## Installation

### Claude Code (Recommended)

```bash
# Add the marketplace
/plugin marketplace add alirezarezvani/claude-skills

# Install by domain
/plugin install engineering-skills@claude-code-skills
/plugin install marketing-skills@claude-code-skills
/plugin install product-skills@claude-code-skills
/plugin install c-level-skills@claude-code-skills

# Install individual skills
/plugin install skill-security-auditor@claude-code-skills
/plugin install playwright-pro@claude-code-skills
/plugin install self-improving-agent@claude-code-skills
```

### Gemini CLI

```bash
git clone https://github.com/alirezarezvani/claude-skills.git
cd claude-skills
./scripts/gemini-install.sh

# Activate a skill
> activate_skill(name="senior-architect")
```

### OpenAI Codex

```bash
npx agent-skills-cli add alirezarezvani/claude-skills --agent codex
```

### OpenClaw

```bash
bash <(curl -s https://raw.githubusercontent.com/alirezarezvani/claude-skills/main/scripts/openclaw-install.sh)
```

### Manual Installation

```bash
git clone https://github.com/alirezarezvani/claude-skills.git
# Copy skill folders to:
# - Claude Code: ~/.claude/skills/
# - Codex: ~/.codex/skills/
# - Gemini CLI: ~/.gemini/skills/
```

## Converting Skills to Other Tools

The library supports 12 AI coding tools via conversion scripts.

### Convert All Skills to All Tools

```bash
cd claude-skills
./scripts/convert.sh --tool all
```

This generates tool-specific outputs in:
- `.cursor/` — Cursor `.mdc` rules
- `.aider/` — Aider `CONVENTIONS.md`
- `.kilocode/` — Kilo Code rules
- `.windsurf/` — Windsurf skills
- `.opencode/` — OpenCode skills
- `.augment/` — Augment rules
- `~/.gemini/antigravity/` — Antigravity skills
- `~/.hermes/` — Hermes Agent skills

### Install Converted Skills into a Project

```bash
# Install with confirmation
./scripts/install.sh --tool cursor --target /path/to/project

# Force install without confirmation
./scripts/install.sh --tool aider --target . --force
```

### Tool-Specific Conversion

```bash
# Cursor
./scripts/convert.sh --tool cursor
./scripts/install.sh --tool cursor --target .

# Aider
./scripts/convert.sh --tool aider
./scripts/install.sh --tool aider --target .

# Windsurf
./scripts/convert.sh --tool windsurf
./scripts/install.sh --tool windsurf --target .

# Hermes Agent
python scripts/sync-hermes-skills.py --verbose
```

### Verify Installation

```bash
# Cursor
find .cursor/rules -name "*.mdc" | wc -l  # Should show 268

# Aider
cat CONVENTIONS.md  # Should contain all skills

# Windsurf
ls -1 .windsurf/skills/ | wc -l  # Should show 268
```

## Key Skills and Domains

### Engineering (32 core + 40 POWERFUL)

```bash
# Core skills
/plugin install senior-architect@claude-code-skills
/plugin install senior-frontend@claude-code-skills
/plugin install senior-backend@claude-code-skills
/plugin install devops-engineer@claude-code-skills
/plugin install playwright-pro@claude-code-skills

# POWERFUL tier
/plugin install agent-designer@claude-code-skills
/plugin install rag-architect@claude-code-skills
/plugin install database-designer@claude-code-skills
/plugin install ci-cd-pipeline-builder@claude-code-skills
/plugin install skill-security-auditor@claude-code-skills
```

### Marketing (44 skills)

```bash
/plugin install marketing-skills@claude-code-skills

# Individual pods
# Content (8), SEO (5), CRO (6), Channels (6), Growth (4), Intelligence (4), Sales (2)
```

### Product (13 skills)

```bash
/plugin install product-skills@claude-code-skills

# Includes: product-manager, agile-po, ux-researcher, ui-design, analytics-tracking, etc.
```

### C-Level Advisory (28 skills)

```bash
/plugin install c-level-skills@claude-code-skills

# Full C-suite: CTO, CMO, CFO, CRO, CPO, COO, CHRO, CISO, GC, CAIO
```

## Using Personas

Personas are pre-configured agent identities with curated skill loadouts and distinct communication styles.

### Available Personas

- **startup-cto** — Architecture decisions, tech stack, team building
- **growth-marketer** — Content-led growth, launch strategy, channel optimization
- **solo-founder** — Cross-domain, MVP building, wearing all hats

### Install a Persona

```bash
# Claude Code
cp agents/personas/startup-cto.md ~/.claude/agents/

# For other tools, convert first
./scripts/convert.sh --tool cursor
```

### Use a Persona

```markdown
# In your conversation with the AI agent:
"I want you to act as the Startup CTO persona. Review my architecture decisions for this new feature."
```

## Orchestration Patterns

Orchestration coordinates personas, skills, and agents across domain boundaries.

### Four Patterns

**1. Solo Sprint** — Switch personas across project phases
```markdown
Week 1-2: startup-cto + aws-solution-architect
Week 3-4: growth-marketer + launch-strategy
Week 5-6: solo-founder + analytics-tracking
```

**2. Domain Deep-Dive** — One persona + multiple stacked skills
```bash
# Example: Security audit
/activate persona startup-cto
/use skill-security-auditor
/use aws-solution-architect
/use compliance-automation
```

**3. Multi-Agent Handoff** — Personas review each other's output
```markdown
1. startup-cto designs architecture
2. devops-engineer reviews deployment strategy
3. skill-security-auditor audits the plan
```

**4. Skill Chain** — Sequential skills, no persona needed
```bash
# Content pipeline
/use copywriting → /use seo-content-optimizer → /use email-sequence
```

See `orchestration/ORCHESTRATION.md` for full protocol and examples.

## Creating a New Skill

### Skill Structure

```
my-skill/
├── SKILL.md          # Main instruction file
├── scripts/          # Python tools (optional)
│   └── tool.py
├── references/       # Documentation (optional)
│   └── guide.md
└── templates/        # Templates (optional)
    └── template.txt
```

### SKILL.md Template

```yaml
---
name: my-skill
description: One-line description of what this skill does
triggers:
  - "phrase a user might say"
  - "another trigger phrase"
  - "how do I..."
  - "show me..."
---

# My Skill

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## Overview
What this skill does and when to use it.

## Key Capabilities
- Capability 1
- Capability 2

## Usage
How to use this skill with code examples.

## Workflows
Step-by-step workflows.

## Troubleshooting
Common issues and solutions.
```

### Python Tool Template

```python
#!/usr/bin/env python3
"""
Description of what this tool does.
"""
import sys
import json
import os

def main():
    # Use environment variables for configuration
    config = os.getenv('MY_CONFIG', 'default_value')
    
    # Your tool logic here
    result = {"status": "success"}
    
    # Output JSON for easy parsing
    print(json.dumps(result, indent=2))

if __name__ == "__main__":
    main()
```

### Add Your Skill

```bash
# 1. Create skill directory
mkdir -p my-skill/scripts

# 2. Write SKILL.md and scripts/tool.py

# 3. Test locally
cp -r my-skill ~/.claude/skills/

# 4. (Optional) Submit PR to claude-skills repo
git checkout -b add-my-skill
git add my-skill/
git commit -m "Add my-skill"
git push origin add-my-skill
```

## Common Patterns

### Using Python Tools from Skills

All 373 Python tools are stdlib-only and can run anywhere:

```python
#!/usr/bin/env python3
# Example: engineering-team/playwright-pro/scripts/generate-test.py
import sys
import json

def generate_test(page_url, selectors):
    """Generate a Playwright test from URL and selectors."""
    test_code = f"""
import {{ test, expect }} from '@playwright/test';

test('test {page_url}', async ({{ page }}) => {{
  await page.goto('{page_url}');
  await expect(page.locator('{selectors[0]}')).toBeVisible();
}});
"""
    return test_code

if __name__ == "__main__":
    url = sys.argv[1] if len(sys.argv) > 1 else "https://example.com"
    selectors = sys.argv[2:] if len(sys.argv) > 2 else ["body"]
    print(generate_test(url, selectors))
```

**Usage:**
```bash
python engineering-team/playwright-pro/scripts/generate-test.py \
  "https://myapp.com/login" \
  "input[name='email']" \
  "button[type='submit']"
```

### Stacking Skills

```bash
# Claude Code
/use senior-architect
/use aws-solution-architect
/use skill-security-auditor

# Then ask:
"Review my architecture for a multi-tenant SaaS with SOC 2 compliance requirements."
```

### Using Skills with Code Generation

```markdown
User: "Generate a REST API with CRUD operations for a blog post resource."

AI (with senior-backend skill):
- Analyzes requirements
- Follows RESTful patterns from skill
- Generates code with proper validation
- Includes security best practices
- Adds tests and documentation
```

## Troubleshooting

### Skills Not Loading

**Claude Code:**
```bash
# Check installation
ls ~/.claude/skills/

# Reinstall
/plugin uninstall my-skill@claude-code-skills
/plugin install my-skill@claude-code-skills
```

**Gemini CLI:**
```bash
# Check installation
ls ~/.gemini/skills/

# Reinstall
./scripts/gemini-install.sh
```

### Conversion Script Fails

```bash
# Ensure you're in the repo root
cd /path/to/claude-skills

# Check Python version (requires 3.7+)
python3 --version

# Run with verbose output
./scripts/convert.sh --tool cursor --verbose
```

### Python Tools Not Executable

```bash
# Make scripts executable
find . -name "*.py" -type f -exec chmod +x {} \;

# Or run with python3 explicitly
python3 engineering-team/playwright-pro/scripts/generate-test.py
```

### Persona Not Activating

```bash
# Ensure persona file is in correct location
# Claude Code
cp agents/personas/startup-cto.md ~/.claude/agents/

# For other tools, convert first
./scripts/convert.sh --tool cursor

# Verify file exists
cat ~/.claude/agents/startup-cto.md
```

### Skill Security Concerns

Use the `skill-security-auditor` before installing third-party skills:

```bash
# Install the security auditor
/plugin install skill-security-auditor@claude-code-skills

# Audit a skill directory
/use skill-security-auditor
"Audit the skills in /path/to/untrusted-skills/"
```

## Configuration

### Environment Variables

Skills respect these environment variables:

```bash
# General
export CLAUDE_SKILLS_PATH="$HOME/.claude/skills"
export GEMINI_SKILLS_PATH="$HOME/.gemini/skills"

# For tools with API integration
export OPENAI_API_KEY="your-key-here"
export ANTHROPIC_API_KEY="your-key-here"

# Project-specific
export PROJECT_ROOT="/path/to/project"
export DOCS_OUTPUT_DIR="./docs"
```

### Tool-Specific Configuration

**Cursor** (`.cursor/config.json`):
```json
{
  "skills_path": ".cursor/rules",
  "auto_load": true
}
```

**Aider** (`.aider.conf.yml`):
```yaml
conventions: CONVENTIONS.md
auto_commits: true
```

**Windsurf** (`.windsurf/config.yml`):
```yaml
skills_dir: .windsurf/skills
auto_activate: true
```

## Advanced Usage

### Creating Skill Bundles

```bash
# Create a custom bundle
mkdir -p my-bundle/skills
cp -r engineering-team/senior-architect my-bundle/skills/
cp -r product-team/product-manager my-bundle/skills/
cp -r marketing-skill/content-pod/* my-bundle/skills/

# Install bundle
cp -r my-bundle/skills/* ~/.claude/skills/
```

### Automating Skill Updates

```bash
#!/bin/bash
# scripts/update-skills.sh
cd ~/claude-skills
git pull origin main
./scripts/convert.sh --tool all
./scripts/install.sh --tool cursor --target ~/my-project --force
echo "Skills updated!"
```

### Using Skills in CI/CD

```yaml
# .github/workflows/skill-audit.yml
name: Security Audit
on: [push]
jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Audit skills
        run: |
          git clone https://github.com/alirezarezvani/claude-skills.git
          python claude-skills/engineering/skill-security-auditor/scripts/audit.py .
```

## Reference Links

- **Repository:** https://github.com/alirezarezvani/claude-skills
- **Documentation:** README.md in repo root
- **Orchestration Guide:** `orchestration/ORCHESTRATION.md`
- **Persona Template:** `agents/personas/TEMPLATE.md`
- **Author:** https://alirezarezvani.medium.com/

## Quick Reference

| Command | Description |
|---------|-------------|
| `./scripts/convert.sh --tool all` | Convert all skills to all tools |
| `./scripts/install.sh --tool cursor --target .` | Install converted skills |
| `/plugin install <skill>@claude-code-skills` | Install single skill (Claude Code) |
| `./scripts/gemini-install.sh` | Install for Gemini CLI |
| `python scripts/sync-hermes-skills.py` | Sync to Hermes Agent |
| `find .cursor/rules -name "*.mdc"` | Verify Cursor installation |

---

**MIT License** — 15,045 stars — 268 production-ready skills — 373 Python tools — 12 AI coding tools supported
