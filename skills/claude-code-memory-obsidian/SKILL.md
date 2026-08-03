---
name: claude-code-memory-obsidian
description: Set up persistent memory, knowledge graphs, and token optimization for Claude Code using Obsidian + Graphify
triggers:
  - set up persistent memory for Claude Code
  - create an Obsidian vault for my coding sessions
  - reduce token usage with knowledge graphs
  - configure Claude Code memory with Obsidian
  - import my Claude chats into Obsidian
  - set up Graphify for codebase mapping
  - create a second brain for AI coding
  - optimize my Claude Code workflow
---

# Claude Code Memory + Obsidian + Graphify

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

This skill helps you set up a persistent memory system for Claude Code that reduces token consumption by up to 71.5x per session. It combines Obsidian (Zettelkasten knowledge base), Graphify (codebase knowledge graphs), and a chat import pipeline to eliminate repetitive context loading.

## What This Project Does

**claude-code-memory-setup** solves two critical problems:

1. **Session amnesia** - Claude Code forgets everything between sessions, forcing you to re-explain your project every time
2. **Codebase re-reading** - Claude Code re-reads all files each session to understand structure, burning 20k+ tokens before answering questions

The solution uses three layers:

- **Obsidian vault** - Centralized knowledge base storing decisions, architecture, and context across all projects
- **Graphify** - AST-based codebase knowledge graphs that persist between sessions (0 tokens in default mode)
- **Chat import pipeline** - Automatically extracts and indexes Claude conversations as searchable notes

## Installation

### Prerequisites

```bash
# Install Obsidian (free desktop app)
# Download from: https://obsidian.md

# Install Python dependencies
pip install graphifyy claude-conversation-extractor

# Install Graphify skill for Claude Code
graphify install --platform claude
```

### Vault Setup

```bash
# Create single vault directory (for ALL projects)
mkdir -p ~/vault/{permanent,inbox,fleeting,templates,logs,references}
mkdir -p ~/vault/{chats/code,chats/web,graphify}

# Create project-specific folders
mkdir -p ~/vault/my-project/{architecture,pipeline,data,features,logs}
```

### Configure CLAUDE.md

Create `~/vault/CLAUDE.md` as the master instructions file:

```markdown
# Vault — Instructions for Claude Code

## What is this vault
Centralized knowledge base for all projects.
Persistent memory across sessions.

## Zettelkasten Rules

### Note creation
- Use wikilinks: [[note-name]] (not markdown links)
- Mandatory YAML frontmatter on every note
- Filenames in kebab-case: `auth-flow.md`
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

## Session Commands

### /resume
When you receive this command:
1. Read the 3 most recent session logs in logs/
2. Read architecture/decisions.md for the current project
3. Summarize current state and what's left to do

### /save
When you receive this command:
1. Create a session log in logs/YYYY-MM-DD-description.md
2. Record: what was done, decisions made, pending items
3. Add wikilinks to created/modified notes
4. Run git commit + push if in a repository
```

### Note Template

Create `~/vault/templates/default-note.md`:

```markdown
---
title: {{title}}
tags: []
created: {{date}}
updated: {{date}}
status: draft
type: permanent
---

# {{title}}

## Context

## Details

## Related links
```

## Chat Import Pipeline Setup

### Create Processing Script

Create `~/scripts/claude_to_obsidian.py`:

```python
#!/usr/bin/env python3
import os
import re
import argparse
from pathlib import Path
from datetime import datetime

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
}

def extract_tags(content):
    """Extract tags from content based on keywords."""
    tags = set(["chat-import"])
    content_lower = content.lower()
    
    for keyword, tag in KEYWORD_TAG_MAP.items():
        if keyword in content_lower:
            tags.add(tag)
    
    return sorted(list(tags))

def find_wikilinks(content, vault_dir):
    """Find existing vault notes to link."""
    wikilinks = []
    vault_path = Path(vault_dir)
    
    # Get all existing note titles
    existing_notes = {}
    for md_file in vault_path.rglob("*.md"):
        if md_file.stem not in ["CLAUDE", "README"]:
            existing_notes[md_file.stem.lower()] = md_file.stem
    
    # Search for note titles in content
    for title_lower, title_original in existing_notes.items():
        if title_lower in content.lower():
            wikilinks.append(f"[[{title_original}]]")
    
    return wikilinks

def add_frontmatter(content, filepath, vault_dir, chat_type="code"):
    """Add YAML frontmatter to chat export."""
    filename = Path(filepath).stem
    tags = extract_tags(content)
    wikilinks = find_wikilinks(content, vault_dir)
    
    frontmatter = f"""---
title: {filename}
tags: {tags}
created: {datetime.now().strftime('%Y-%m-%d')}
updated: {datetime.now().strftime('%Y-%m-%d')}
status: imported
type: chat
source: claude-{chat_type}
---

"""
    
    if wikilinks:
        frontmatter += f"## Related notes\n{', '.join(wikilinks)}\n\n"
    
    return frontmatter + content

def process_exports(export_dir, vault_dir, move=False):
    """Process all exported chats and copy to vault."""
    export_path = Path(export_dir)
    vault_path = Path(vault_dir)
    
    for chat_type in ["code", "web"]:
        source_dir = export_path / chat_type
        target_dir = vault_path / "chats" / chat_type
        target_dir.mkdir(parents=True, exist_ok=True)
        
        if not source_dir.exists():
            continue
        
        for md_file in source_dir.glob("*.md"):
            content = md_file.read_text(encoding="utf-8")
            processed = add_frontmatter(content, md_file, vault_dir, chat_type)
            
            target_file = target_dir / md_file.name
            target_file.write_text(processed, encoding="utf-8")
            
            if move:
                md_file.unlink()
            
            print(f"Processed: {md_file.name} -> chats/{chat_type}/")

if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--export-dir", required=True)
    parser.add_argument("--vault-dir", required=True)
    parser.add_argument("--move", action="store_true")
    args = parser.parse_args()
    
    process_exports(args.export_dir, args.vault_dir, args.move)
```

### Create Automation Script

Create `~/scripts/sync_claude_obsidian.sh`:

```bash
#!/bin/bash
EXPORT_DIR="$HOME/claude-exports"
VAULT_DIR="$HOME/vault"
SCRIPT_DIR="$HOME/scripts"
LOG="$SCRIPT_DIR/sync.log"

echo "[$(date)] Sync started" >> "$LOG"

# Create export directories if needed
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

```bash
chmod +x ~/scripts/sync_claude_obsidian.sh
chmod +x ~/scripts/claude_to_obsidian.py

# Schedule daily at 10 PM
(crontab -l 2>/dev/null; echo "0 22 * * * $HOME/scripts/sync_claude_obsidian.sh") | crontab -
```

## Graphify Usage

### Set API Key (for semantic extraction)

```bash
# Use one of these providers
export ANTHROPIC_API_KEY="your-anthropic-key"
# or
export MOONSHOT_API_KEY="your-moonshot-key"
```

### Generate Knowledge Graph

**Inside Claude Code (recommended):**

```
/graphify . --obsidian --obsidian-dir ~/vault/graphify/my-project
```

**From terminal (AST-only, 0 tokens):**

```bash
cd /path/to/project
graphify extract . --out ~/vault/graphify/my-project --no-cluster
```

**With semantic extraction (uses LLM):**

```bash
graphify extract . --out ~/vault/graphify/my-project --deep
```

### Query the Graph

```python
from graphify import GraphQuery

# Load project graph
graph = GraphQuery("~/vault/graphify/my-project")

# Find authentication logic
auth_nodes = graph.find_by_keyword("auth")

# Get component dependencies
deps = graph.get_dependencies("UserService")

# Find all API routes
routes = graph.find_by_type("route")
```

## Complete Workflow

### Starting a New Session

```markdown
/resume

# Claude will:
# 1. Read recent session logs from logs/
# 2. Check architecture/decisions.md
# 3. Summarize current state
```

### During Development

Create atomic notes as you go:

```markdown
Please create a note about the authentication flow we just implemented.

# Claude creates: ~/vault/my-project/architecture/auth-flow.md
---
title: Authentication Flow
tags: [my-project, authentication, architecture]
created: 2026-08-03
updated: 2026-08-03
status: active
type: permanent
---

# Authentication Flow

## Context
Implementing Supabase Auth with React frontend.

## Flow
1. User enters credentials in [[login-form]]
2. Supabase validates and returns JWT
3. Token stored in [[auth-context]]
4. Protected routes check [[use-auth]] hook

## Related links
[[supabase-setup]], [[protected-routes]], [[user-session]]
```

### Using Graphify for Context

```markdown
/graphify . --obsidian --obsidian-dir ~/vault/graphify/my-project

# Then query instead of re-reading:
Show me all components that import the UserContext
```

### Ending a Session

```markdown
/save

# Claude will:
# 1. Create session log in logs/2026-08-03-auth-implementation.md
# 2. Document decisions and changes
# 3. Link to created/modified notes
# 4. Git commit if in repo
```

## Session Log Template

When using `/save`, Claude creates logs like:

```markdown
---
title: Session 2026-08-03 - Auth Implementation
tags: [session-log, my-project, authentication]
created: 2026-08-03
updated: 2026-08-03
status: completed
type: session-log
---

# Session: Auth Implementation

## What was done
- Implemented Supabase authentication flow
- Created [[auth-context]] and [[use-auth]] hook
- Set up protected routes with [[route-guard]]

## Decisions made
- Using Supabase Auth instead of custom JWT
- Storing tokens in localStorage (discussed security tradeoffs)
- Email/password only for MVP, social auth later

## Pending items
- [ ] Add password reset flow
- [ ] Implement session refresh
- [ ] Add role-based access control

## Files modified
- src/contexts/AuthContext.tsx (new)
- src/hooks/useAuth.ts (new)
- src/components/ProtectedRoute.tsx (modified)

## Related notes
[[supabase-setup]], [[auth-flow]], [[protected-routes]]
```

## Configuration Options

### Vault Structure Customization

```bash
# Minimal structure (single project)
~/vault/
├── CLAUDE.md
├── notes/
├── logs/
└── graphify/

# Multi-project structure (recommended)
~/vault/
├── CLAUDE.md
├── permanent/          # shared knowledge
├── project-a/
│   ├── architecture/
│   ├── logs/
│   └── features/
├── project-b/
│   └── ...
├── chats/
└── graphify/
    ├── project-a/
    └── project-b/
```

### Obsidian Plugins (Recommended)

| Plugin | Purpose | Installation |
|--------|---------|--------------|
| **BRAT** | Install beta plugins | Community Plugins → Search "BRAT" |
| **3D Graph** | Visualize vault in 3D | Via BRAT (search "3D Graph") |
| **Folders to Graph** | Show folders as nodes | Community Plugins → Search |
| **Calendar** | Navigate daily notes | Community Plugins → Search |

### Graphify Options

```bash
# AST-only (fastest, 0 tokens)
graphify extract . --out ./graph --no-cluster

# With semantic clustering (uses LLM)
graphify extract . --out ./graph --deep

# Export to Obsidian format
graphify extract . --out ./graph --obsidian --obsidian-dir ~/vault/graphify/project

# Specific languages only
graphify extract . --out ./graph --language python --language javascript

# Exclude directories
graphify extract . --out ./graph --exclude node_modules --exclude .venv
```

## Common Patterns

### Pattern 1: Feature Development

```markdown
# Start session
/resume

# Generate graph if codebase changed
/graphify . --obsidian --obsidian-dir ~/vault/graphify/my-project

# Work on feature...

# Document decision
Please create a note about why we chose React Query over Redux

# End session
/save
```

### Pattern 2: Debugging

```markdown
# Resume context
/resume

# Query graph for bug location
/graphify . --obsidian --obsidian-dir ~/vault/graphify/my-project
Show me all files that handle user authentication

# Fix bug...

# Document root cause
Create a note about the null pointer bug in UserService
```

### Pattern 3: Code Review

```markdown
# Load project context
/resume

# Use graph to understand changes
/graphify . --obsidian --obsidian-dir ~/vault/graphify/my-project
What components depend on the AuthContext we're modifying?

# Review and document
Create a note about the security implications of this PR
```

## Troubleshooting

### Chat Import Not Working

```bash
# Verify extractor is installed
pip show claude-conversation-extractor

# Test manual export
claude-extract --all --output ~/test-export

# Check permissions
chmod +x ~/scripts/sync_claude_obsidian.sh
chmod +x ~/scripts/claude_to_obsidian.py

# Run manually to see errors
~/scripts/sync_claude_obsidian.sh
```

### Graphify Errors

```bash
# API key not set
export ANTHROPIC_API_KEY="your-key-here"

# Skill not installed
graphify install --platform claude

# Verify skill exists
ls ~/.claude/skills/graphify/SKILL.md

# Test graph generation
cd /path/to/project
graphify extract . --out ./test-graph --no-cluster
```

### Wikilinks Not Creating

```markdown
# Ensure consistent naming
- Use kebab-case: auth-flow.md (not Auth Flow.md)
- No spaces in filenames
- Frontmatter must have 'title' field

# Fix existing notes
Please rename all notes in my-project/ to kebab-case
```

### `/resume` or `/save` Not Working

Check that `CLAUDE.md` exists and contains the command definitions. Re-open Claude Code to reload the context.

```bash
# Verify CLAUDE.md location
ls ~/vault/CLAUDE.md

# Check it's being loaded
# Claude Code should show vault path in context
```

### Session Logs Not Linking

Ensure note titles match wikilink syntax exactly:

```markdown
# Won't link
[[Auth Flow]]  # (file is auth-flow.md)

# Will link
[[auth-flow]]  # (matches filename)
```

## Real Results

From the original project testing:

- **Before**: ~20,000 tokens per session (40-file codebase re-reading)
- **After**: ~280 tokens per session (graph queries only)
- **Reduction**: 71.5x fewer tokens
- **Sessions**: Persistent context across 10+ sessions without re-explaining

### Token Breakdown

| Operation | Before | After | Savings |
|-----------|--------|-------|---------|
| Codebase orientation | 20,000 | 0 | 100% |
| Context re-explanation | 5,000 | 50 | 99% |
| File re-reading | 15,000 | 230 | 98.5% |
| **Total per session** | **40,000** | **280** | **98.3%** |

## Resources

- **Project repo**: https://github.com/lucasrosati/claude-code-memory-setup
- **Graphify repo**: https://github.com/safishamsi/graphify
- **Obsidian docs**: https://help.obsidian.md
- **Zettelkasten guide**: https://zettelkasten.de/introduction/
