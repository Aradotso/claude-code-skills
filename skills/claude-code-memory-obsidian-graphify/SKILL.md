---
name: claude-code-memory-obsidian-graphify
description: Set up persistent memory and knowledge graphs for Claude Code using Obsidian Zettelkasten and Graphify to reduce tokens by up to 71.5x
triggers:
  - set up claude code memory with obsidian
  - configure persistent memory for coding agent
  - implement zettelkasten for claude code
  - reduce token usage with graphify
  - create knowledge graph for codebase
  - set up obsidian vault for claude
  - configure chat import pipeline
  - implement second brain for coding
---

# Claude Code Memory with Obsidian + Graphify

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

This skill enables you to set up a persistent memory system for Claude Code that reduces token consumption by up to 71.5x through Obsidian Zettelkasten notes and Graphify knowledge graphs. You'll maintain context across sessions, eliminate codebase re-reading, and preserve conversation history.

## What This System Does

**Solves Two Core Problems:**

1. **Session Amnesia**: Claude Code forgets everything between sessions — you constantly re-explain your stack, decisions, and progress
2. **Token Waste**: Re-reading ~40 files costs ~20k tokens per session just for orientation

**Three-Layer Solution:**

- **Obsidian Vault**: Centralized Zettelkasten with atomic notes, wikilinks, and YAML frontmatter for persistent project memory
- **Graphify**: AST-based codebase knowledge graphs that replace file re-reading with efficient queries
- **Chat Import Pipeline**: Python script + cron job to preserve Claude conversation insights as vault notes

## Prerequisites

```bash
# Required installations
pip install graphifyy
pip install claude-conversation-extractor

# Download Obsidian
# Visit: https://obsidian.md (free)
```

## Installation & Setup

### 1. Install Graphify

```bash
# Install Graphify
pip install graphifyy

# Install Claude Code skill
graphify install --platform claude

# Verify skill installation
ls ~/.claude/skills/graphify/SKILL.md
```

### 2. Configure API Keys

```bash
# Add to ~/.bashrc or ~/.zshrc
export ANTHROPIC_API_KEY="sk-ant-your-key-here"
# OR for Moonshot (Kimi)
export MOONSHOT_API_KEY="your-moonshot-key"

# Reload shell
source ~/.bashrc
```

### 3. Create Obsidian Vault Structure

```bash
# Create single vault for all projects
mkdir -p ~/vault/{permanent,inbox,fleeting,templates,logs,references}
mkdir -p ~/vault/chats/{code,web}
mkdir -p ~/vault/graphify

# Create project structure (repeat for each project)
PROJECT_NAME="my-project"
mkdir -p ~/vault/$PROJECT_NAME/{architecture,pipeline,data,features,logs}
```

### 4. Create CLAUDE.md Configuration

Create `~/vault/CLAUDE.md`:

```markdown
# Vault — Instructions for Claude Code

## What is this vault
Centralized knowledge base for all projects.
Persistent memory across sessions using Zettelkasten method.

## Zettelkasten Rules

### Note creation
- Use wikilinks: [[note-name]] (not markdown links)
- Mandatory YAML frontmatter on every note
- Filenames in kebab-case: `auth-flow.md`, not `Auth Flow.md`
- 1 concept per permanent note (atomicity)
- Minimum 2 wikilinks per note (dense linking)

### Standard frontmatter
---
title: Note Name
tags: [project, topic]
created: YYYY-MM-DD
updated: YYYY-MM-DD
status: active
type: permanent
---

### Never do
- Don't delete notes without asking
- Don't use markdown links for internal notes
- Don't create notes without frontmatter
- Don't change folder structure without documenting

## Session Commands

### /resume
1. Read 3 most recent session logs in logs/
2. Read architecture/decisions.md for current project
3. Summarize current state and pending work

### /save
1. Create session log in logs/YYYY-MM-DD-description.md
2. Record: completed work, decisions, pending items
3. Add wikilinks to created/modified notes
4. Run git commit + push if in repository
```

### 5. Set Up Chat Import Pipeline

Create `~/scripts/claude_to_obsidian.py`:

```python
#!/usr/bin/env python3
import os
import re
import argparse
from pathlib import Path
from datetime import datetime

# Keyword to tag mapping
KEYWORD_TAG_MAP = {
    "python": "python",
    "react": "react",
    "typescript": "typescript",
    "supabase": "supabase",
    "deploy": "deploy",
    "bug": "debugging",
    "refactor": "refactoring",
    "api": "api",
    "database": "database",
    "auth": "authentication",
    "error": "debugging",
    "test": "testing",
}

def extract_tags_from_content(content):
    """Extract tags based on keywords in content."""
    tags = set(["chat-import"])
    content_lower = content.lower()
    
    for keyword, tag in KEYWORD_TAG_MAP.items():
        if keyword in content_lower:
            tags.add(tag)
    
    return sorted(list(tags))

def find_wikilinks(content, vault_dir):
    """Find existing vault notes and insert wikilinks."""
    vault_path = Path(vault_dir)
    existing_notes = {}
    
    # Build index of existing notes
    for md_file in vault_path.rglob("*.md"):
        note_name = md_file.stem
        existing_notes[note_name.lower()] = note_name
    
    # Replace note references with wikilinks
    for note_lower, note_name in existing_notes.items():
        # Avoid replacing inside existing wikilinks
        pattern = rf'\b{re.escape(note_name)}\b(?![^\[]*\]\])'
        content = re.sub(
            pattern,
            f'[[{note_name}]]',
            content,
            flags=re.IGNORECASE
        )
    
    return content

def process_chat(file_path, vault_dir, origin):
    """Process a chat export file."""
    with open(file_path, 'r', encoding='utf-8') as f:
        content = f.read()
    
    # Extract title from first heading or filename
    title_match = re.search(r'^#\s+(.+)$', content, re.MULTILINE)
    title = title_match.group(1) if title_match else Path(file_path).stem
    
    # Extract date from filename or use current
    date_match = re.search(r'(\d{4}-\d{2}-\d{2})', Path(file_path).stem)
    date = date_match.group(1) if date_match else datetime.now().strftime('%Y-%m-%d')
    
    # Generate tags
    tags = extract_tags_from_content(content)
    
    # Add origin-specific tags
    if origin == "code":
        tags.append("claude-code")
    elif origin == "web":
        tags.append("claude-web")
    
    # Insert wikilinks
    content = find_wikilinks(content, vault_dir)
    
    # Create frontmatter
    frontmatter = f"""---
title: {title}
tags: [{', '.join(tags)}]
created: {date}
updated: {datetime.now().strftime('%Y-%m-%d')}
status: active
type: chat
origin: {origin}
---

"""
    
    # Combine frontmatter with content
    if content.startswith('---'):
        # Remove existing frontmatter
        content = re.sub(r'^---\n.*?\n---\n', '', content, flags=re.DOTALL)
    
    final_content = frontmatter + content.strip()
    
    return final_content, title, date

def main():
    parser = argparse.ArgumentParser(description='Process Claude chat exports for Obsidian')
    parser.add_argument('--export-dir', required=True, help='Claude export directory')
    parser.add_argument('--vault-dir', required=True, help='Obsidian vault directory')
    parser.add_argument('--move', action='store_true', help='Move files instead of copy')
    
    args = parser.parse_args()
    
    export_dir = Path(args.export_dir)
    vault_dir = Path(args.vault_dir)
    
    # Process both code and web exports
    for origin in ['code', 'web']:
        source_dir = export_dir / origin
        target_dir = vault_dir / 'chats' / origin
        
        if not source_dir.exists():
            continue
        
        target_dir.mkdir(parents=True, exist_ok=True)
        
        for chat_file in source_dir.glob('*.md'):
            try:
                processed_content, title, date = process_chat(
                    chat_file, vault_dir, origin
                )
                
                # Create safe filename
                safe_title = re.sub(r'[^\w\s-]', '', title)
                safe_title = re.sub(r'[-\s]+', '-', safe_title).strip('-')
                filename = f"{date}-{safe_title}.md"
                
                target_file = target_dir / filename
                
                # Write processed content
                with open(target_file, 'w', encoding='utf-8') as f:
                    f.write(processed_content)
                
                print(f"Processed: {filename}")
                
                # Remove source if moving
                if args.move:
                    chat_file.unlink()
                    
            except Exception as e:
                print(f"Error processing {chat_file}: {e}")

if __name__ == '__main__':
    main()
```

Make executable:

```bash
chmod +x ~/scripts/claude_to_obsidian.py
```

### 6. Create Automation Script

Create `~/scripts/sync_claude_obsidian.sh`:

```bash
#!/bin/bash
EXPORT_DIR="$HOME/claude-exports"
VAULT_DIR="$HOME/vault"
SCRIPT_DIR="$HOME/scripts"
LOG="$SCRIPT_DIR/sync.log"

echo "[$(date)] Sync started" >> "$LOG"

# Create export directories
mkdir -p "$EXPORT_DIR/code" "$EXPORT_DIR/web"

# Export Claude Code chats
claude-extract --all --output "$EXPORT_DIR/code" 2>> "$LOG"

# Process and import to vault
python3 "$SCRIPT_DIR/claude_to_obsidian.py" \
    --export-dir "$EXPORT_DIR" \
    --vault-dir "$VAULT_DIR" \
    --move 2>> "$LOG"

echo "[$(date)] Sync completed" >> "$LOG"
```

Make executable and schedule:

```bash
chmod +x ~/scripts/sync_claude_obsidian.sh

# Run daily at 10 PM
(crontab -l 2>/dev/null; echo "0 22 * * * $HOME/scripts/sync_claude_obsidian.sh") | crontab -
```

## Using Graphify

### Generate Knowledge Graph for a Project

**Inside Claude Code:**

```
/graphify . --obsidian --obsidian-dir ~/vault/graphify/my-project
```

**From Terminal (AST-only mode, 0 tokens):**

```bash
cd /path/to/project
graphify extract . --out ./graphify-out --no-cluster
```

**With Semantic Extraction (uses LLM):**

```bash
# Full semantic analysis
graphify extract . --out ./graphify-out --deep

# With custom model
graphify extract . --out ./graphify-out --deep --model claude-3-5-sonnet-20241022
```

### Graphify Commands Reference

```bash
# Basic extraction
graphify extract <path> --out <output-dir>

# Obsidian export
graphify extract <path> --obsidian --obsidian-dir ~/vault/graphify/project-name

# Skip LLM clustering
graphify extract <path> --out ./out --no-cluster

# Deep semantic analysis
graphify extract <path> --out ./out --deep

# Specify model
graphify extract <path> --model claude-3-5-sonnet-20241022

# Query the graph
graphify query <graph-dir> "find all auth functions"

# Supported languages (via tree-sitter)
# Python, JavaScript, TypeScript, Go, Rust, Java, C, C++, Ruby, C#, 
# Kotlin, Scala, PHP, Swift, Lua, Zig, and more
```

### Graph Output Structure

```
graphify-out/
├── graph.json              # Full knowledge graph
├── nodes.json             # Node index
├── edges.json             # Edge index
└── obsidian/              # Obsidian-formatted notes (if --obsidian)
    ├── components/
    ├── functions/
    └── relationships/
```

## Workflow Patterns

### Starting a New Session

```markdown
/resume
```

This command makes Claude Code:
1. Read the 3 most recent session logs
2. Read `architecture/decisions.md` for your project
3. Summarize current state and pending work

### Ending a Session

```markdown
/save
```

This command makes Claude Code:
1. Create a session log in `logs/YYYY-MM-DD-description.md`
2. Document completed work, decisions, and pending items
3. Add wikilinks to created/modified notes
4. Commit and push to git if in a repository

### Creating Project Notes

```markdown
Create a new note about our authentication flow in [[my-project/architecture/auth-flow]]. 

Include:
- OAuth provider integration
- Session management
- Token refresh strategy

Link to [[supabase-setup]] and [[api-design]].
```

### Querying the Knowledge Graph

```markdown
Query the Graphify graph for my-project:
- Find all functions that interact with the database
- Show dependencies for the auth module
- List unused utility functions
```

### Updating the Graph

```bash
# Re-run after significant code changes
cd /path/to/project
/graphify . --obsidian --obsidian-dir ~/vault/graphify/my-project
```

## Zettelkasten Best Practices

### Atomic Notes

**Good:**
```markdown
---
title: JWT Token Refresh Strategy
tags: [my-project, authentication, api]
created: 2024-01-15
updated: 2024-01-15
status: active
type: permanent
---

# JWT Token Refresh Strategy

## Context
We use sliding window refresh to minimize user disruption.

## Implementation
- Access token: 15 min expiry
- Refresh token: 7 day expiry
- Auto-refresh at 80% of access token lifetime

## Related
- [[authentication-flow]]
- [[supabase-auth-config]]
```

**Bad (not atomic):**
```markdown
# Everything About Authentication

Contains JWT, OAuth, session management, password reset...
(Too many concepts in one note)
```

### Dense Linking

Every permanent note should have **at least 2 wikilinks** to other notes. This creates the knowledge graph structure.

```markdown
## Related
- [[parent-concept]]
- [[sibling-concept]]
- [[implementation-detail]]
```

### Frontmatter Standards

```yaml
---
title: Human Readable Title
tags: [project-name, topic, subtopic]
created: YYYY-MM-DD
updated: YYYY-MM-DD
status: active|draft|archived
type: permanent|fleeting|reference|chat
---
```

## Troubleshooting

### Graphify Skill Not Found

```bash
# Reinstall skill
graphify install --platform claude

# Verify installation
cat ~/.claude/skills/graphify/SKILL.md
```

### API Key Issues

```bash
# Check environment variables
echo $ANTHROPIC_API_KEY
echo $MOONSHOT_API_KEY

# Test API key
curl https://api.anthropic.com/v1/messages \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{
    "model": "claude-3-5-sonnet-20241022",
    "max_tokens": 1024,
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```

### Chat Import Not Working

```bash
# Test extractor
claude-extract --help

# Check permissions
chmod +x ~/scripts/claude_to_obsidian.py
chmod +x ~/scripts/sync_claude_obsidian.sh

# Manual run
~/scripts/sync_claude_obsidian.sh

# Check logs
tail -f ~/scripts/sync.log
```

### Obsidian Wikilinks Not Resolving

1. Ensure filenames are in kebab-case
2. Check that wikilinks match exact filename without `.md`
3. Use relative paths for cross-folder links: `[[folder/note-name]]`
4. Verify YAML frontmatter is valid

### Graph Query Returns Empty Results

```bash
# Verify graph generation
ls ~/vault/graphify/my-project/

# Check graph.json exists
cat ~/vault/graphify/my-project/graph.json | head

# Re-extract with verbose output
graphify extract . --out ./out --verbose
```

### Token Count Still High

**Checklist:**
- ✅ Generated Graphify graph for project
- ✅ CLAUDE.md exists in vault root
- ✅ Using `/resume` and `/save` commands
- ✅ Session logs contain wikilinks to relevant notes
- ✅ Graph is up-to-date (re-run after major code changes)

### Cron Job Not Running

```bash
# Check cron is active
sudo systemctl status cron

# Verify crontab entry
crontab -l

# Test manual execution
~/scripts/sync_claude_obsidian.sh

# Check cron logs
grep CRON /var/log/syslog
```

## Configuration Examples

### Multi-Project Setup

```
~/vault/
├── CLAUDE.md
├── project-a/
│   ├── architecture/
│   ├── features/
│   └── logs/
├── project-b/
│   ├── architecture/
│   ├── features/
│   └── logs/
└── graphify/
    ├── project-a/
    └── project-b/
```

### Custom Tag Mapping

Edit `~/scripts/claude_to_obsidian.py`:

```python
KEYWORD_TAG_MAP = {
    # Your stack
    "nextjs": "nextjs",
    "tailwind": "tailwind",
    "prisma": "prisma",
    "vercel": "deploy",
    
    # Your patterns
    "hook": "react-hooks",
    "component": "components",
    "util": "utilities",
    
    # Your workflow
    "review": "code-review",
    "optimize": "performance",
}
```

### Graphify Model Selection

```bash
# Use cheaper model for large codebases
graphify extract . --model claude-3-haiku-20240307

# Use most capable model for complex projects
graphify extract . --model claude-3-opus-20240229

# Use latest Sonnet (recommended)
graphify extract . --model claude-3-5-sonnet-20241022
```

## Expected Results

- **Initial session**: ~20k tokens for codebase orientation
- **With Graphify + Obsidian**: ~280 tokens per session
- **Reduction**: 71.5x fewer tokens
- **Memory persistence**: 100% context retention across sessions
- **Chat preservation**: Zero conversation insights lost

## Resources

- [Graphify GitHub](https://github.com/safishamsi/graphify)
- [Obsidian Documentation](https://help.obsidian.md/)
- [Zettelkasten Method](https://zettelkasten.de/posts/overview/)
- [Claude Code Skills](https://ara.so)
