---
name: claudecodeui-web-interface
description: CloudCLI (Claude Code UI) - web and mobile interface for managing Claude Code, Cursor CLI, Codex, and Gemini CLI sessions remotely
triggers:
  - "set up CloudCLI web interface"
  - "install Claude Code UI"
  - "manage Claude sessions remotely"
  - "run CloudCLI on mobile"
  - "configure CloudCLI plugins"
  - "enable Claude Code tools in CloudCLI"
  - "set up CloudCLI with Docker sandbox"
  - "access Claude Code from browser"
---

# CloudCLI (Claude Code UI) Web Interface

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

CloudCLI (aka Claude Code UI) is a web and mobile interface for managing Claude Code, Cursor CLI, Codex, and Gemini CLI sessions. It provides a responsive UI with chat interface, file explorer, Git integration, session management, and plugin system. Use it locally or remotely to access your AI coding sessions from any device.

## Installation

### Quick Start with npx (Recommended)

Try instantly without installation (requires Node.js v22+):

```bash
npx @cloudcli-ai/cloudcli
```

Then open `http://localhost:3001` in your browser.

### Global Installation

```bash
npm install -g @cloudcli-ai/cloudcli
cloudcli
```

### Docker Sandbox (Experimental)

For isolated agent sessions with hypervisor-level isolation:

```bash
# Requires Docker Sandboxes CLI: https://docs.docker.com/ai/sandboxes/get-started/
npx @cloudcli-ai/cloudcli@latest sandbox ~/my-project
```

## Configuration

### Basic Configuration

CloudCLI discovers existing Claude Code sessions automatically from `~/.claude`. No additional configuration needed for basic usage.

### Environment Variables

```bash
# Custom port (default: 3001)
PORT=8080 npx @cloudcli-ai/cloudcli

# Custom host (default: localhost)
HOST=0.0.0.0 npx @cloudcli-ai/cloudcli

# Custom Claude config directory
CLAUDE_CONFIG_DIR=~/custom/path npx @cloudcli-ai/cloudcli
```

### PM2 Process Manager

For production deployments:

```bash
# Install PM2
npm install -g pm2

# Install CloudCLI globally
npm install -g @cloudcli-ai/cloudcli

# Start with PM2
pm2 start cloudcli --name "cloudcli-server"

# Save PM2 config
pm2 save

# Enable auto-start on boot
pm2 startup
```

### Remote Server Access

To access from other devices on your network:

```bash
# Start with 0.0.0.0 to allow network access
HOST=0.0.0.0 PORT=3001 npx @cloudcli-ai/cloudcli
```

Access from any device: `http://[your-server-ip]:3001`

## Tool Security Configuration

**All Claude Code tools are disabled by default** for security. Enable only what you need:

1. Click the gear icon in the sidebar
2. Navigate to Tools Settings
3. Enable specific tools individually
4. Click "Apply Settings"

Common tools to enable:
- `read_file` - Read file contents
- `write_file` - Modify files
- `execute_command` - Run shell commands
- `search_files` - Search in project
- `list_directory` - Browse directories

Settings are saved to `~/.claude/config.json` and sync with native Claude Code.

## Plugin System

### Installing Plugins

1. Open Settings (gear icon)
2. Navigate to Plugins tab
3. Enter git repository URL
4. Click "Install Plugin"

Example plugins:

```
# Project Stats
https://github.com/cloudcli-ai/cloudcli-plugin-starter

# Web Terminal
https://github.com/cloudcli-ai/cloudcli-plugin-terminal

# CloudCLI Scheduler
https://github.com/grostim/cloudcli-cron
```

### Creating a Custom Plugin

Plugin structure:

```
my-plugin/
├── manifest.json          # Plugin configuration
├── client/
│   └── index.tsx         # React component (frontend)
└── server/
    └── index.js          # Node.js backend (optional)
```

**manifest.json:**

```json
{
  "name": "my-plugin",
  "displayName": "My Plugin",
  "version": "1.0.0",
  "description": "My custom plugin",
  "author": "Your Name",
  "entry": "client/index.tsx",
  "server": "server/index.js",
  "permissions": ["fs:read", "process:spawn"],
  "dependencies": {
    "react": "^18.0.0"
  }
}
```

**client/index.tsx:**

```typescript
import React, { useEffect, useState } from 'react';

export default function MyPlugin({ context, rpc }) {
  const [projectPath, setProjectPath] = useState('');

  useEffect(() => {
    // Access current project context
    setProjectPath(context.currentProject?.path || 'No project');
  }, [context.currentProject]);

  const handleAction = async () => {
    // Call backend RPC
    const result = await rpc.call('getStats', { path: projectPath });
    console.log(result);
  };

  return (
    <div className="p-4">
      <h2 className="text-xl font-bold mb-4">My Plugin</h2>
      <p>Project: {projectPath}</p>
      <button
        onClick={handleAction}
        className="mt-4 px-4 py-2 bg-blue-500 text-white rounded"
      >
        Get Stats
      </button>
    </div>
  );
}
```

**server/index.js:**

```javascript
export function activate(context) {
  // Register RPC methods
  context.rpc.registerMethod('getStats', async (params) => {
    const fs = await import('fs/promises');
    const files = await fs.readdir(params.path);
    return { fileCount: files.length, files };
  });

  return {
    dispose: () => {
      // Cleanup on deactivate
    }
  };
}
```

See the [plugin starter template](https://github.com/cloudcli-ai/cloudcli-plugin-starter) for complete examples.

## API Usage

CloudCLI exposes a REST API for programmatic access:

### Get Active Sessions

```bash
curl http://localhost:3001/api/sessions
```

Response:

```json
{
  "sessions": [
    {
      "id": "session-123",
      "name": "my-project",
      "path": "/home/user/projects/my-project",
      "created": "2025-01-01T10:00:00Z",
      "lastActive": "2025-01-01T12:30:00Z"
    }
  ]
}
```

### Send Message to Session

```bash
curl -X POST http://localhost:3001/api/sessions/session-123/message \
  -H "Content-Type: application/json" \
  -d '{"message": "Implement user authentication"}'
```

### Get Project Files

```bash
curl http://localhost:3001/api/projects/my-project/files
```

### Execute Command

```bash
curl -X POST http://localhost:3001/api/sessions/session-123/execute \
  -H "Content-Type: application/json" \
  -d '{"command": "npm test"}'
```

## Common Patterns

### Self-Hosted Development Setup

```bash
# Install globally
npm install -g @cloudcli-ai/cloudcli

# Start server
cloudcli

# In another terminal, start Claude Code
claude-code

# Access UI at http://localhost:3001
```

### Production Deployment

```bash
# Install dependencies
npm install -g pm2 @cloudcli-ai/cloudcli

# Start with PM2
pm2 start cloudcli --name cloudcli \
  -- --host 0.0.0.0 --port 3001

# Save and enable auto-start
pm2 save
pm2 startup

# Monitor logs
pm2 logs cloudcli
```

### Docker Sandbox Workflow

```bash
# Start isolated Claude Code session
npx @cloudcli-ai/cloudcli@latest sandbox ~/my-project

# Access at http://localhost:3001
# All file operations are sandboxed
```

### Multi-Agent Configuration

```typescript
// In your project's .cloudcli/config.json
{
  "agents": [
    {
      "type": "claude-code",
      "model": "claude-3-7-sonnet-20250219",
      "name": "Backend Agent"
    },
    {
      "type": "cursor-cli",
      "model": "gpt-4",
      "name": "Frontend Agent"
    },
    {
      "type": "gemini-cli",
      "model": "gemini-2.0-flash-exp",
      "name": "Testing Agent"
    }
  ]
}
```

### MCP Server Configuration

CloudCLI syncs with `~/.claude/config.json`:

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/workspace"],
      "env": {
        "NODE_ENV": "production"
      }
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

Changes made in CloudCLI UI are immediately reflected in native Claude Code.

## Troubleshooting

### Port Already in Use

```bash
# Find process using port 3001
lsof -i :3001

# Kill the process
kill -9 <PID>

# Or use different port
PORT=8080 npx @cloudcli-ai/cloudcli
```

### Cannot Access from Other Devices

```bash
# Ensure binding to 0.0.0.0
HOST=0.0.0.0 npx @cloudcli-ai/cloudcli

# Check firewall
sudo ufw allow 3001/tcp

# Verify network access
curl http://[your-ip]:3001
```

### Sessions Not Appearing

```bash
# Verify Claude config directory
ls -la ~/.claude/

# Check for sessions
ls -la ~/.claude/sessions/

# Specify custom config dir
CLAUDE_CONFIG_DIR=~/custom/path npx @cloudcli-ai/cloudcli
```

### Plugin Installation Fails

```bash
# Check plugin manifest
curl https://raw.githubusercontent.com/owner/plugin/main/manifest.json

# Verify permissions in manifest
# Ensure required dependencies are compatible

# Check CloudCLI logs
tail -f ~/.cloudcli/logs/plugin-manager.log
```

### Tool Permissions Not Saving

Tools settings are stored in `~/.claude/config.json`. Verify:

```bash
# Check config file
cat ~/.claude/config.json

# Ensure write permissions
ls -la ~/.claude/

# Reset to defaults
rm ~/.claude/config.json
# Restart CloudCLI to regenerate
```

### Docker Sandbox Issues

```bash
# Verify Docker Sandboxes CLI installed
sbx --version

# Check Docker daemon running
docker ps

# View sandbox logs
docker logs <container-id>

# Clean up stopped sandboxes
docker container prune
```

## Integration Examples

### VS Code Extension Integration

```typescript
// Access CloudCLI from VS Code extension
import axios from 'axios';

const CLOUDCLI_URL = 'http://localhost:3001';

async function sendToAgent(message: string) {
  const sessions = await axios.get(`${CLOUDCLI_URL}/api/sessions`);
  const activeSession = sessions.data.sessions[0];
  
  await axios.post(
    `${CLOUDCLI_URL}/api/sessions/${activeSession.id}/message`,
    { message }
  );
}
```

### CI/CD Integration

```yaml
# .github/workflows/ai-review.yml
name: AI Code Review
on: [pull_request]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Start CloudCLI
        run: |
          npx @cloudcli-ai/cloudcli &
          sleep 5
      
      - name: Request AI Review
        run: |
          curl -X POST http://localhost:3001/api/sessions/create \
            -H "Content-Type: application/json" \
            -d '{"project": "."}'
          
          curl -X POST http://localhost:3001/api/sessions/default/message \
            -H "Content-Type: application/json" \
            -d '{"message": "Review this pull request for security issues"}'
```

### n8n Workflow Integration

CloudCLI Cloud supports n8n webhooks for workflow automation. Self-hosted can use HTTP Request nodes:

```json
{
  "nodes": [
    {
      "name": "HTTP Request",
      "type": "n8n-nodes-base.httpRequest",
      "parameters": {
        "url": "http://localhost:3001/api/sessions/default/message",
        "method": "POST",
        "jsonParameters": true,
        "options": {},
        "bodyParametersJson": "={{ {\"message\": $json.prompt} }}"
      }
    }
  ]
}
```

## Advanced Configuration

### Custom Model Configuration

Add to `.cloudcli/config.json` in your project:

```json
{
  "models": {
    "custom-claude": {
      "provider": "anthropic",
      "model": "claude-3-7-sonnet-20250219",
      "temperature": 0.7,
      "maxTokens": 4096
    },
    "custom-gpt": {
      "provider": "openai",
      "model": "gpt-4-turbo-preview",
      "temperature": 0.5
    }
  }
}
```

### Workspace-Specific Settings

```json
{
  "workspace": {
    "name": "My Project",
    "description": "Full-stack application",
    "defaultAgent": "claude-code",
    "autoSave": true,
    "gitIntegration": true,
    "fileWatcher": {
      "enabled": true,
      "ignored": ["node_modules", ".git", "dist"]
    }
  }
}
```

### Network Security

For remote access with authentication:

```bash
# Use reverse proxy with auth (nginx example)
server {
    listen 443 ssl;
    server_name cloudcli.example.com;

    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    auth_basic "CloudCLI Access";
    auth_basic_user_file /etc/nginx/.htpasswd;

    location / {
        proxy_pass http://localhost:3001;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

## Resources

- **Documentation**: https://cloudcli.ai/docs
- **Plugin Starter**: https://github.com/cloudcli-ai/cloudcli-plugin-starter
- **Discord Community**: https://discord.gg/buxwujPNRE
- **GitHub Issues**: https://github.com/siteboon/claudecodeui/issues
- **Contributing Guide**: https://github.com/siteboon/claudecodeui/blob/main/CONTRIBUTING.md
