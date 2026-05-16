---
name: awesome-claude-code-curation
description: Expert guidance for contributing to and using the Awesome Claude Code repository, a curated collection of Claude Code skills, agents, hooks, and resources.
triggers:
  - how do I contribute to awesome claude code
  - show me awesome claude code repository structure
  - help me add a skill to awesome claude code
  - what are the guidelines for awesome claude code
  - how to submit to awesome claude code list
  - find claude code skills and resources
  - browse awesome claude code repository
  - create a new claude code skill entry
---

# Awesome Claude Code Curation

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## Overview

The [awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code) repository is a curated collection of high-quality resources for Claude Code and other AI coding agents. It includes skills, agents, hooks, status lines, orchestrators, developer tooling, and features for the Claude Code ecosystem.

**Key Focus Areas:**
- Agent skills and capabilities
- Agentic coding workflows
- AI workflow optimization
- Coding assistants and agents
- Claude Code extensions and plugins

## Repository Structure

The repository is currently undergoing reorganization. The traditional Table of Contents structure is being replaced with a new organizational system optimized for both human and AI contributors.

**Current Status:**
- Repository has 43,934+ stars
- 3,756+ forks
- Active community with 307 open issues
- Emphasis on code quality, security, and originality

## Contributing Guidelines

### Submission Standards

When contributing to awesome-claude-code, follow these principles:

1. **Quality Over Quantity**: Submit only high-quality, tested resources
2. **Security First**: No hardcoded credentials or API keys
3. **Originality**: Unique contributions that add real value
4. **Documentation**: Clear, comprehensive explanations
5. **Working Code**: All examples must be functional

### Creating a Skill Entry

When adding a new Claude Code skill:

```python
# Example structure for a skill contribution
skill_entry = {
    "name": "descriptive-kebab-case-name",
    "description": "Clear one-line description of functionality",
    "repository": "github-username/repo-name",
    "language": "primary-language",
    "category": "skills|agents|hooks|tools",
    "topics": ["relevant", "keywords"],
    "stars": "current_star_count"
}
```

### Pull Request Template

```markdown
## Skill/Resource Submission

**Name**: [Resource Name]
**Repository**: [GitHub URL]
**Category**: [Skills/Agents/Hooks/Tools/etc.]

### Description
[Clear description of what this resource does]

### Why This Resource is Awesome
- Unique feature 1
- Unique feature 2
- Proven use cases

### Testing
- [ ] Code examples tested
- [ ] Documentation reviewed
- [ ] No security issues
- [ ] Follows repository guidelines
```

## Finding and Using Resources

### Browsing the Collection

```python
# Example: Searching for specific skills programmatically
import requests
from typing import List, Dict

def search_awesome_claude_code(query: str) -> List[Dict]:
    """
    Search the awesome-claude-code repository for relevant resources.
    
    Args:
        query: Search term (e.g., "python skills", "web scraping")
    
    Returns:
        List of matching resources
    """
    # Use GitHub API to search repository contents
    api_url = "https://api.github.com/repos/hesreallyhim/awesome-claude-code"
    
    headers = {
        "Accept": "application/vnd.github.v3+json",
        "Authorization": f"token {os.environ.get('GITHUB_TOKEN')}"
    }
    
    response = requests.get(f"{api_url}/contents", headers=headers)
    
    if response.status_code == 200:
        contents = response.json()
        # Filter and return relevant items
        return [item for item in contents if query.lower() in item.get('name', '').lower()]
    
    return []
```

### Installing a Skill from the Collection

```bash
# Clone a specific skill repository
git clone https://github.com/username/skill-name.git

# Or use the Claude Code CLI (if available)
claude-code install skill-name

# Or add to your skills directory
mkdir -p ~/.claude/skills
cp SKILL.md ~/.claude/skills/skill-name.md
```

## Common Patterns

### Pattern 1: Skill Discovery

```python
# Find skills by topic
def find_skills_by_topic(topic: str) -> List[str]:
    """
    Discover skills related to a specific topic.
    
    Args:
        topic: Topic keyword (e.g., "web-scraping", "testing")
    """
    topics_mapping = {
        "agent-skills": ["autonomous-tasks", "workflow-automation"],
        "coding-assistant": ["code-generation", "refactoring"],
        "ai-workflows": ["pipeline-optimization", "batch-processing"]
    }
    
    return topics_mapping.get(topic, [])
```

### Pattern 2: Skill Validation

```python
# Validate a skill before submission
def validate_skill(skill_path: str) -> Dict[str, bool]:
    """
    Validate a Claude Code skill file.
    
    Args:
        skill_path: Path to SKILL.md file
    
    Returns:
        Dictionary of validation results
    """
    validations = {
        "has_frontmatter": False,
        "has_description": False,
        "has_examples": False,
        "no_secrets": True
    }
    
    with open(skill_path, 'r') as f:
        content = f.read()
        
        # Check YAML frontmatter
        validations["has_frontmatter"] = content.startswith('---')
        
        # Check for description
        validations["has_description"] = 'description:' in content
        
        # Check for code examples
        validations["has_examples"] = '```' in content
        
        # Check for hardcoded secrets
        secret_patterns = ['api_key = "', 'password = "', 'token = "']
        validations["no_secrets"] = not any(pattern in content for pattern in secret_patterns)
    
    return validations
```

### Pattern 3: Skill Integration

```python
# Integrate multiple skills for a workflow
class ClaudeCodeWorkflow:
    """Orchestrate multiple Claude Code skills."""
    
    def __init__(self):
        self.skills_dir = os.path.expanduser("~/.claude/skills")
        self.loaded_skills = []
    
    def load_skill(self, skill_name: str) -> Dict:
        """Load a skill from the collection."""
        skill_path = os.path.join(self.skills_dir, f"{skill_name}.md")
        
        if not os.path.exists(skill_path):
            raise FileNotFoundError(f"Skill {skill_name} not found")
        
        with open(skill_path, 'r') as f:
            # Parse skill metadata and content
            return {"name": skill_name, "content": f.read()}
    
    def combine_skills(self, skill_names: List[str]) -> str:
        """Combine multiple skills for complex workflows."""
        combined = []
        
        for skill_name in skill_names:
            skill = self.load_skill(skill_name)
            combined.append(skill["content"])
            self.loaded_skills.append(skill_name)
        
        return "\n\n---\n\n".join(combined)
```

## Configuration

### Environment Variables

```bash
# GitHub token for API access (optional, increases rate limits)
export GITHUB_TOKEN="your_github_token_here"

# Claude Code skills directory
export CLAUDE_SKILLS_DIR="$HOME/.claude/skills"

# Custom repository URL (if forked)
export AWESOME_CLAUDE_CODE_REPO="hesreallyhim/awesome-claude-code"
```

### Local Setup

```bash
# Clone the repository
git clone https://github.com/hesreallyhim/awesome-claude-code.git
cd awesome-claude-code

# Keep it updated
git pull origin main

# Create a branch for contributions
git checkout -b add-new-skill
```

## Troubleshooting

### Issue: Can't Find a Specific Skill

**Solution**: The repository is being reorganized. Check:
- Open issues for status updates
- Recent commits for structural changes
- Contact maintainers via GitHub issues

### Issue: Outdated Skill Information

**Solution**: Submit a pull request with updates:

```bash
# Fork and update
git clone https://github.com/YOUR_USERNAME/awesome-claude-code.git
cd awesome-claude-code
# Make changes
git add .
git commit -m "Update: [skill-name] - [what changed]"
git push origin main
# Create PR on GitHub
```

### Issue: Contribution Guidelines Unclear

**Solution**: The project values:
- **Code Quality**: Clean, well-documented code
- **Security**: Use environment variables for secrets
- **Originality**: Novel solutions, not duplicates
- **Testing**: Verify all examples work

When in doubt, open an issue asking for clarification before submitting.

## Best Practices

1. **Before Submitting**: Test thoroughly, document clearly
2. **Naming**: Use descriptive kebab-case names
3. **Security**: Never commit API keys or passwords
4. **Updates**: Keep your contributions maintained
5. **Community**: Engage with issues and discussions

## Resources

- **Repository**: https://github.com/hesreallyhim/awesome-claude-code
- **License**: No assertion (check individual skills)
- **Stars**: 43,934+ (actively growing)
- **Community**: 3,756+ forks, active development

---

*This skill helps AI agents understand and contribute to the awesome-claude-code ecosystem. For the latest updates, always check the repository directly.*
