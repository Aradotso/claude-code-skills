---
name: free-claude-code-proxy
description: Run Claude Code CLI, VS Code, or JetBrains ACP through a local proxy that routes to NVIDIA NIM, Kimi, OpenRouter, DeepSeek, or local LLMs
triggers:
  - set up free claude code proxy
  - configure claude code with local models
  - route claude code to nvidia nim
  - use claude code with openrouter
  - run claude code through local proxy server
  - configure free-claude-code providers
  - switch claude code backend models
  - manage claude code proxy settings
---

# free-claude-code-proxy

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

Free Claude Code is a local proxy server that intercepts Anthropic Messages API calls from Claude Code and routes them to alternative providers like NVIDIA NIM, Kimi, Wafer, OpenRouter, DeepSeek, LM Studio, llama.cpp, Ollama, OpenCode Zen, or Z.ai. It maintains Claude Code's client-side protocol while letting you choose free, paid, or local models with full streaming, tool use, and reasoning block support.

## Installation

### Prerequisites

Install [uv](https://docs.astral.sh/uv/) and Python 3.14:

```bash
# macOS/Linux
curl -LsSf https://astral.sh/uv/install.sh | sh
uv self update
uv python install 3.14

# Windows PowerShell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
uv self update
uv python install 3.14
```

### Install Claude Code

```bash
npm install -g @anthropic-ai/claude-code
```

### Install Free Claude Code Proxy

```bash
uv tool install --force git+https://github.com/Alishahryar1/free-claude-code.git
```

Update to latest version with the same command.

## Starting the Proxy

```bash
fcc-server
```

Output shows:
```
Server URL: http://127.0.0.1:8082
Admin UI: http://127.0.0.1:8082/admin
```

The server runs on port `8082` by default (configurable via `PORT` env var).

## Admin UI Configuration

Open `http://127.0.0.1:8082/admin` in your browser to configure providers and models through a local web interface.

### Quick Setup: NVIDIA NIM

1. Get API key from https://build.nvidia.com/settings/api-keys
2. In Admin UI, paste into `NVIDIA_NIM_API_KEY`
3. Click **Validate** then **Apply**
4. Default `MODEL` is already set to `nvidia_nim/z-ai/glm4.7`

### Environment Variables

The Admin UI manages these settings (or set via `.env` file):

```bash
# Server config
PORT=8082
AUTH_TOKEN=freecc
LOG_LEVEL=INFO

# Provider API keys (set one or more)
NVIDIA_NIM_API_KEY=nvapi-xxx
KIMI_API_KEY=sk-xxx
WAFER_API_KEY=sk-xxx
OPENROUTER_API_KEY=sk-xxx
DEEPSEEK_API_KEY=sk-xxx
OPENCODE_API_KEY=sk-xxx
ZAI_API_KEY=sk-xxx

# Local provider URLs
LM_STUDIO_BASE_URL=http://localhost:1234
LLAMACPP_BASE_URL=http://localhost:8080
OLLAMA_BASE_URL=http://localhost:11434

# Model routing (prefix with provider/)
MODEL=nvidia_nim/z-ai/glm4.7
MODEL_OPUS=nvidia_nim/moonshotai/kimi-k2.5
MODEL_SONNET=open_router/deepseek/deepseek-r1-0528:free
MODEL_HAIKU=lmstudio/unsloth/GLM-4.7-Flash-GGUF
```

## Provider Configuration

### NVIDIA NIM

```bash
# In Admin UI or .env
NVIDIA_NIM_API_KEY=nvapi-your-key-here
MODEL=nvidia_nim/z-ai/glm4.7
```

Popular models:
- `nvidia_nim/z-ai/glm4.7`
- `nvidia_nim/z-ai/glm5`
- `nvidia_nim/moonshotai/kimi-k2.5`
- `nvidia_nim/minimaxai/minimax-m2.5`

### Kimi

```bash
KIMI_API_KEY=sk-your-key-here
MODEL=kimi/kimi-k2.5
```

### Wafer

```bash
WAFER_API_KEY=sk-your-key-here
MODEL=wafer/DeepSeek-V4-Pro
```

Popular models:
- `wafer/DeepSeek-V4-Pro`
- `wafer/MiniMax-M2.7`
- `wafer/Qwen3.5-397B-A17B`
- `wafer/GLM-5.1`

### OpenRouter

```bash
OPENROUTER_API_KEY=sk-your-key-here
MODEL=open_router/stepfun/step-3.5-flash:free
```

Browse models at https://openrouter.ai/models

### DeepSeek

```bash
DEEPSEEK_API_KEY=sk-your-key-here
MODEL=deepseek/deepseek-chat
```

Uses Anthropic-compatible endpoint, not OpenAI chat completions.

### LM Studio

```bash
LM_STUDIO_BASE_URL=http://localhost:1234
MODEL=lmstudio/your-model-name
```

Start LM Studio server and load a model with tool-use support.

### llama.cpp

```bash
LLAMACPP_BASE_URL=http://localhost:8080
MODEL=llamacpp/your-model
```

Start `llama-server` with sufficient context:
```bash
llama-server --model model.gguf --ctx-size 32768 --port 8080
```

### Ollama

```bash
OLLAMA_BASE_URL=http://localhost:11434
MODEL=ollama/llama3.1
```

Pull and serve model:
```bash
ollama pull llama3.1
ollama serve
```

### OpenCode Zen

```bash
OPENCODE_API_KEY=sk-your-key-here
MODEL=opencode/gpt-5.3-codex
```

Popular models:
- `opencode/gpt-5.3-codex`
- `opencode/claude-sonnet-4`
- `opencode/deepseek-v4-flash-free` (free)
- `opencode/big-pickle` (free)

### Z.ai

```bash
ZAI_API_KEY=sk-your-key-here
MODEL=zai/glm-5.1
```

Models:
- `zai/glm-5.1`
- `zai/glm-5-turbo`

## Connecting Claude Code Clients

### CLI (Recommended)

```bash
fcc-claude
```

This launcher:
- Reads current port and auth token from Admin UI config
- Sets `ANTHROPIC_BASE_URL` and `ANTHROPIC_AUTH_TOKEN`
- Launches the real `claude` command

Keep `fcc-server` running in another terminal.

### Manual CLI Setup

```bash
export ANTHROPIC_BASE_URL=http://localhost:8082
export ANTHROPIC_AUTH_TOKEN=freecc
export CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY=1
claude
```

### VS Code Extension

Open Settings → search `claude-code.environmentVariables` → Edit in `settings.json`:

```json
{
  "claudeCode.environmentVariables": [
    { "name": "ANTHROPIC_BASE_URL", "value": "http://localhost:8082" },
    { "name": "ANTHROPIC_AUTH_TOKEN", "value": "freecc" },
    { "name": "CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY", "value": "1" }
  ]
}
```

Reload VS Code window.

### JetBrains ACP

Edit installed ACP config:
- Windows: `C:\Users\%USERNAME%\AppData\Roaming\JetBrains\acp-agents\installed.json`
- Linux/macOS: `~/.jetbrains/acp.json`

Add environment to `acp.registry.claude-acp`:

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "http://localhost:8082",
    "ANTHROPIC_AUTH_TOKEN": "freecc",
    "CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY": "1"
  }
}
```

Restart IDE.

## Per-Tier Model Routing

Route different Claude Code model tiers to different providers:

```bash
# Fallback
MODEL=zai/glm-5.1

# Override specific tiers
MODEL_OPUS=nvidia_nim/moonshotai/kimi-k2.5
MODEL_SONNET=open_router/deepseek/deepseek-r1-0528:free
MODEL_HAIKU=lmstudio/unsloth/GLM-4.7-Flash-GGUF
```

Leave tier blank to inherit from `MODEL`.

## Model Picker Support

With `CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY=1`, Claude Code's `/model` picker shows models from the proxy's `/v1/models` endpoint.

The proxy returns configured models based on:
- Single provider: lists that provider's models
- Multiple providers: shows models from all configured providers
- Blank `MODEL`: returns empty list

Example: If you configure `nvidia_nim` and `open_router`, the picker shows models from both.

## Code Examples

### Python: Direct Proxy Request

```python
import os
import requests

url = "http://localhost:8082/v1/messages"
headers = {
    "x-api-key": os.getenv("AUTH_TOKEN", "freecc"),
    "anthropic-version": "2023-06-01",
    "content-type": "application/json"
}
payload = {
    "model": "claude-3-5-sonnet-20241022",
    "max_tokens": 1024,
    "messages": [
        {"role": "user", "content": "Write a Python hello world"}
    ]
}

response = requests.post(url, json=payload, headers=headers, stream=True)
for line in response.iter_lines():
    if line:
        print(line.decode())
```

### Python: Check Available Models

```python
import requests

response = requests.get(
    "http://localhost:8082/v1/models",
    headers={"x-api-key": "freecc"}
)
models = response.json()
for model in models.get("data", []):
    print(f"{model['id']}: {model['name']}")
```

### Shell: Test Proxy Streaming

```bash
curl -N http://localhost:8082/v1/messages \
  -H "x-api-key: freecc" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{
    "model": "claude-3-5-sonnet-20241022",
    "max_tokens": 1024,
    "messages": [{"role": "user", "content": "Hello"}],
    "stream": true
  }'
```

### JavaScript: Use Proxy in Custom Client

```javascript
const response = await fetch('http://localhost:8082/v1/messages', {
  method: 'POST',
  headers: {
    'x-api-key': process.env.AUTH_TOKEN || 'freecc',
    'anthropic-version': '2023-06-01',
    'content-type': 'application/json'
  },
  body: JSON.stringify({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 1024,
    messages: [{role: 'user', content: 'Explain async/await'}]
  })
});

const data = await response.json();
console.log(data.content[0].text);
```

## Common Workflows

### Switch Between Providers

1. Open Admin UI: `http://localhost:8082/admin`
2. Update API key for new provider
3. Change `MODEL` to new provider prefix (e.g., `kimi/kimi-k2.5`)
4. Click **Validate** then **Apply**
5. Proxy restarts automatically

No need to restart `fcc-claude` — it reads config on each launch.

### Use Local Model with Tool Support

```bash
# Start LM Studio with a tool-capable model
# In Admin UI:
LM_STUDIO_BASE_URL=http://localhost:1234
MODEL=lmstudio/deepseek-coder-6.7b-instruct

# Or for llama.cpp:
llama-server --model deepseek-coder-6.7b-instruct.Q4_K_M.gguf \
  --ctx-size 16384 --port 8080

LLAMACPP_BASE_URL=http://localhost:8080
MODEL=llamacpp/deepseek-coder-6.7b-instruct
```

### Mix Free and Paid Models

```bash
# Free tier for Haiku
MODEL_HAIKU=open_router/deepseek/deepseek-r1-0528:free

# Paid NVIDIA NIM for Opus
MODEL_OPUS=nvidia_nim/moonshotai/kimi-k2.5

# Fallback to free OpenCode
MODEL=opencode/big-pickle
```

### Debug Proxy Requests

Set log level in Admin UI or `.env`:

```bash
LOG_LEVEL=DEBUG
```

Restart proxy to see full request/response logs.

## Troubleshooting

### Proxy returns 401 Unauthorized

- Check `AUTH_TOKEN` matches in proxy and client
- Verify `ANTHROPIC_AUTH_TOKEN` env var in Claude Code client
- Admin UI shows current `AUTH_TOKEN` value

### Claude Code says "Invalid API key"

- Ensure provider API key is set correctly in Admin UI
- Click **Validate** to test the key
- Check provider dashboard for key status

### Local model returns HTTP 400

For llama.cpp/LM Studio:
- Increase `--ctx-size` for llama.cpp
- Verify model supports tool use
- Check server logs for context length errors
- Try a smaller conversation history

### Model picker shows no models

- Verify `CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY=1` is set
- Check that at least one provider is configured with valid credentials
- Visit `http://localhost:8082/v1/models` to see raw response

### VS Code extension doesn't use proxy

- Reload VS Code window after changing `settings.json`
- Check Output panel → Claude Code for environment variable logs
- Verify `claudeCode.environmentVariables` syntax is correct JSON array

### Admin UI changes don't apply

- Click **Apply** after **Validate**
- Proxy auto-restarts on runtime setting changes
- Check terminal for restart confirmation message

### Port 8082 already in use

Change port in Admin UI or `.env`:
```bash
PORT=8083
```

Restart proxy and update client `ANTHROPIC_BASE_URL` accordingly.

### llama.cpp/Ollama not found

- Verify server is running: `curl http://localhost:8080/health` (llama.cpp) or `curl http://localhost:11434` (Ollama)
- Check `*_BASE_URL` doesn't include `/v1` suffix for Ollama
- Ensure firewall allows localhost connections

## Advanced Features

### Discord Bot Integration

```bash
# In Admin UI or .env
MESSAGING_PLATFORM=discord
DISCORD_BOT_TOKEN=your-bot-token
ALLOWED_DISCORD_CHANNELS=123456789,987654321
CLAUDE_WORKSPACE=./agent_workspace
```

Start bot:
```bash
fcc-bot
```

### Voice Transcription

```bash
# Local Whisper
VOICE_PROVIDER=whisper
WHISPER_MODEL=base

# Or NVIDIA NIM
VOICE_PROVIDER=nvidia_nim
NVIDIA_NIM_API_KEY=nvapi-xxx
```

### Health Check Endpoint

```bash
curl http://localhost:8082/health
```

Returns proxy status and configured provider.

### Request Optimization

The proxy automatically:
- Strips unsupported parameters for local models
- Converts thinking blocks for providers without native support
- Handles streaming SSE format differences
- Retries with fallback models on provider errors

## Development

Clone and install for development:

```bash
git clone https://github.com/Alishahryar1/free-claude-code.git
cd free-claude-code
uv sync
uv run pytest
```

Run from source:
```bash
uv run fcc-server
```

The project uses:
- `uv` for dependency management
- `pytest` for testing
- `ruff` for linting/formatting
- `loguru` for logging
- `ty` for type checking
