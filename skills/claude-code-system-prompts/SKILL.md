---
name: claude-code-system-prompts
description: Expert knowledge of Claude Code's system prompts, builtin tools, sub-agents, and prompt engineering for AI coding agents
triggers:
  - "show me claude code's system prompts"
  - "how does claude code's explore agent work"
  - "what tools does claude code have built in"
  - "customize claude code system prompt"
  - "what's in claude code's agent prompts"
  - "how to modify claude code prompts"
  - "explain claude code's builtin tools"
  - "what agents does claude code use"
---

# Claude Code System Prompts

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

Expert knowledge for working with Claude Code's system prompts, builtin tool descriptions, sub-agent prompts, and utility prompts. This skill enables AI agents to understand, reference, and help users customize Claude Code's internal prompt architecture.

## What This Project Provides

The `claude-code-system-prompts` repository contains:

- **All system prompts** used by Claude Code (main agent, sub-agents, utilities)
- **24 builtin tool descriptions** (Write, Bash, Read, etc.)
- **Sub-agent prompts** (Plan, Explore, Task, and 30+ others)
- **Utility prompts** (CLAUDE.md generation, compaction, security review, etc.)
- **System reminders** (~40 conditional prompt fragments)
- **Token counts** for each prompt component
- **Version history** (CHANGELOG.md) tracking prompt changes across 180+ releases

This is updated within minutes of each Claude Code release (currently v2.1.143, May 15 2026).

## Installation & Access

Clone the repository:

```bash
git clone https://github.com/Piebald-AI/claude-code-system-prompts.git
cd claude-code-system-prompts
```

The prompts are organized in `./system-prompts/` as individual markdown files:

```
system-prompts/
├── agent-prompt-explore.md
├── agent-prompt-plan-mode-enhanced.md
├── builtin-tool-write.md
├── main-system-prompt.md
├── system-reminder-*.md
└── ...
```

## Key Components

### 1. Main System Prompt

The core Claude Code agent prompt:

```bash
cat system-prompts/main-system-prompt.md
```

**Token count:** ~8,500 tokens (varies by configuration)

Key sections:
- Identity and capabilities
- Builtin tool usage guidelines
- File operation best practices
- Error handling and recovery
- Code quality standards

### 2. Builtin Tools

Claude Code has 24 builtin tools. Each has a detailed description:

**Key tools:**
- `Write` - File creation/editing (2,100 tokens)
- `Bash` - Command execution (1,800 tokens)
- `Read` - File reading (900 tokens)
- `List` - Directory listing (600 tokens)
- `Search` - Codebase search (1,200 tokens)

Example: View the Write tool description:

```bash
cat system-prompts/builtin-tool-write.md
```

### 3. Sub-Agent Prompts

Claude Code uses specialized sub-agents:

**Explore Agent** (575 tokens):
```bash
cat system-prompts/agent-prompt-explore.md
```

**Plan Agent** (715 tokens):
```bash
cat system-prompts/agent-prompt-plan-mode-enhanced.md
```

**Other agents:**
- General purpose sub-agent (285 tokens)
- Security review agent (2,521 tokens)
- Background job agent (427 tokens)
- Memory synthesis agent (443 tokens)

### 4. System Reminders

~40 conditional prompt fragments added based on context:

```bash
# List all system reminders
ls system-prompts/system-reminder-*.md

# Example: File operation reminder
cat system-prompts/system-reminder-always-use-write-for-file-changes.md
```

Common reminders:
- Always use Write for file changes
- Avoid reading large files unnecessarily
- Use Search before editing unfamiliar code
- Confirm before destructive operations

## Customizing System Prompts

### Using tweakcc (Recommended)

[tweakcc](https://github.com/Piebald-AI/tweakcc) lets you modify prompts and patch your Claude Code installation:

```bash
# Install tweakcc
npm install -g tweakcc

# Export current prompts
tweakcc export

# Edit a prompt
nano tweaks/main-system-prompt.md

# Apply changes
tweakcc apply
```

**Example:** Add custom file operation rules:

```bash
tweakcc export
```

Edit `tweaks/main-system-prompt.md`:

```markdown
## Custom File Rules

- Always create backup files before major refactors
- Use semantic commit messages
- Run tests after file changes
```

Apply:

```bash
tweakcc apply
```

### Direct Prompt Reference

When helping users understand Claude Code behavior, reference specific prompts:

```javascript
// Example: Explaining why Claude uses Write tool
const fs = require('fs');
const writePrompt = fs.readFileSync(
  './system-prompts/builtin-tool-write.md', 
  'utf-8'
);

// Parse to understand Write tool capabilities and constraints
console.log(`Write tool token count: ${writePrompt.split(/\s+/).length}`);
```

## Common Patterns

### 1. Understanding Tool Usage

When a user asks "Why did Claude use X tool?":

```bash
# Find the tool description
cat system-prompts/builtin-tool-${TOOL_NAME}.md

# Check related system reminders
grep -r "tool.*${TOOL_NAME}" system-prompts/system-reminder-*.md
```

### 2. Analyzing Agent Behavior

To understand sub-agent decisions:

```bash
# View agent prompt
cat system-prompts/agent-prompt-explore.md

# Check token budget (shown in file)
# Explore: 575 tokens
# Plan: 715 tokens
```

### 3. Comparing Versions

Use CHANGELOG.md to track prompt evolution:

```bash
# See recent changes
head -n 100 CHANGELOG.md

# Search for specific prompt changes
grep -A 5 "agent-prompt-explore" CHANGELOG.md
```

### 4. Token Budget Analysis

Each prompt file shows token count in the main README:

```javascript
const fs = require('fs');
const readme = fs.readFileSync('README.md', 'utf-8');

// Extract token counts
const tokenCounts = {};
const regex /\[([^\]]+)\]\([^)]+\)\s+\*\*(\d+)\*\*\s+tks/g;
let match;
while ((match = regex.exec(readme)) !== null) {
  tokenCounts[match[1]] = parseInt(match[2]);
}

console.log(tokenCounts);
// { 'Agent Prompt: Explore': 575, 'Agent Prompt: Plan': 715, ... }
```

## Working with Slash Commands

Many slash commands have dedicated agent prompts:

**Security Review** (`/security-review`):
```bash
cat system-prompts/agent-prompt-security-review-slash-command.md
# 2,521 tokens - comprehensive security analysis prompt
```

**Batch Operations** (`/batch`):
```bash
cat system-prompts/agent-prompt-batch-slash-command.md
# 1,106 tokens - parallel codebase changes
```

**PR Review** (`/review-pr`):
```bash
cat system-prompts/agent-prompt-review-pr-slash-command.md
# 211 tokens - GitHub PR analysis
```

## Configuration & Environment

The prompts reference several environment variables and configs:

**Environment variables:**
- `ANTHROPIC_API_KEY` - API authentication
- `CLAUDE_CODE_CONFIG_DIR` - Config location (default: `~/.claude-code`)

**Config files referenced in prompts:**
- `.claudeignore` - File exclusion patterns
- `CLAUDE.md` - Project context
- `.claude/memories/` - Persistent memory files

## Troubleshooting

### Issue: Claude Not Following Custom Rules

1. Check if prompt was properly applied:
```bash
tweakcc diff
```

2. Verify token budget isn't exceeded (sum all active prompts)

3. Check system reminders aren't contradicting your changes:
```bash
grep -r "your-custom-keyword" system-prompts/system-reminder-*.md
```

### Issue: Understanding Unexpected Behavior

1. Find relevant agent prompt:
```bash
# List all agent prompts
ls system-prompts/agent-prompt-*.md

# Search for keywords
grep -l "memory" system-prompts/agent-prompt-*.md
```

2. Check CHANGELOG for recent modifications:
```bash
grep -B 3 -A 10 "version 2.1.143" CHANGELOG.md
```

### Issue: Token Budget Concerns

Calculate total active prompt tokens:

```javascript
const fs = require('fs');
const path = require('path');

const systemPromptsDir = './system-prompts';
let totalTokens = 0;

fs.readdirSync(systemPromptsDir).forEach(file => {
  const content = fs.readFileSync(
    path.join(systemPromptsDir, file), 
    'utf-8'
  );
  const tokens = content.split(/\s+/).length;
  console.log(`${file}: ~${tokens} tokens`);
  totalTokens += tokens;
});

console.log(`\nTotal: ~${totalTokens} tokens`);
```

## Integration Examples

### Example 1: Custom Prompt Validator

Validate custom prompts don't conflict with system reminders:

```javascript
const fs = require('fs');
const path = require('path');

function validateCustomPrompt(customPromptPath) {
  const custom = fs.readFileSync(customPromptPath, 'utf-8').toLowerCase();
  const remindersDir = './system-prompts';
  
  const conflicts = [];
  
  fs.readdirSync(remindersDir)
    .filter(f => f.startsWith('system-reminder-'))
    .forEach(file => {
      const reminder = fs.readFileSync(
        path.join(remindersDir, file), 
        'utf-8'
      ).toLowerCase();
      
      // Check for contradictory keywords
      if (custom.includes('never use write') && 
          reminder.includes('always use write')) {
        conflicts.push({
          file,
          issue: 'Contradicts Write tool requirement'
        });
      }
    });
  
  return conflicts;
}

// Usage
const conflicts = validateCustomPrompt('./my-custom-prompt.md');
if (conflicts.length > 0) {
  console.error('Conflicts found:', conflicts);
}
```

### Example 2: Prompt Documentation Generator

Generate markdown docs from prompt structure:

```javascript
const fs = require('fs');
const path = require('path');

function generatePromptDocs() {
  const readme = fs.readFileSync('README.md', 'utf-8');
  const lines = readme.split('\n');
  
  const docs = {
    agents: [],
    tools: [],
    reminders: []
  };
  
  lines.forEach(line => {
    const match = line.match(/- \[(.*?)\]\((.*?)\) \(\*\*(\d+)\*\* tks\) - (.*)/);
    if (match) {
      const [, name, filepath, tokens, description] = match;
      
      const entry = { name, filepath, tokens: parseInt(tokens), description };
      
      if (filepath.includes('agent-prompt-')) docs.agents.push(entry);
      else if (filepath.includes('builtin-tool-')) docs.tools.push(entry);
      else if (filepath.includes('system-reminder-')) docs.reminders.push(entry);
    }
  });
  
  return docs;
}

// Usage
const docs = generatePromptDocs();
console.log(`Found ${docs.agents.length} agents, ${docs.tools.length} tools`);
```

### Example 3: Prompt Diff Analyzer

Compare prompt changes across versions:

```bash
#!/bin/bash
# compare-prompts.sh

VERSION_OLD="v2.1.140"
VERSION_NEW="v2.1.143"

git show $VERSION_OLD:system-prompts/main-system-prompt.md > /tmp/old.md
git show $VERSION_NEW:system-prompts/main-system-prompt.md > /tmp/new.md

diff -u /tmp/old.md /tmp/new.md | grep "^[+-]" | grep -v "^[+-][+-][+-]"
```

## Advanced Usage

### Extracting Prompts from Source

The prompts in this repo are extracted from Claude Code's compiled source. To understand the extraction:

```javascript
// Conceptual example - actual extraction is more complex
const claudeCodeSource = fs.readFileSync(
  'node_modules/@anthropic-ai/claude-code/dist/index.js',
  'utf-8'
);

// Prompts are embedded as strings in the compiled JS
// Look for patterns like:
const promptPattern = /systemPrompt:\s*[`"'](.{100,})[`"']/g;
```

### Memory System Prompts

Claude Code uses persistent memory. Key prompts:

**Memory Synthesis** (443 tokens):
```bash
cat system-prompts/agent-prompt-memory-synthesis.md
```

**Dream Consolidation** (859 tokens):
```bash
cat system-prompts/agent-prompt-dream-memory-consolidation.md
```

**Memory Pruning** (456 tokens):
```bash
cat system-prompts/agent-prompt-dream-memory-pruning.md
```

## Resources

- **Repository:** https://github.com/Piebald-AI/claude-code-system-prompts
- **tweakcc Tool:** https://github.com/Piebald-AI/tweakcc
- **Piebald (Alternative):** https://piebald.ai
- **Changelog:** https://github.com/Piebald-AI/claude-code-system-prompts/blob/main/CHANGELOG.md
- **X Updates:** https://x.com/PiebaldAI

## License

MIT - Same as the source repository.
