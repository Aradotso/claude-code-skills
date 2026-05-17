---
name: claude-code-local-mlx
description: Run Claude Code 100% on-device with local AI on Apple Silicon using MLX-native models (Qwen 3.5 122B, Llama 3.3 70B, Gemma 4 31B, DeepSeek V4 Flash)
triggers:
  - set up local claude code
  - run claude code offline
  - install mlx local ai
  - configure qwen llama gemma locally
  - run deepseek v4 flash locally
  - setup airgap ai on apple silicon
  - run anthropic api server locally
  - configure mlx-lm with claude code
---

# Claude Code Local — MLX On-Device AI

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

Run Claude Code 100% on-device with local AI on Apple Silicon. This project provides an MLX-native Anthropic-API-compatible server that lets you use powerful local models (Qwen 3.5 122B at 65 tok/s, Llama 3.3 70B, Gemma 4 31B, DeepSeek V4 Flash with 1M context) as drop-in replacements for Claude. Built for privacy-critical workflows (NDA, legal, healthcare) where data cannot leave the device.

## What It Does

- **MLX-native inference** optimized for Apple Silicon (M1/M2/M3/M4)
- **Anthropic API compatibility** — point Claude Code clients at `localhost:8000`
- **Four model options**: Gemma 4 31B (fast), Qwen 3.5 122B (balanced), Llama 3.3 70B (dense), DeepSeek V4 Flash (1M context)
- **100% offline** — works in airplane mode, airgap-ready
- **Voice mode** — hands-free coding with on-device STT/TTS
- **Browser agent** — remote access from any device on your LAN

## Installation

### Prerequisites

```bash
# macOS with Apple Silicon (M1/M2/M3/M4)
# Minimum RAM:
#   - Gemma 4 31B: 32 GB
#   - Qwen 3.5 122B: 96 GB
#   - Llama 3.3 70B: 96 GB
#   - DeepSeek V4 Flash: 128 GB

# Python 3.10+
python3 --version

# Homebrew (for dependencies)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### Quick Start (3 Commands)

```bash
# 1. Clone the repo
git clone https://github.com/nicedreamzapp/claude-code-local.git
cd claude-code-local

# 2. Install dependencies
pip install -r requirements.txt

# 3. Start the server (Qwen 3.5 122B example)
bash scripts/start-mlx-server.sh
```

The server will:
1. Download the model from HuggingFace (~65 GB for Qwen)
2. Start an Anthropic-API-compatible server on `http://localhost:8000`
3. Print usage instructions

### Model Selection

Set `MLX_MODEL` environment variable before starting:

```bash
# Gemma 4 31B (fast, 32GB+ RAM)
export MLX_MODEL="bartowski/gemma-4-code-it-31b-MLX-4bit"
bash scripts/start-mlx-server.sh

# Qwen 3.5 122B (balanced, 96GB+ RAM)
export MLX_MODEL="mlx-community/Qwen3.5-MoE-A10B-4bit"
bash scripts/start-mlx-server.sh

# Llama 3.3 70B (dense, 96GB+ RAM)
export MLX_MODEL="divinetribe/Llama-3.3-70B-Instruct-abliterated-8bit-mlx"
bash scripts/start-mlx-server.sh

# DeepSeek V4 Flash (1M context, 128GB+ RAM) — uses ds4 engine
# See DeepSeek section below
```

### DeepSeek V4 Flash Setup

DeepSeek uses the separate `ds4` engine:

```bash
# 1. Install ds4
git clone https://github.com/antirez/ds4.git ~/.ds4
cd ~/.ds4
make

# 2. Download model (81GB for q2, 153GB for q4)
huggingface-cli download antirez/deepseek-v4-gguf \
  deepseek-v4-q2.gguf --local-dir ~/.ds4/models

# 3. Start ds4 server wrapper
~/.local/bin/ds4-server-up

# 4. Use with Claude Code
~/.local/bin/claude-ds4 "your prompt here"
```

## Configuration

### Claude Code Client Setup

Point your Claude Code client to the local server:

```bash
# Set API base URL
export ANTHROPIC_API_KEY="sk-ant-local-placeholder"  # Any string works
export ANTHROPIC_BASE_URL="http://localhost:8000"

# Or use the wrapper script
./claude-local "create a react component for a todo list"
```

### Server Configuration

Create `config/server.yaml`:

```yaml
# MLX server config
server:
  host: "0.0.0.0"  # Listen on all interfaces
  port: 8000
  
model:
  name: "mlx-community/Qwen3.5-MoE-A10B-4bit"
  max_tokens: 256000  # Qwen supports 256K context
  temperature: 0.7
  
performance:
  cache_prompt: true  # Enable prompt caching
  max_kv_size: 256000
```

### Launcher Scripts

The project includes `.command` launchers for double-click execution:

```bash
# macOS launchers (in repo root)
"Claude Local.command"          # Qwen 3.5 122B
"Gemma 4 Code.command"          # Gemma 4 31B
"DeepSeek V4 Flash.app"         # DeepSeek bundle
```

## Key Commands

### Starting the Server

```bash
# Basic start
bash scripts/start-mlx-server.sh

# With custom model
MLX_MODEL="mlx-community/Qwen3.5-MoE-A10B-4bit" \
  bash scripts/start-mlx-server.sh

# With custom port
PORT=8080 bash scripts/start-mlx-server.sh

# Monitor resource usage
python3 scripts/monitor.py
```

### Using the API

```python
import anthropic

# Point to local server
client = anthropic.Anthropic(
    api_key="sk-ant-local-placeholder",
    base_url="http://localhost:8000"
)

# Stream a response
with client.messages.stream(
    model="qwen-3.5-122b",  # Name doesn't matter, uses running model
    max_tokens=4096,
    messages=[
        {"role": "user", "content": "Write a Python function to parse JSON"}
    ]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

### Voice Mode Setup

```bash
# Install voice dependencies
pip install -r requirements-voice.txt

# Download voice models (whisper + kokoro TTS)
python3 scripts/download-voice-models.py

# Start voice-enabled server
bash scripts/start-voice-server.sh

# Talk to Claude Code (hands-free)
python3 scripts/voice-client.py
```

### Browser Agent

Access Claude Code from any device on your LAN:

```bash
# Start browser agent server
python3 scripts/browser-agent-server.py

# Open in any browser (phone, tablet, etc.)
# Navigate to: http://YOUR_MAC_IP:5000
```

## Real Code Examples

### Example 1: Basic Coding Task

```python
#!/usr/bin/env python3
"""Send a coding task to local Claude Code"""
import anthropic

client = anthropic.Anthropic(
    api_key="sk-ant-local",
    base_url="http://localhost:8000"
)

response = client.messages.create(
    model="local",
    max_tokens=8192,
    messages=[{
        "role": "user",
        "content": """Create a FastAPI endpoint that:
        1. Accepts POST with JSON {query: str}
        2. Validates query is 1-500 chars
        3. Returns {result: str, timestamp: int}
        Include error handling and type hints."""
    }]
)

print(response.content[0].text)
```

### Example 2: Document Analysis (Airgap Mode)

```python
#!/usr/bin/env python3
"""Analyze confidential document with offline AI"""
import anthropic
import sys

def analyze_nda(filepath):
    with open(filepath, 'r') as f:
        doc_text = f.read()
    
    client = anthropic.Anthropic(
        api_key="sk-ant-local",
        base_url="http://localhost:8000"
    )
    
    response = client.messages.create(
        model="llama-3.3-70b",
        max_tokens=16384,
        messages=[{
            "role": "user",
            "content": f"""Analyze this NDA for:
            1. Term length
            2. Geographic scope
            3. Definition of confidential info
            4. Carve-outs and exceptions
            5. Red flags
            
            Document:
            {doc_text}"""
        }]
    )
    
    return response.content[0].text

if __name__ == "__main__":
    result = analyze_nda(sys.argv[1])
    print(result)
```

### Example 3: Long-Context Processing (DeepSeek)

```python
#!/usr/bin/env python3
"""Use DeepSeek V4 Flash for 1M token context"""
import anthropic
import glob

def analyze_codebase(directory):
    # Collect all Python files
    files = glob.glob(f"{directory}/**/*.py", recursive=True)
    codebase = ""
    
    for filepath in files[:100]:  # Process up to 100 files
        with open(filepath, 'r') as f:
            codebase += f"\n\n# {filepath}\n{f.read()}"
    
    client = anthropic.Anthropic(
        api_key="sk-ant-local",
        base_url="http://localhost:8001"  # ds4 runs on 8001
    )
    
    response = client.messages.create(
        model="deepseek-v4-flash",
        max_tokens=32000,
        messages=[{
            "role": "user",
            "content": f"""Analyze this entire codebase:
            1. Architecture patterns used
            2. Security vulnerabilities
            3. Performance bottlenecks
            4. Suggested refactorings
            
            Codebase:
            {codebase}"""
        }]
    )
    
    return response.content[0].text

result = analyze_codebase("./my-project")
print(result)
```

### Example 4: Streaming with Tool Use

```python
#!/usr/bin/env python3
"""Stream responses with tool calling"""
import anthropic

tools = [{
    "name": "run_bash",
    "description": "Execute a bash command and return output",
    "input_schema": {
        "type": "object",
        "properties": {
            "command": {"type": "string", "description": "Bash command"}
        },
        "required": ["command"]
    }
}]

client = anthropic.Anthropic(
    api_key="sk-ant-local",
    base_url="http://localhost:8000"
)

with client.messages.stream(
    model="qwen-3.5",
    max_tokens=4096,
    tools=tools,
    messages=[{
        "role": "user",
        "content": "List all Python files in current directory and count lines"
    }]
) as stream:
    for event in stream:
        if event.type == "content_block_delta":
            if hasattr(event.delta, "text"):
                print(event.delta.text, end="", flush=True)
        elif event.type == "tool_use":
            print(f"\n[Tool call: {event.name}]")
```

## Common Patterns

### Pattern 1: Multi-Model Workflow

```python
#!/usr/bin/env python3
"""Use different models for different tasks"""
import anthropic

def get_client(model_port):
    return anthropic.Anthropic(
        api_key="sk-ant-local",
        base_url=f"http://localhost:{model_port}"
    )

# Fast model for quick responses (Gemma on 8000)
gemma = get_client(8000)
quick_response = gemma.messages.create(
    model="gemma-4",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Summarize: <long text>"}]
)

# Heavy model for complex reasoning (Qwen on 8001)
qwen = get_client(8001)
deep_analysis = qwen.messages.create(
    model="qwen-3.5",
    max_tokens=8192,
    messages=[{
        "role": "user",
        "content": f"Analyze this summary: {quick_response.content[0].text}"
    }]
)
```

### Pattern 2: Prompt Caching for Speed

```python
#!/usr/bin/env python3
"""Leverage KV cache for repeated contexts"""
import anthropic

client = anthropic.Anthropic(
    api_key="sk-ant-local",
    base_url="http://localhost:8000"
)

# Long system prompt (cached after first use)
system_prompt = """You are an expert Python developer.
Follow PEP 8, use type hints, write docstrings.
Prefer dataclasses over dicts, use pathlib over os.path.
[... 5000 more tokens of coding guidelines ...]"""

# First call: slow (prefills cache)
response1 = client.messages.create(
    model="qwen",
    max_tokens=2048,
    system=system_prompt,
    messages=[{"role": "user", "content": "Write a parser"}]
)

# Second call: fast (reuses cached prompt)
response2 = client.messages.create(
    model="qwen",
    max_tokens=2048,
    system=system_prompt,  # Same prompt = cache hit
    messages=[{"role": "user", "content": "Write a validator"}]
)
```

### Pattern 3: Airgap Verification

```python
#!/usr/bin/env python3
"""Verify no network calls during inference"""
import subprocess
import anthropic
import time

# Start monitoring network
monitor = subprocess.Popen(
    ["lsof", "-i", "-n", "-P"],
    stdout=subprocess.PIPE,
    stderr=subprocess.PIPE
)

client = anthropic.Anthropic(
    api_key="sk-ant-local",
    base_url="http://localhost:8000"
)

# Make AI request
response = client.messages.create(
    model="llama-3.3",
    max_tokens=4096,
    messages=[{"role": "user", "content": "Analyze sensitive data"}]
)

time.sleep(2)
monitor.terminate()
stdout, _ = monitor.communicate()

# Check for external connections
external = [
    line for line in stdout.decode().split('\n')
    if 'ESTABLISHED' in line and '127.0.0.1' not in line
]

if external:
    print(f"⚠️  External connections detected:\n{chr(10).join(external)}")
else:
    print("✅ Airgap verified: no external connections")
```

## Troubleshooting

### Issue: Model download fails

```bash
# Check HuggingFace credentials
huggingface-cli whoami

# Login if needed
huggingface-cli login

# Manual download
huggingface-cli download mlx-community/Qwen3.5-MoE-A10B-4bit \
  --local-dir ~/.cache/huggingface/hub/models--mlx-community--Qwen3.5-MoE-A10B-4bit
```

### Issue: Out of memory

```bash
# Check available RAM
vm_stat | perl -ne '/page size of (\d+)/ and $size=$1; /Pages\s+([^:]+)[^\d]+(\d+)/ and printf("%-16s % 16.2f Mi\n", "$1:", $2 * $size / 1048576);'

# Use smaller model
export MLX_MODEL="bartowski/gemma-4-code-it-31b-MLX-4bit"  # 18GB

# Or reduce max context
export MAX_KV_SIZE=32768  # Default is model-specific
```

### Issue: Slow inference

```bash
# Check active MoE params (Qwen)
# Should activate ~10B per token, not all 122B
python3 scripts/profile-inference.py

# Monitor GPU usage
sudo powermetrics --samplers gpu_power -i 1000

# Enable Metal debugging
export METAL_DEVICE_WRAPPER_TYPE=1
export METAL_DEBUG_ERROR_MODE=0
```

### Issue: API compatibility errors

```bash
# Check server logs
tail -f logs/mlx-server.log

# Test raw API
curl http://localhost:8000/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: sk-ant-local" \
  -d '{
    "model": "local",
    "max_tokens": 100,
    "messages": [{"role": "user", "content": "test"}]
  }'

# Verify Anthropic SDK version
pip show anthropic  # Should be >= 0.18.0
```

### Issue: DeepSeek V4 Flash won't start

```bash
# Check ds4 build
cd ~/.ds4
make clean && make

# Verify model file
ls -lh ~/.ds4/models/deepseek-v4-q2.gguf

# Check ds4 server logs
tail -f ~/.ds4/server.log

# Test ds4 directly (bypass wrapper)
~/.ds4/ds4 -m ~/.ds4/models/deepseek-v4-q2.gguf \
  -p "Test prompt" -c 200000
```

### Issue: Voice mode not working

```bash
# Check microphone permissions
# System Settings > Privacy & Security > Microphone

# Test whisper separately
python3 -c "
from faster_whisper import WhisperModel
model = WhisperModel('base')
segments, _ = model.transcribe('test.wav')
print(list(segments))
"

# Test kokoro TTS
python3 scripts/test-tts.py "Hello world"
```

## Performance Benchmarks

| Model | Tokens/sec | Claude Code Task | RAM | Context |
|-------|-----------|------------------|-----|---------|
| Gemma 4 31B | ~15 | 56s | 18 GB | 128K |
| Qwen 3.5 122B | **65** | **17.6s** | 75 GB | 256K |
| Llama 3.3 70B | ~12 | 137s | 75 GB | 128K |
| DeepSeek V4 Flash | ~32 | 45s | 81 GB | **1M** |

*Tested on M4 Max 128GB. "Claude Code Task" = generate HTML physics animation with live counters.*

## Environment Variables Reference

```bash
# Model selection
export MLX_MODEL="mlx-community/Qwen3.5-MoE-A10B-4bit"

# Server config
export PORT=8000
export HOST="0.0.0.0"

# Performance
export MAX_KV_SIZE=256000
export CACHE_PROMPT=true

# Anthropic SDK
export ANTHROPIC_API_KEY="sk-ant-local-placeholder"
export ANTHROPIC_BASE_URL="http://localhost:8000"

# DeepSeek (ds4 engine)
export DS4_MODEL_PATH="$HOME/.ds4/models/deepseek-v4-q2.gguf"
export DS4_PORT=8001
export DS4_CONTEXT=200000

# Voice mode
export WHISPER_MODEL="base"  # or "small", "medium"
export TTS_VOICE="af_bella"  # kokoro voice
export VOICE_SERVER_PORT=5001

# Browser agent
export BROWSER_AGENT_PORT=5000
export BROWSER_AGENT_HOST="0.0.0.0"
```

## Integration with Claude Code

### VS Code Extension

1. Install Claude Code extension
2. Configure settings.json:

```json
{
  "claude.apiKey": "sk-ant-local-placeholder",
  "claude.apiUrl": "http://localhost:8000",
  "claude.model": "qwen-3.5-122b"
}
```

### CLI Wrapper

```bash
# Install wrapper
ln -s "$(pwd)/claude-local" /usr/local/bin/claude-local

# Use like normal claude CLI
claude-local "refactor this function to use async/await"
```

### Cursor IDE

Add to Cursor settings:

```json
{
  "claude.overrideApiUrl": "http://localhost:8000",
  "claude.overrideApiKey": "sk-ant-local"
}
```

This skill covers the essential knowledge for using claude-code-local with MLX on Apple Silicon, from installation through advanced patterns and troubleshooting.
