---
name: collection-claude-code-source-code
description: Reference collection of Claude Code's source architecture, reimplementations, and analysis for building AI coding agents
triggers:
  - how does Claude Code's architecture work
  - show me Claude Code source code analysis
  - what are the core components of Claude Code
  - how to build a coding agent like Claude Code
  - explain Claude Code's tool system
  - I want to understand Claude Code internals
  - help me implement a coding agent
  - show me Claude Code reimplementation examples
---

# collection-claude-code-source-code

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## Overview

This repository is the definitive academic research collection for understanding how Anthropic's Claude Code works internally. It contains:

- **original-source-code**: Raw leaked TypeScript source (1,884 files)
- **claude-code-source-code**: Decompiled v2.1.88 source with extensive documentation (163,318 lines)
- **claw-code**: Python clean-room architectural rewrite
- **nano-claude-code**: Minimal multi-provider reimplementation

Use this collection to understand agent architecture patterns, tool systems, memory management, and multi-agent coordination for building AI coding assistants.

## Installation

```bash
# Clone the repository
git clone https://github.com/chauncygu/collection-claude-code-source-code.git
cd collection-claude-code-source-code

# The main source is in subdirectories - no build required for reading
# For running the Python reimplementations:
cd claw-code  # or nano-claude-code
pip install -e .
```

## Repository Structure

```
collection-claude-code-source-code/
├── original-source-code/       # Raw leaked source
│   └── src/                    # 1,884 TypeScript files
├── claude-code-source-code/    # Annotated decompiled source
│   ├── src/                    # Full TypeScript codebase
│   └── docs/                   # Analysis documents (EN/ZH)
├── claw-code/                  # Python rewrite (109 files)
└── nano-claude-code/           # Minimal Python version (~30 files)
```

## Core Architecture Patterns

### Main Agent Loop (query.ts)

The heart of Claude Code is the `query()` function:

```typescript
// Simplified from claude-code-source-code/src/query.ts
export async function* query(
  userInput: string,
  context: QueryContext
): AsyncGenerator<SDKMessage> {
  // 1. Assemble system prompt
  const systemPrompts = await fetchSystemPromptParts(context);
  
  // 2. Create streaming executor
  const executor = new StreamingToolExecutor({
    tools: registeredTools,
    maxParallel: 3
  });
  
  // 3. Start agent loop
  while (!isDone) {
    const response = await anthropic.messages.create({
      model: "claude-3-7-sonnet",
      system: systemPrompts,
      messages: history,
      tools: executor.getToolDefinitions(),
      stream: true
    });
    
    // 4. Execute tool calls in parallel
    for await (const toolCall of response.toolCalls) {
      const result = await executor.execute(toolCall);
      yield result;
    }
    
    // 5. Auto-compact context when needed
    if (shouldCompact(context)) {
      await autoCompact(context);
    }
  }
}
```

### Tool System Architecture

Claude Code uses 40+ tools organized by category:

```typescript
// From claude-code-source-code/src/tools.ts
import { buildTool } from './Tool';

// Example tool definition
export const readFileTool = buildTool({
  name: 'read_file',
  description: 'Read contents of a file',
  parameters: {
    type: 'object',
    properties: {
      path: { type: 'string', description: 'File path' }
    },
    required: ['path']
  },
  execute: async (params) => {
    const content = await fs.readFile(params.path, 'utf-8');
    return { content, size: content.length };
  }
});

// Tool categories
export const toolPresets = {
  filesystem: [readFileTool, writeFileTool, listDirectoryTool],
  code: [grepTool, searchDefinitionTool, lintTool],
  terminal: [executeBashTool, checkProcessTool],
  memory: [vectorSearchTool, saveMemoryTool],
  // ... 10+ categories total
};
```

### Memory System (7 Layers)

```typescript
// From claude-code-source-code/src/memdir/
interface MemoryLayer {
  shortTerm: Message[];          // Recent conversation
  workingContext: FileContext[]; // Active files
  episodic: TaskMemory[];        // Task history
  semantic: VectorStore;         // Searchable knowledge
  procedural: Skill[];           // Learned patterns
  metacognitive: Analytics;      // Self-reflection
  dreaming: BackgroundProcessor; // Offline consolidation
}

// Memory search example
const searchMemory = async (query: string) => {
  return await vectorStore.search(query, {
    layers: ['semantic', 'episodic'],
    limit: 5,
    threshold: 0.7
  });
};
```

### Slash Commands

```typescript
// From claude-code-source-code/src/commands.ts
export const commands = {
  '/model': switchModel,
  '/context': manageContext,
  '/reset': resetSession,
  '/approve': approveCommand,
  '/deny': denyCommand,
  '/memory': queryMemory,
  '/task': manageTask,
  '/voice': toggleVoice,
  // ... 87 total commands
};

// Example command implementation
const switchModel = async (args: string[]) => {
  const model = args[0] || 'claude-3-7-sonnet';
  config.set('model', model);
  return `Switched to ${model}`;
};
```

## Building Your Own Coding Agent

### Using the Nano Implementation

The minimal Python rewrite shows the core pattern:

```python
# From nano-claude-code/src/agent.py
import anthropic
from typing import AsyncGenerator

class CodingAgent:
    def __init__(self, api_key: str = None):
        self.client = anthropic.Anthropic(
            api_key=api_key or os.getenv("ANTHROPIC_API_KEY")
        )
        self.tools = self._load_tools()
        self.history = []
    
    async def query(self, user_input: str) -> AsyncGenerator:
        """Main agent loop"""
        self.history.append({
            "role": "user",
            "content": user_input
        })
        
        while True:
            response = await self.client.messages.create(
                model="claude-3-5-sonnet-20241022",
                max_tokens=8192,
                system=self._build_system_prompt(),
                messages=self.history,
                tools=self.tools,
                stream=True
            )
            
            # Handle tool calls
            tool_results = []
            async for event in response:
                if event.type == "tool_use":
                    result = await self._execute_tool(event)
                    tool_results.append(result)
                    yield result
                elif event.type == "text":
                    yield event.text
            
            if not tool_results:
                break
            
            self.history.append({
                "role": "assistant",
                "content": tool_results
            })
    
    def _load_tools(self):
        """Load tool definitions"""
        return [
            {
                "name": "execute_command",
                "description": "Run a shell command",
                "input_schema": {
                    "type": "object",
                    "properties": {
                        "command": {"type": "string"}
                    },
                    "required": ["command"]
                }
            },
            # Add more tools...
        ]
```

### Using with Multiple Providers

```python
# From nano-claude-code/src/providers.py
from abc import ABC, abstractmethod

class LLMProvider(ABC):
    @abstractmethod
    async def create_message(self, **kwargs):
        pass

class AnthropicProvider(LLMProvider):
    def __init__(self, api_key: str = None):
        self.client = anthropic.Anthropic(
            api_key=api_key or os.getenv("ANTHROPIC_API_KEY")
        )
    
    async def create_message(self, **kwargs):
        return await self.client.messages.create(**kwargs)

class OpenAIProvider(LLMProvider):
    def __init__(self, api_key: str = None):
        import openai
        self.client = openai.AsyncOpenAI(
            api_key=api_key or os.getenv("OPENAI_API_KEY")
        )
    
    async def create_message(self, **kwargs):
        # Translate to OpenAI format
        response = await self.client.chat.completions.create(
            model=kwargs.get("model", "gpt-4"),
            messages=kwargs["messages"],
            tools=self._convert_tools(kwargs.get("tools", [])),
            stream=kwargs.get("stream", False)
        )
        return response

# Usage
agent = CodingAgent(provider=AnthropicProvider())
# or
agent = CodingAgent(provider=OpenAIProvider())
```

## Key Configuration Patterns

### Environment Setup

```bash
# Required
export ANTHROPIC_API_KEY=your_key_here

# Optional - for multi-provider support
export OPENAI_API_KEY=your_key_here
export GOOGLE_AI_API_KEY=your_key_here
export GROQ_API_KEY=your_key_here

# Memory configuration
export CLAUDE_MEMORY_DIR=~/.claude/memory
export CLAUDE_VECTOR_DB=chromadb  # or qdrant, pinecone
```

### Agent Configuration

```python
# From nano-claude-code/config.yaml
agent:
  model: claude-3-5-sonnet-20241022
  max_tokens: 8192
  temperature: 0.7
  max_iterations: 50

tools:
  enabled:
    - execute_command
    - read_file
    - write_file
    - search_code
    - git_operations
  
  permissions:
    require_approval:
      - execute_command
      - write_file
      - git_push

memory:
  vector_store: chromadb
  embedding_model: text-embedding-3-small
  max_context_files: 20
  compression_threshold: 0.8

safety:
  sandbox_mode: true
  allowed_paths: 
    - ./
  blocked_commands:
    - rm -rf /
    - sudo
```

## Common Usage Patterns

### Basic File Operation Agent

```python
from nano_claude_code import CodingAgent

async def main():
    agent = CodingAgent()
    
    # Simple file read
    async for response in agent.query("Read the package.json file"):
        print(response)
    
    # Code modification
    async for response in agent.query(
        "Add error handling to the authenticate function in auth.py"
    ):
        print(response)
    
    # Multi-step task
    async for response in agent.query(
        "Create a new feature: add rate limiting to the API"
    ):
        print(response)

import asyncio
asyncio.run(main())
```

### Custom Tool Implementation

```python
from nano_claude_code import CodingAgent, Tool

class CustomTool(Tool):
    def __init__(self):
        super().__init__(
            name="analyze_dependencies",
            description="Analyze project dependencies for security issues",
            parameters={
                "type": "object",
                "properties": {
                    "package_file": {
                        "type": "string",
                        "description": "Path to package.json or requirements.txt"
                    }
                },
                "required": ["package_file"]
            }
        )
    
    async def execute(self, params):
        import subprocess
        result = subprocess.run(
            ["npm", "audit", "--json"],
            capture_output=True,
            text=True
        )
        return {
            "vulnerabilities": result.stdout,
            "status": "success"
        }

# Register custom tool
agent = CodingAgent()
agent.register_tool(CustomTool())
```

### Multi-Agent Coordination

```python
# From claw-code/src/coordinator.py
class AgentCoordinator:
    def __init__(self):
        self.agents = {
            "planner": CodingAgent(role="planner"),
            "executor": CodingAgent(role="executor"),
            "reviewer": CodingAgent(role="reviewer")
        }
    
    async def execute_task(self, task: str):
        # 1. Planning phase
        plan = await self.agents["planner"].query(
            f"Create a step-by-step plan for: {task}"
        )
        
        # 2. Execution phase
        results = []
        async for step in plan.steps:
            result = await self.agents["executor"].query(step)
            results.append(result)
        
        # 3. Review phase
        review = await self.agents["reviewer"].query(
            f"Review these changes: {results}"
        )
        
        return review

# Usage
coordinator = AgentCoordinator()
await coordinator.execute_task("Refactor the authentication system")
```

## Advanced Patterns

### Memory-Augmented Agent

```python
from nano_claude_code import CodingAgent, VectorMemory

class MemoryAgent(CodingAgent):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.memory = VectorMemory(
            persist_dir=os.getenv("CLAUDE_MEMORY_DIR")
        )
    
    async def query(self, user_input: str):
        # Search relevant memories
        memories = await self.memory.search(user_input, limit=5)
        
        # Augment system prompt with memories
        context = "\n".join([
            f"Relevant memory: {m.content}"
            for m in memories
        ])
        
        # Execute with enhanced context
        async for response in super().query(
            f"{context}\n\nUser request: {user_input}"
        ):
            yield response
        
        # Save new memory
        await self.memory.save(user_input, response)
```

### Skill Learning System

```python
# From claw-code/src/skills.py
class SkillManager:
    def __init__(self, agent: CodingAgent):
        self.agent = agent
        self.skills = self._load_skills()
    
    def _load_skills(self):
        """Load built-in skills"""
        return {
            "git_workflow": Skill(
                name="git_workflow",
                pattern="create feature branch and PR",
                steps=[
                    "git checkout -b feature/{name}",
                    "# make changes",
                    "git add .",
                    "git commit -m '{message}'",
                    "git push origin feature/{name}",
                    "gh pr create"
                ]
            ),
            # More skills...
        }
    
    async def execute_skill(self, skill_name: str, params: dict):
        skill = self.skills[skill_name]
        for step in skill.steps:
            formatted_step = step.format(**params)
            await self.agent.execute_tool({
                "name": "execute_command",
                "input": {"command": formatted_step}
            })
```

## Troubleshooting

### Token Limit Issues

```python
# Auto-compact context when approaching limits
class SmartAgent(CodingAgent):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.max_tokens = 8192
    
    async def query(self, user_input: str):
        if self._estimate_tokens() > self.max_tokens * 0.8:
            await self._compact_context()
        
        async for response in super().query(user_input):
            yield response
    
    def _estimate_tokens(self):
        """Rough token count estimation"""
        return sum(len(m["content"]) // 4 for m in self.history)
    
    async def _compact_context(self):
        """Compress context using Claude"""
        summary = await self.client.messages.create(
            model="claude-3-haiku-20240307",  # Cheaper model
            messages=[{
                "role": "user",
                "content": f"Summarize this conversation:\n{self.history}"
            }]
        )
        self.history = [{
            "role": "system",
            "content": summary.content
        }]
```

### Rate Limiting

```python
import asyncio
from functools import wraps

def rate_limit(calls_per_minute: int):
    min_interval = 60.0 / calls_per_minute
    last_called = [0.0]
    
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            elapsed = asyncio.get_event_loop().time() - last_called[0]
            wait_time = max(0, min_interval - elapsed)
            if wait_time > 0:
                await asyncio.sleep(wait_time)
            
            last_called[0] = asyncio.get_event_loop().time()
            return await func(*args, **kwargs)
        return wrapper
    return decorator

class RateLimitedAgent(CodingAgent):
    @rate_limit(calls_per_minute=50)
    async def query(self, user_input: str):
        async for response in super().query(user_input):
            yield response
```

### Debug Mode

```python
import logging

logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger(__name__)

class DebugAgent(CodingAgent):
    async def query(self, user_input: str):
        logger.debug(f"User input: {user_input}")
        logger.debug(f"Current history length: {len(self.history)}")
        
        async for response in super().query(user_input):
            logger.debug(f"Response chunk: {response}")
            yield response
        
        logger.debug(f"Tools called: {self._get_tool_usage()}")
```

## Further Reading

- **Architecture Analysis**: See `claude-code-source-code/docs/en/` for detailed breakdowns
- **Research Report**: `claude-code-source-code/docs/claude-code-deep-dive-xelatex.pdf`
- **Hacker News Discussion**: Links in main README news section
- **Video Analysis**: YouTube links in news section (Chinese)

## References

- Original TypeScript source: `original-source-code/src/`
- Decompiled annotated source: `claude-code-source-code/src/`
- Python reimplementation: `claw-code/` and `nano-claude-code/`
- Tool definitions: `claude-code-source-code/src/tools/`
- Memory system: `claude-code-source-code/src/memdir/`

This skill provides the foundational knowledge for understanding and building AI coding agents based on Claude Code's proven architecture.
