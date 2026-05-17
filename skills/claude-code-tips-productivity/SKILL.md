---
name: claude-code-tips-productivity
description: Master Claude Code with 45 expert tips covering voice input, context management, workflows, custom status lines, container usage, and advanced automation techniques
triggers:
  - how do I use Claude Code more effectively
  - what are best practices for Claude Code
  - show me Claude Code tips and tricks
  - how to optimize my Claude Code workflow
  - help me set up voice input for Claude
  - how do I manage context in Claude Code
  - teach me advanced Claude Code techniques
  - what are essential Claude Code slash commands
---

# Claude Code Tips & Productivity

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

This skill provides expertise in using Claude Code effectively, based on the comprehensive tips collection from ykdojo/claude-code-tips. It covers everything from basic slash commands to advanced workflows including custom status lines, voice input, context management, container usage, and multi-agent automation.

## Core Concepts

### Essential Slash Commands

The most important built-in commands:

- `/usage` - Check rate limits and token usage
- `/chrome` - Toggle native browser integration
- `/mcp` - Manage Model Context Protocol servers
- `/stats` - View usage statistics with activity graph
- `/clear` - Clear conversation and start fresh

To see all available commands, type `/` in the chat.

### Context Management Philosophy

**"AI context is like milk; it's best served fresh and condensed!"**

Key principles:
- Proactively compact context before it becomes bloated
- Break down large problems into smaller conversations
- Use Git and worktrees to isolate work
- Clone/fork conversations for parallel experimentation

## Custom Status Line Setup

Create a custom status bar showing model, directory, git branch, uncommitted files, and token usage.

**Installation:**

```bash
# Download the status line script
curl -o ~/.claude-status.sh https://raw.githubusercontent.com/ykdojo/claude-code-tips/main/scripts/context-bar.sh
chmod +x ~/.claude-status.sh

# Add to your CLAUDE.md
echo 'status_line_command: "~/.claude-status.sh"' >> CLAUDE.md
```

**Configuration (in script):**

```bash
# Choose color theme (orange, blue, teal, green, lavender, rose, gold, slate, cyan, gray)
COLOR_THEME="orange"

# Customize what's shown
SHOW_MODEL=true
SHOW_DIRECTORY=true
SHOW_GIT_BRANCH=true
SHOW_TOKEN_USAGE=true
```

Example output:
```
Opus 4.5 | 📁project-name | 🔀main (2 uncommitted, synced 12m ago) | ██░░░░░░░░ 18% of 200k tokens
💬 Last message preview...
```

## Voice Input Integration

Voice input dramatically increases communication speed with Claude Code.

### Local Voice Transcription Options

**macOS:**
- [superwhisper](https://superwhisper.com/) - Fast, accurate
- [MacWhisper](https://goodsnooze.gumroad.com/l/macwhisper) - Privacy-focused
- [Super Voice Assistant](https://github.com/ykdojo/super-voice-assistant) - Open source, Parakeet v2/v3

**Setup Example (Super Voice Assistant):**

```bash
# Clone and install
git clone https://github.com/ykdojo/super-voice-assistant
cd super-voice-assistant
pip install -r requirements.txt

# Configure hotkey in config.json
{
  "model": "parakeet-v3",
  "hotkey": "cmd+shift+space",
  "output_method": "clipboard"
}

# Run
python main.py
```

**Best Practices:**
- Claude understands mistranscribed words from context
- Use earphones/EarPods for quiet whispering in offices
- Works even in noisy environments (planes, cafes)
- Built-in voice mode also available in recent Claude Code versions

## Context Optimization Techniques

### Proactive Context Compaction

```javascript
// Instead of keeping full conversation history
// Ask Claude to summarize periodically:
// "Please summarize our progress so far and what remains"

// Use /clear strategically and start new conversations with:
// "We previously built X. Now let's work on Y."
```

### Breaking Down Large Problems

```
User: "Build a full-stack app with auth, payments, and notifications"
Better approach:
  1. "Set up project structure and auth system"
  2. "Integrate payment processing" (new conversation)
  3. "Add notification system" (new conversation)
```

### Git Worktrees for Parallel Work

```bash
# Create worktrees for parallel development
git worktree add ../feature-branch feature-branch
git worktree add ../bugfix bugfix

# Now run separate Claude Code sessions in each:
# Terminal 1: cd ../feature-branch && claude
# Terminal 2: cd ../bugfix && claude
```

## Terminal & Git Workflow

### Recommended Aliases

```bash
# Add to ~/.bashrc or ~/.zshrc
alias c='claude'
alias cwd='pwd | pbcopy'  # Copy current directory
alias gst='git status'
alias glog='git log --oneline --graph --all'

# Quick Claude with context
alias cc='claude --prompt "Continue where we left off"'
```

### Git Integration Best Practices

```bash
# Always work in branches
git checkout -b feature/new-task

# Let Claude handle commits
# Instead of manual git commands, ask:
# "Review changes and create a commit with good message"

# Use GitHub CLI for PR workflows
gh pr create --title "Feature: ..." --body "..."
gh pr review --approve
```

### Getting Terminal Output

```bash
# Pipe command output directly to Claude
npm test 2>&1 | pbcopy
# Then: "Here's the test output: [paste]"

# Or save to file for reference
npm run build > build-output.txt 2>&1
# Then: "@build-output.txt what's wrong?"
```

## Container Usage for Risky Tasks

Run Claude Code itself in a container for isolated, long-running, or risky operations.

```dockerfile
# Dockerfile
FROM node:18-slim

RUN apt-get update && apt-get install -y git curl

# Install Claude Code
RUN npm install -g @anthropic-ai/claude-code

# Set up workspace
WORKDIR /workspace

CMD ["claude"]
```

```bash
# Build and run
docker build -t claude-sandbox .

# Run with volume mount
docker run -it \
  -v $(pwd):/workspace \
  -v ~/.gitconfig:/root/.gitconfig:ro \
  -e ANTHROPIC_API_KEY \
  claude-sandbox

# For long-running tasks
docker run -d --name claude-worker \
  -v $(pwd):/workspace \
  claude-sandbox

# Check progress
docker logs -f claude-worker
```

## Using Gemini CLI as Fallback

When Claude Code can't access certain sites or APIs, use Gemini CLI as a "minion."

```bash
# Install Gemini CLI
npm install -g @google/generative-ai-cli

# Set up API key
export GOOGLE_API_KEY=your_key_here

# Use in Claude Code workflow
gemini "Search for latest React documentation on hooks" > gemini-output.txt

# Then feed to Claude
# "Here's what Gemini found: @gemini-output.txt"
```

## Advanced Workflow Patterns

### Multi-Agent Workflows

```bash
# Terminal 1: Main Claude (architecture & planning)
claude --session main

# Terminal 2: Test Claude (writing tests)
claude --session tests

# Terminal 3: Docs Claude (documentation)
claude --session docs

# Coordinate between them:
# Main: "Create API spec and save to api-spec.md"
# Tests: "@api-spec.md Write comprehensive tests"
# Docs: "@api-spec.md Generate API documentation"
```

### Clone/Fork Conversations

```
# In active conversation, ask:
"Clone this conversation so I can experiment with approach B"

# Or use conversation history:
# 1. Open conversation list
# 2. Right-click conversation
# 3. Select "Fork from here"
```

### Background Tasks

```bash
# Run long tasks in background
nohup npm run build > build.log 2>&1 &
BUILD_PID=$!

# Check progress periodically
tail -f build.log

# Ask Claude to monitor
# "Check build.log every 5 minutes and report status"
```

## CLAUDE.md vs Skills vs Plugins

**CLAUDE.md (Project-level instructions):**
```markdown
# Project: MyApp

## Code Style
- Use TypeScript strict mode
- Prefer functional components
- Max line length: 100

## Testing
- Write tests for all new features
- Run `npm test` before committing

## Commands
- `npm run dev` - Start dev server
- `npm run build` - Production build
```

**Skills (Global expertise):**
- Installed via MCP or direct import
- Available across all projects
- This file is a Skill!

**Plugins (Tool integrations):**
```bash
# Install the dx plugin for enhanced developer experience
claude plugin install dx

# Configure in ~/.claude.json
{
  "plugins": {
    "dx": {
      "enabled": true,
      "shortcuts": {
        "test": "npm test",
        "lint": "npm run lint"
      }
    }
  }
}
```

## Real-World Code Examples

### Interactive PR Review

```javascript
// review-pr.js - Script to prepare PR for Claude review
const { execSync } = require('child_process');
const fs = require('fs');

function preparePRReview(prNumber) {
  // Fetch PR details
  const prData = execSync(`gh pr view ${prNumber} --json title,body,files`);
  const pr = JSON.parse(prData);
  
  // Get diff
  const diff = execSync(`gh pr diff ${prNumber}`).toString();
  
  // Create review context
  const context = {
    title: pr.title,
    description: pr.body,
    files: pr.files.map(f => f.path),
    diff: diff
  };
  
  fs.writeFileSync('pr-context.json', JSON.stringify(context, null, 2));
  console.log('PR context saved to pr-context.json');
  console.log('\nTell Claude: "Review @pr-context.json for code quality, bugs, and improvements"');
}

// Usage: node review-pr.js 123
preparePRReview(process.argv[2]);
```

### Audit Approved Commands

```bash
#!/bin/bash
# audit-commands.sh - Review commands Claude has been allowed to run

HISTORY_FILE="$HOME/.claude/command-history.log"

echo "Recent approved commands:"
tail -n 50 "$HISTORY_FILE" | while read -r cmd; do
  echo "  $cmd"
done

echo -e "\nMost common commands:"
sort "$HISTORY_FILE" | uniq -c | sort -rn | head -10

echo -e "\nRisky patterns to review:"
grep -E "(rm -rf|sudo|curl.*sh|eval)" "$HISTORY_FILE" || echo "  None found"
```

### Test-Driven Development Pattern

```javascript
// Example TDD workflow with Claude
// 1. Write test first
// tests/user-auth.test.js
describe('User Authentication', () => {
  test('should hash password before saving', async () => {
    const user = await User.create({
      email: 'test@example.com',
      password: 'plaintext123'
    });
    
    expect(user.password).not.toBe('plaintext123');
    expect(user.password).toMatch(/^\$2[ayb]\$.{56}$/); // bcrypt format
  });
});

// 2. Ask Claude: "Make this test pass"
// 3. Claude implements the feature
// 4. Ask: "Run tests and show results"
```

## Troubleshooting

### Rate Limit Issues

```bash
# Check current usage
claude /usage

# If rate limited, switch models
# In conversation: "Switch to Sonnet for this task"

# Or use multiple API keys (rotate in ~/.claude.json)
{
  "apiKeys": [
    {"key": "${ANTHROPIC_KEY_1}", "priority": 1},
    {"key": "${ANTHROPIC_KEY_2}", "priority": 2}
  ]
}
```

### Context Window Full

```
# Symptoms: Claude says "context window is full"

# Solutions:
1. /clear and summarize: "Summarize our work, then start fresh"
2. Extract key code to files: "Save important functions to utils.js"
3. Use conversation fork: Fork before context fills up
4. Start new specialized conversation for sub-tasks
```

### Commands Not Executing

```bash
# Check if command approval is blocking
# In CLAUDE.md, add:
auto_approve_commands:
  - "npm test"
  - "git status"
  - "ls -la"

# Or use allowlist patterns:
command_patterns:
  - "npm (test|run|install)"
  - "git (status|log|diff)"
```

### Performance Issues

```bash
# Reduce status line complexity
# In status script, disable expensive operations:
SHOW_GIT_STATUS=false  # Skip git checks if slow
SHOW_DISK_USAGE=false  # Skip df commands

# Minimize CLAUDE.md size
# Keep it under 1000 lines, extract details to separate docs

# Use .claudeignore for large directories
node_modules/
.git/
dist/
build/
*.log
```

## Best Practices Summary

1. **Voice First**: Use voice input for 3-5x faster communication
2. **Context Hygiene**: Compact early and often, break down problems
3. **Git Everything**: Use branches, worktrees, and let Claude handle commits
4. **Test Driven**: Write tests first, let Claude implement
5. **Automate Yourself**: Build scripts and workflows that make you faster
6. **Multi-Agent**: Run parallel Claude sessions for complex projects
7. **Container Isolation**: Use Docker for risky or long-running tasks
8. **Monitor Usage**: Keep /usage open in a tab, watch your limits
9. **Custom Status**: Always know your context, branch, and token usage
10. **Keep Learning**: The tool evolves fast, experiment with new features

## Quick Setup Script

```bash
#!/bin/bash
# setup-claude-workflow.sh - One command to set up optimal workflow

set -e

echo "Setting up Claude Code optimal workflow..."

# Install status line
curl -o ~/.claude-status.sh https://raw.githubusercontent.com/ykdojo/claude-code-tips/main/scripts/context-bar.sh
chmod +x ~/.claude-status.sh

# Create .claude directory
mkdir -p ~/.claude

# Setup CLAUDE.md template
cat > ~/CLAUDE.md.template << 'EOF'
status_line_command: "~/.claude-status.sh"

## Code Style
- Follow project's existing patterns
- Write tests for new features
- Keep functions small and focused

## Workflow
- Always use git branches
- Run tests before committing
- Keep context fresh - summarize when needed
EOF

# Add useful aliases
cat >> ~/.bashrc << 'EOF'
# Claude Code aliases
alias c='claude'
alias cwd='pwd | pbcopy'
alias cnew='claude --new'
EOF

# Install dx plugin (if available)
claude plugin install dx 2>/dev/null || echo "dx plugin not available yet"

echo "Setup complete! Restart your terminal or run: source ~/.bashrc"
echo "Copy ~/CLAUDE.md.template to your project directories"
```

Run with:
```bash
curl -s https://raw.githubusercontent.com/ykdojo/claude-code-tips/main/quick-setup.sh | bash
```
