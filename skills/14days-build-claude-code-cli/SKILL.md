---
name: 14days-build-claude-code-cli
description: Build a Claude Code-style AI agent CLI from scratch in Python with tool calling, file editing, bash execution, and MCP integration
triggers:
  - how do I build an AI code agent CLI
  - teach me to create a tool-calling agent
  - implement file editing tools for my agent
  - add bash execution to my AI assistant
  - create an agent harness with permissions
  - build a code agent with session memory
  - implement MCP protocol in my agent
  - add subagents and skills to my CLI
---

# 14days-build-claude-code-cli

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

A 14-day tutorial project teaching you to build a production-style AI code agent CLI (`agent-code`) in Python. Learn the harness architecture: tool calling, file operations, bash execution, permissions, session memory, hooks, skills, subagents, worktree isolation, and MCP integration.

## What This Project Does

This is an educational implementation of a Claude Code-style agent harness. You'll build `agent-code`, a CLI that:

- Accepts one-shot prompts or enters an interactive REPL
- Calls real LLMs (default: DeepSeek via Anthropic-compatible endpoint)
- Uses tool calling (`tool_use` / `tool_result`) with file operations, web search, and bash
- Implements safety: read-before-edit, diff preview, permission system
- Manages sessions, project memory, and context compression
- Supports slash commands, hooks, skills, subagents, and MCP tools

**Core philosophy**: The model handles reasoning; the harness handles context, tools, permissions, execution, state, and feedback loops.

## Installation

Each day is a standalone package. To follow the tutorial from Day 1:

```bash
# Start your own project
mkdir my-agent-code && cd my-agent-code
uv init
uv add anthropic typer rich aiofiles

# Follow docs/day-01-hello-agent.md
```

To run a reference snapshot:

```bash
git clone https://github.com/bozhouDev/14days-build-claude-code-cli
cd 14days-build-claude-code-cli/packages/day-08-interactive-shell-plan-mode
uv sync
uv run agent-code "list files in this project"
```

**Web tutorial**: Visit https://buildcc.dev or run locally:

```bash
cd agent-code-learn
npm install && npm run dev
# Open http://localhost:3000
```

## Configuration

Set environment variables for the LLM provider:

```bash
# Default: DeepSeek with Anthropic-compatible API
export ANTHROPIC_AUTH_TOKEN="sk-your-deepseek-key"
export ANTHROPIC_BASE_URL="https://api.deepseek.com/anthropic"

# Or use official Claude
export ANTHROPIC_AUTH_TOKEN="sk-ant-..."
export ANTHROPIC_BASE_URL="https://api.anthropic.com"
```

Override model in CLI:

```bash
uv run agent-code --model deepseek-v4-flash "your task"
uv run agent-code --model claude-3-7-sonnet-20250219 "your task"
```

## 14-Day Learning Path

| Day | Topic | Key Harness Concepts |
|-----|-------|---------------------|
| 1 | Hello Agent | CLI, REPL, MockProvider, minimal Agent Loop |
| 2 | Real Model + Tool Calling | AnthropicProvider, `tool_use` / `tool_result` |
| 3 | File + Web Tools | cwd boundary, file read/search, web tools |
| 4 | Safe Edit | read-before-edit, string replacement, diff preview |
| 5 | Bash + Permission | command execution, permission system, background tasks |
| 6 | Session + Memory | session JSONL, project memory, memdir |
| 7 | Slash + Hooks | slash commands, hooks, cron `/loop` |
| 8 | Interactive Shell + Plan Mode | interactive shell, TodoWrite, plan approval |
| 9 | Skills | load domain knowledge and workflows on demand |
| 10 | Subagents | delegate tasks to specialized subagents |
| 11 | Context Compact | long context compression, cost tracking |
| 12 | Agent Coordinator | lightweight multi-agent orchestration |
| 13 | Worktree + Final Demo | worktree isolation, end-to-end code tasks |
| 14 | MCP + ToolSearch | MCP client, tool discovery, ecosystem extension |

## Key Commands

### Basic Usage

```bash
# One-shot prompt
uv run agent-code "create a hello.py file"

# Interactive REPL
uv run agent-code

# Specify model
uv run agent-code --model deepseek-v4-flash "analyze this codebase"

# Resume session
uv run agent-code --session my-session-id
```

### Interactive REPL Commands (Day 7+)

```
> /help              # List all slash commands
> /quit              # Exit REPL
> /new               # Start new session
> /save my-session   # Save current session
> /load my-session   # Load previous session
> /clear             # Clear context
> /tools             # List available tools
> /loop 5m "task"    # Run task every 5 minutes
```

### Plan Mode (Day 8+)

```bash
# Enable plan mode for multi-step tasks
uv run agent-code --plan-mode "refactor auth module"

# Agent creates todo list, you approve each step
```

## Core Architecture Examples

### Day 1-2: Minimal Agent Loop with Tool Calling

```python
# src/agent_code/providers/anthropic_provider.py
from anthropic import Anthropic
from typing import List, Dict

class AnthropicProvider:
    def __init__(self, model: str = "deepseek-v4-flash"):
        self.client = Anthropic()
        self.model = model
    
    def chat(self, messages: List[Dict], tools: List[Dict] = None) -> Dict:
        response = self.client.messages.create(
            model=self.model,
            messages=messages,
            tools=tools or [],
            max_tokens=8192
        )
        return response.model_dump()

# src/agent_code/core/agent.py
class Agent:
    def __init__(self, provider, tool_registry):
        self.provider = provider
        self.tools = tool_registry
        self.messages = []
    
    def run(self, user_input: str):
        self.messages.append({"role": "user", "content": user_input})
        
        while True:
            response = self.provider.chat(
                self.messages, 
                self.tools.get_tool_schemas()
            )
            
            if response["stop_reason"] == "end_turn":
                break
            
            # Handle tool calls
            for block in response["content"]:
                if block["type"] == "tool_use":
                    result = self.tools.execute(
                        block["name"], 
                        block["input"]
                    )
                    self.messages.append({
                        "role": "user",
                        "content": [{
                            "type": "tool_result",
                            "tool_use_id": block["id"],
                            "content": result
                        }]
                    })
```

### Day 3: File Tools with Safety

```python
# src/agent_code/tools/file_tools.py
import os
from pathlib import Path

class FileTools:
    def __init__(self, workspace_root: Path):
        self.workspace = workspace_root
    
    def _check_path(self, path: str) -> Path:
        """Ensure path is within workspace."""
        full_path = (self.workspace / path).resolve()
        if not str(full_path).startswith(str(self.workspace)):
            raise ValueError(f"Path {path} outside workspace")
        return full_path
    
    def read_file(self, path: str) -> str:
        """Read file contents safely."""
        file_path = self._check_path(path)
        return file_path.read_text()
    
    def list_files(self, pattern: str = "*") -> List[str]:
        """List files matching pattern."""
        return [
            str(p.relative_to(self.workspace))
            for p in self.workspace.rglob(pattern)
            if p.is_file()
        ]
```

### Day 4: Safe Edit with Diff Preview

```python
# src/agent_code/tools/edit_tools.py
from difflib import unified_diff

class EditTools:
    def __init__(self, file_tools):
        self.files = file_tools
    
    def edit_file(self, path: str, old_text: str, new_text: str) -> str:
        """Replace text with read-before-edit validation."""
        # 1. Read current content
        current = self.files.read_file(path)
        
        # 2. Validate old_text exists
        if old_text not in current:
            raise ValueError(f"old_text not found in {path}")
        
        # 3. Generate diff preview
        updated = current.replace(old_text, new_text, 1)
        diff = "\n".join(unified_diff(
            current.splitlines(),
            updated.splitlines(),
            fromfile=path,
            tofile=path,
            lineterm=""
        ))
        
        # 4. Apply change
        self.files._check_path(path).write_text(updated)
        
        return f"Applied:\n{diff}"
```

### Day 5: Bash Tool with Permissions

```python
# src/agent_code/tools/bash_tools.py
import subprocess
from typing import Set

DANGEROUS_COMMANDS = {"rm", "sudo", "chmod", "mv"}

class BashTools:
    def __init__(self, permission_callback):
        self.ask_permission = permission_callback
        self.allowed_commands: Set[str] = set()
    
    def execute_bash(self, command: str) -> str:
        """Execute bash command with permission check."""
        first_word = command.split()[0]
        
        if first_word in DANGEROUS_COMMANDS:
            if first_word not in self.allowed_commands:
                if not self.ask_permission(f"Allow: {command}?"):
                    return "Permission denied"
                self.allowed_commands.add(first_word)
        
        result = subprocess.run(
            command,
            shell=True,
            capture_output=True,
            text=True,
            timeout=30
        )
        
        return result.stdout + result.stderr
```

### Day 6: Session Persistence

```python
# src/agent_code/core/session.py
import json
from pathlib import Path
from datetime import datetime

class Session:
    def __init__(self, session_dir: Path):
        self.dir = session_dir
        self.dir.mkdir(parents=True, exist_ok=True)
    
    def save(self, session_id: str, messages: List[Dict]):
        """Save session to JSONL."""
        file = self.dir / f"{session_id}.jsonl"
        with file.open("w") as f:
            for msg in messages:
                f.write(json.dumps(msg) + "\n")
    
    def load(self, session_id: str) -> List[Dict]:
        """Load session from JSONL."""
        file = self.dir / f"{session_id}.jsonl"
        if not file.exists():
            return []
        
        messages = []
        with file.open() as f:
            for line in f:
                messages.append(json.loads(line))
        return messages
    
    def list_sessions(self) -> List[str]:
        """List all saved sessions."""
        return [
            p.stem for p in self.dir.glob("*.jsonl")
        ]
```

### Day 7: Slash Commands

```python
# src/agent_code/cli/slash_commands.py
from typing import Callable, Dict

class SlashRouter:
    def __init__(self):
        self.commands: Dict[str, Callable] = {}
    
    def register(self, name: str, handler: Callable):
        self.commands[name] = handler
    
    def handle(self, user_input: str) -> bool:
        """Return True if handled as slash command."""
        if not user_input.startswith("/"):
            return False
        
        parts = user_input[1:].split(maxsplit=1)
        cmd = parts[0]
        args = parts[1] if len(parts) > 1 else ""
        
        if cmd in self.commands:
            self.commands[cmd](args)
            return True
        
        return False

# Usage in REPL
slash = SlashRouter()
slash.register("help", lambda _: print("Available: /help, /quit, /save"))
slash.register("save", lambda args: session.save(args, agent.messages))

while True:
    user_input = input("> ")
    if slash.handle(user_input):
        continue
    agent.run(user_input)
```

### Day 8: Plan Mode with TodoWrite

```python
# src/agent_code/tools/todo_tools.py
class TodoTools:
    def __init__(self):
        self.todos = []
        self.current_index = 0
    
    def write_todos(self, tasks: List[str]) -> str:
        """Agent writes plan as todo list."""
        self.todos = tasks
        self.current_index = 0
        return f"Created plan with {len(tasks)} steps"
    
    def get_next_todo(self) -> str:
        """Get next unapproved task."""
        if self.current_index >= len(self.todos):
            return "No more tasks"
        return self.todos[self.current_index]
    
    def approve_todo(self):
        """User approves current task."""
        self.current_index += 1

# In agent loop (plan mode enabled)
def run_with_plan_mode(agent, user_input):
    # 1. Agent creates plan
    agent.run(f"Create a todo list for: {user_input}")
    
    # 2. Execute each todo with approval
    while True:
        todo = todo_tools.get_next_todo()
        if todo == "No more tasks":
            break
        
        print(f"\nNext task: {todo}")
        if input("Approve? [y/n]: ").lower() != "y":
            break
        
        todo_tools.approve_todo()
        agent.run(f"Execute approved task: {todo}")
```

## Common Patterns

### Tool Registry Pattern

```python
class ToolRegistry:
    def __init__(self):
        self.tools = {}
    
    def register(self, name: str, fn: Callable, schema: Dict):
        self.tools[name] = {"fn": fn, "schema": schema}
    
    def get_tool_schemas(self) -> List[Dict]:
        return [t["schema"] for t in self.tools.values()]
    
    def execute(self, name: str, params: Dict) -> str:
        if name not in self.tools:
            raise ValueError(f"Unknown tool: {name}")
        return self.tools[name]["fn"](**params)

# Register all tools
registry = ToolRegistry()
registry.register("read_file", file_tools.read_file, {
    "name": "read_file",
    "description": "Read file contents",
    "input_schema": {
        "type": "object",
        "properties": {"path": {"type": "string"}},
        "required": ["path"]
    }
})
```

### Permission System Pattern

```python
class PermissionEngine:
    def __init__(self):
        self.rules = {}
    
    def check(self, action: str, context: Dict) -> bool:
        """Check if action is allowed."""
        if action in self.rules:
            return self.rules[action](context)
        
        # Default: ask user
        response = input(f"Allow {action}? [y/n]: ")
        return response.lower() == "y"

# Usage
permissions = PermissionEngine()
permissions.rules["delete_file"] = lambda ctx: ctx["path"] != "main.py"

if permissions.check("delete_file", {"path": "temp.txt"}):
    os.remove("temp.txt")
```

## Testing

Each day includes tests. Run from the specific day's directory:

```bash
cd packages/day-02-real-model-tool-calling
uv run pytest

# Run specific test
uv run pytest tests/test_anthropic_provider.py -v

# Run all tests for a day
uv run pytest tests/
```

Example test structure:

```python
# tests/test_file_tools.py
def test_read_file_within_workspace(tmp_path):
    tools = FileTools(tmp_path)
    test_file = tmp_path / "test.txt"
    test_file.write_text("hello")
    
    content = tools.read_file("test.txt")
    assert content == "hello"

def test_read_file_outside_workspace_raises(tmp_path):
    tools = FileTools(tmp_path)
    
    with pytest.raises(ValueError, match="outside workspace"):
        tools.read_file("../etc/passwd")
```

## Troubleshooting

**API key issues:**

```bash
# Verify env vars
echo $ANTHROPIC_AUTH_TOKEN
echo $ANTHROPIC_BASE_URL

# Test with curl
curl https://api.deepseek.com/anthropic/v1/messages \
  -H "x-api-key: $ANTHROPIC_AUTH_TOKEN" \
  -H "anthropic-version: 2023-06-01" \
  -d '{"model":"deepseek-v4-flash","messages":[{"role":"user","content":"hi"}],"max_tokens":100}'
```

**Tool call not working:**

Check tool schema matches Anthropic format:

```python
{
    "name": "tool_name",
    "description": "Clear description",
    "input_schema": {
        "type": "object",
        "properties": {
            "param": {"type": "string", "description": "What it does"}
        },
        "required": ["param"]
    }
}
```

**File path errors:**

Always resolve paths relative to workspace:

```python
# BAD
with open(user_path) as f:  # Could be /etc/passwd

# GOOD
safe_path = (workspace_root / user_path).resolve()
if not str(safe_path).startswith(str(workspace_root)):
    raise ValueError("Path outside workspace")
```

**Session not persisting:**

Ensure session directory exists and is writable:

```python
session_dir = Path.home() / ".agent-code" / "sessions"
session_dir.mkdir(parents=True, exist_ok=True)
```

**Model context overflow:**

Implement context trimming (Day 11):

```python
def trim_messages(messages: List[Dict], max_tokens: int = 100000):
    # Keep system message and recent turns
    system = messages[0] if messages[0]["role"] == "system" else None
    recent = messages[-10:]  # Last 10 turns
    
    return ([system] if system else []) + recent
```

## Advanced: MCP Integration (Day 14)

The Model Context Protocol (MCP) lets agents discover and use external tools:

```python
# src/agent_code/mcp/client.py
import httpx
from typing import List, Dict

class MCPClient:
    def __init__(self, server_url: str):
        self.url = server_url
    
    async def list_tools(self) -> List[Dict]:
        """Discover tools from MCP server."""
        async with httpx.AsyncClient() as client:
            response = await client.get(f"{self.url}/tools")
            return response.json()["tools"]
    
    async def call_tool(self, name: str, params: Dict) -> str:
        """Call remote tool via MCP."""
        async with httpx.AsyncClient() as client:
            response = await client.post(
                f"{self.url}/tools/{name}",
                json=params
            )
            return response.json()["result"]

# Integrate with agent
mcp = MCPClient("http://localhost:3000")
tools = await mcp.list_tools()

for tool in tools:
    registry.register(
        tool["name"],
        lambda **params: asyncio.run(mcp.call_tool(tool["name"], params)),
        tool
    )
```

## Further Learning

- Follow the 14-day tutorial in order: `docs/day-01-hello-agent.md` → `docs/day-14-mcp-toolsearch.md`
- Study reference snapshots: `packages/day-*/` for working code
- Read Claude Code architecture: The harness separates model reasoning from execution, permissions, and context management
- Experiment with custom tools: Add domain-specific tools to the registry
- Extend with MCP: Connect to MCP servers for GitHub, databases, etc.

**Web tutorial**: https://buildcc.dev for interactive learning experience.
