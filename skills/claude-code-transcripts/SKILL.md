---
name: claude-code-transcripts
description: Convert Claude Code session files (JSON/JSONL) to clean, paginated HTML transcripts with GitHub integration
triggers:
  - convert claude code session to html
  - export claude code transcript
  - publish claude code session to gist
  - create html from claude code session
  - view claude code session history
  - extract claude code transcript
  - convert all claude sessions to html archive
  - share claude code session as webpage
---

# claude-code-transcripts

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## Overview

`claude-code-transcripts` converts Claude Code session files (JSON or JSONL) into clean, mobile-friendly, paginated HTML transcripts. It supports local sessions from `~/.claude/projects`, web sessions via the Claude API, and can publish directly to GitHub Gists for easy sharing.

## Installation

Install using `uv` (recommended):

```bash
uv tool install claude-code-transcripts
```

Or run without installing:

```bash
uvx claude-code-transcripts --help
```

Install with pip:

```bash
pip install claude-code-transcripts
```

## Key Commands

### 1. Local Sessions (Default)

Select from local Claude Code sessions stored in `~/.claude/projects`:

```bash
# Interactive picker for recent sessions
claude-code-transcripts

# Explicitly use local command
claude-code-transcripts local

# Show more sessions
claude-code-transcripts local --limit 20

# Save to specific directory
claude-code-transcripts local -o ./my-transcript

# Auto-name output directory based on session
claude-code-transcripts local -a

# Include original JSON file
claude-code-transcripts local -o ./output --json
```

### 2. Web Sessions

Import sessions from Claude API (macOS automatically retrieves credentials from keychain):

```bash
# Interactive session picker
claude-code-transcripts web

# Import specific session by ID
claude-code-transcripts web SESSION_ABC123

# Filter by GitHub repo
claude-code-transcripts web --repo simonw/datasette

# Manually provide credentials (non-macOS)
claude-code-transcripts web --token $CLAUDE_TOKEN --org-uuid $ORG_UUID
```

### 3. Convert Specific JSON/JSONL Files

Convert a session file directly:

```bash
# From local file
claude-code-transcripts json session.json -o ./output

# From URL
claude-code-transcripts json https://example.com/session.json --open

# JSONL format (from ~/.claude/projects)
claude-code-transcripts json ~/.claude/projects/my-project/session.jsonl -a
```

### 4. Convert All Sessions to Archive

Create a browsable HTML archive of all local sessions:

```bash
# Default: converts all sessions to ./claude-archive
claude-code-transcripts all

# Preview without creating files
claude-code-transcripts all --dry-run

# Custom output directory
claude-code-transcripts all -o ./my-archive

# Include agent sessions (excluded by default)
claude-code-transcripts all --include-agents

# Open in browser when done
claude-code-transcripts all --open

# Quiet mode (errors only)
claude-code-transcripts all -q
```

## Publishing to GitHub Gist

Upload transcripts to GitHub Gist and get a shareable preview URL:

```bash
# Requires: gh CLI installed and authenticated (gh auth login)

# From local session
claude-code-transcripts --gist

# From web session
claude-code-transcripts web SESSION_ID --gist

# From JSON file
claude-code-transcripts json session.json --gist

# Keep local copy too
claude-code-transcripts json session.json -o ./backup --gist
```

Output example:
```
Gist: https://gist.github.com/username/abc123def456
Preview: https://gisthost.github.io/?abc123def456/index.html
Files: /tmp/session-abc123
```

The preview URL uses gisthost.github.io to render the HTML gist with working relative links.

## Common Options

All commands support these options:

- `-o, --output DIRECTORY` - output directory (default: temp dir + auto-open browser)
- `-a, --output-auto` - auto-name subdirectory based on session ID/filename
- `--repo OWNER/NAME` - GitHub repo for commit links (auto-detected from git)
- `--open` - open `index.html` in default browser (default when no `-o`)
- `--gist` - upload to GitHub Gist and output preview URL
- `--json` - include original session file in output directory

## Output Structure

Generated files include:

```
output-directory/
├── index.html          # Timeline of prompts and commits
├── page-001.html       # First page of transcript
├── page-002.html       # Second page of transcript
├── ...
└── session_ABC.json    # Original session (if --json flag used)
```

## Python API Usage

Use as a Python library:

```python
from claude_code_transcripts.core import convert_session_to_html
from pathlib import Path

# Convert a session file
session_file = Path("~/.claude/projects/my-project/session.jsonl").expanduser()
output_dir = Path("./transcript")

convert_session_to_html(
    session_file=session_file,
    output_dir=output_dir,
    repo="owner/repo",  # Optional: for GitHub commit links
    open_browser=True
)
```

Fetch web sessions programmatically:

```python
from claude_code_transcripts.web import fetch_session_data, list_sessions

# List available sessions
sessions = list_sessions(token="...", org_uuid="...")
for session in sessions:
    print(f"{session['id']}: {session.get('description', 'No description')}")

# Fetch specific session
session_data = fetch_session_data(
    session_id="SESSION_ABC123",
    token="...",
    org_uuid="..."
)
```

## Configuration

### GitHub Repository Detection

The tool auto-detects the GitHub repository from your local git config. Override with `--repo`:

```bash
claude-code-transcripts --repo myorg/myrepo
```

This enables clickable commit links in the generated HTML.

### Pagination

Transcripts are automatically paginated for readability. Each page includes:
- Navigation between pages
- Back to index link
- Timestamp for each message
- Syntax-highlighted code blocks
- Tool usage details

## Common Patterns

### Quick Preview Workflow

```bash
# Select recent session, generate HTML, and open in browser
claude-code-transcripts
```

### Share Session Publicly

```bash
# Convert web session to Gist with one command
claude-code-transcripts web SESSION_ID --gist
```

### Archive All Sessions

```bash
# Create complete browsable archive
claude-code-transcripts all -o ~/claude-archive --open
```

### Backup with Source Data

```bash
# Keep both HTML and original JSON
claude-code-transcripts json session.json -o ./backup --json
```

### Batch Processing

```bash
# Convert multiple sessions
for file in ~/.claude/projects/*/session.jsonl; do
    claude-code-transcripts json "$file" -a -o ./all-transcripts
done
```

## Troubleshooting

### Web Sessions Not Working

**Issue**: `web` command fails to list/fetch sessions (currently broken - see issue #77)

**Solution**: Use local sessions or JSON exports instead:
```bash
# Use local sessions
claude-code-transcripts local

# Or export JSON from web UI and convert
claude-code-transcripts json exported-session.json
```

### Gist Upload Fails

**Issue**: `--gist` option fails with authentication error

**Solution**: Install and authenticate GitHub CLI:
```bash
# Install gh CLI (macOS)
brew install gh

# Authenticate
gh auth login
```

### Missing Commit Links

**Issue**: Commit hashes not linking to GitHub

**Solution**: Specify repository explicitly:
```bash
claude-code-transcripts --repo owner/repo
```

Or ensure you're in a git repository with GitHub remote:
```bash
git remote -v  # Should show github.com URL
```

### No Sessions Found

**Issue**: `claude-code-transcripts local` shows no sessions

**Solution**: Check Claude Code sessions directory:
```bash
ls -la ~/.claude/projects/*/session.jsonl
```

If empty, you may not have local sessions or they're stored elsewhere.

### Large Session Files

**Issue**: Conversion takes a long time for very large sessions

**Solution**: Sessions are automatically paginated. For extremely large sessions, consider:
```bash
# Use quiet mode to suppress progress output
claude-code-transcripts json large-session.json -q
```

## Environment Variables

While the tool doesn't require environment variables for most use cases, web API access may use:

- `CLAUDE_TOKEN` - Claude API token (alternative to `--token`)
- `ORG_UUID` - Organization UUID (alternative to `--org-uuid`)

Note: On macOS, these are automatically retrieved from the system keychain when you're logged into Claude Code.

## Examples

### Example 1: Quick Local Session Export

```bash
# Interactive picker + auto-open
claude-code-transcripts
```

### Example 2: Share Web Session

```bash
# Get shareable link
claude-code-transcripts web abc123def456 --gist

# Output:
# Gist: https://gist.github.com/user/xyz789
# Preview: https://gisthost.github.io/?xyz789/index.html
```

### Example 3: Archive with Metadata

```bash
# Full archive with source files
claude-code-transcripts all \
  -o ~/Documents/claude-sessions \
  --include-agents \
  --json \
  --open
```

### Example 4: Process URL

```bash
# Convert session from URL
claude-code-transcripts json \
  https://raw.githubusercontent.com/user/repo/main/session.json \
  -o ./transcript \
  --repo user/repo \
  --gist
```
