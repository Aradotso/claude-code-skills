---
name: claude-code-agent-monitor
description: Real-time monitoring dashboard for Claude Code agent activity with SQLite, Express, React, WebSocket, and native desktop apps
triggers:
  - set up monitoring for claude code agents
  - track agent sessions and tool usage
  - create a dashboard for claude code activity
  - monitor agent performance in real time
  - configure claude code session tracking
  - visualize agent tool invocations
  - integrate claude code monitoring hooks
  - deploy agent monitoring dashboard
---

# claude-code-agent-monitor

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## Overview

**Claude Code Agent Monitor** is a full-stack monitoring platform that tracks Claude Code agent sessions, tool usage, subagent orchestration, and performance metrics in real-time. It uses a hook-based architecture where Claude Code fires events (tool use, session stop) that are captured by a Node.js handler, stored in SQLite, and broadcast via WebSocket to a React dashboard.

**Stack**: Node.js + Express + better-sqlite3 (WAL mode) + WebSocket (RFC 6455) + React 18 + TypeScript + Vite + TailwindCSS + Electron (desktop apps).

**Key capabilities**:
- Real-time session tracking with live agent cards
- Tool usage analytics (bar charts, heatmaps, D3.js visualizations)
- Subagent orchestration flow diagrams (Mermaid)
- Kanban board for agent status
- Browser notifications (Web Push API + VAPID)
- RESTful API + Swagger/OpenAPI docs
- MCP server for dashboard introspection
- VS Code extension + macOS/Windows native apps
- i18n support (English, Chinese, Vietnamese, Korean, Spanish)
- Prometheus metrics export + Grafana dashboards
- Kubernetes, Docker, Terraform deployment recipes

## Installation

### Prerequisites

- **Node.js** ≥ 20.x
- **Python** ≥ 3.6 (for hook handler script)
- **Claude Code** installed and configured

### Quick Start

```bash
# Clone the repository
git clone https://github.com/hoangsonww/Claude-Code-Agent-Monitor.git
cd Claude-Code-Agent-Monitor

# Install dependencies
npm install

# Build the React frontend
npm run build

# Start the server (production mode)
npm start
# Server runs on http://localhost:3000
```

For development with hot reload:

```bash
# Terminal 1: Start backend server
npm run dev

# Terminal 2: Start Vite dev server for React frontend
npm run dev:client
# Frontend runs on http://localhost:5173
```

### Hook Handler Setup

The Python hook handler captures Claude Code events and forwards them to the dashboard server.

1. **Copy the hook script** to your Claude Code hooks directory:

```bash
# macOS/Linux
cp hook-handler/hook_handler.py ~/.claude-code/hooks/

# Windows
copy hook-handler\hook_handler.py %USERPROFILE%\.claude-code\hooks\
```

2. **Configure the hook endpoint** in `hook_handler.py`:

```python
# hook_handler.py
DASHBOARD_URL = os.getenv("DASHBOARD_URL", "http://localhost:3000")
```

3. **Enable hooks in Claude Code config** (`~/.claude-code/config.json`):

```json
{
  "hooks": {
    "enabled": true,
    "onToolUse": "~/.claude-code/hooks/hook_handler.py",
    "onSessionStop": "~/.claude-code/hooks/hook_handler.py"
  }
}
```

4. **Test the hook**:

```bash
python3 hook_handler.py --test
# Should POST to http://localhost:3000/api/events
```

## Configuration

### Environment Variables

Create a `.env` file in the project root:

```bash
# Server
PORT=3000
HOST=0.0.0.0
NODE_ENV=production

# Database
DB_PATH=./data/agent-monitor.db
# WAL mode is enabled by default for better concurrency

# WebSocket
WS_PORT=3001
WS_PATH=/ws

# VAPID keys for push notifications (generate with npm run generate-vapid)
VAPID_PUBLIC_KEY=your_public_key_here
VAPID_PRIVATE_KEY=your_private_key_here
VAPID_SUBJECT=mailto:your-email@example.com

# CORS (comma-separated origins)
CORS_ORIGINS=http://localhost:5173,http://localhost:3000

# Retention policy (days)
DATA_RETENTION_DAYS=30

# Prometheus metrics
METRICS_ENABLED=true
METRICS_PORT=9090
```

### Server Configuration

Edit `server/config.js` for advanced settings:

```javascript
module.exports = {
  server: {
    port: process.env.PORT || 3000,
    host: process.env.HOST || '0.0.0.0',
  },
  database: {
    path: process.env.DB_PATH || './data/agent-monitor.db',
    walMode: true, // Write-Ahead Logging for concurrent reads/writes
    busyTimeout: 5000,
  },
  websocket: {
    port: process.env.WS_PORT || 3001,
    path: process.env.WS_PATH || '/ws',
    heartbeatInterval: 30000, // ping clients every 30s
  },
  retention: {
    enabled: true,
    intervalHours: 24,
    retentionDays: parseInt(process.env.DATA_RETENTION_DAYS, 10) || 30,
  },
  cors: {
    origins: process.env.CORS_ORIGINS?.split(',') || ['http://localhost:5173'],
  },
};
```

## API Reference

### REST Endpoints

The server exposes a RESTful API documented via Swagger at `http://localhost:3000/api-docs`.

#### POST `/api/events`

**Receive hook events** from Claude Code.

```bash
curl -X POST http://localhost:3000/api/events \
  -H "Content-Type: application/json" \
  -d '{
    "type": "tool_use",
    "sessionId": "sess_abc123",
    "agentId": "agent_main",
    "toolName": "file_read",
    "timestamp": "2026-08-02T22:01:04Z",
    "metadata": {
      "path": "/src/app.ts",
      "duration_ms": 150
    }
  }'
```

#### GET `/api/sessions`

**List all sessions** with pagination.

```bash
curl http://localhost:3000/api/sessions?page=1&limit=20
```

Response:

```json
{
  "sessions": [
    {
      "id": "sess_abc123",
      "agentId": "agent_main",
      "startTime": "2026-08-02T20:00:00Z",
      "endTime": null,
      "status": "active",
      "toolCount": 47,
      "subagentCount": 3
    }
  ],
  "total": 150,
  "page": 1,
  "limit": 20
}
```

#### GET `/api/sessions/:id`

**Get session details** including tool usage and subagents.

```bash
curl http://localhost:3000/api/sessions/sess_abc123
```

#### GET `/api/agents`

**List all agents** with activity summary.

```bash
curl http://localhost:3000/api/agents
```

#### GET `/api/tools/stats`

**Tool usage statistics** (counts, durations, success rates).

```bash
curl http://localhost:3000/api/tools/stats?period=7d
```

#### GET `/api/health`

**Health check** with database status, WebSocket connection count, and system metrics.

```bash
curl http://localhost:3000/api/health
```

Response:

```json
{
  "status": "healthy",
  "timestamp": "2026-08-02T22:01:04Z",
  "database": {
    "connected": true,
    "walMode": true,
    "rowCount": 15234
  },
  "websocket": {
    "clients": 3
  },
  "uptime": 864213
}
```

#### DELETE `/api/sessions/:id`

**Delete a session** and all associated events.

```bash
curl -X DELETE http://localhost:3000/api/sessions/sess_abc123
```

### WebSocket Events

Connect to `ws://localhost:3001/ws` to receive real-time updates.

**Client → Server**:

```json
{
  "type": "subscribe",
  "channels": ["sessions", "tools", "agents"]
}
```

**Server → Client**:

```json
{
  "type": "session_start",
  "data": {
    "sessionId": "sess_abc123",
    "agentId": "agent_main",
    "timestamp": "2026-08-02T22:01:04Z"
  }
}
```

Event types: `session_start`, `session_stop`, `tool_use`, `subagent_spawn`, `agent_status_change`.

## Hook Events

The hook handler sends events in the following format:

### Tool Use Event

```json
{
  "type": "tool_use",
  "sessionId": "sess_abc123",
  "agentId": "agent_main",
  "toolName": "file_edit",
  "timestamp": "2026-08-02T22:01:04Z",
  "metadata": {
    "path": "/src/components/Dashboard.tsx",
    "linesChanged": 15,
    "duration_ms": 230,
    "success": true
  }
}
```

### Session Stop Event

```json
{
  "type": "session_stop",
  "sessionId": "sess_abc123",
  "agentId": "agent_main",
  "timestamp": "2026-08-02T23:00:00Z",
  "metadata": {
    "duration_seconds": 3600,
    "toolsUsed": 47,
    "subagentsSpawned": 3,
    "exitReason": "user_stop"
  }
}
```

### Subagent Spawn Event

```json
{
  "type": "subagent_spawn",
  "sessionId": "sess_abc123",
  "parentAgentId": "agent_main",
  "subagentId": "agent_test_runner",
  "timestamp": "2026-08-02T22:15:00Z",
  "metadata": {
    "purpose": "run_vitest_suite",
    "context": {
      "testFile": "src/components/Dashboard.test.tsx"
    }
  }
}
```

## Code Examples

### React Component: Live Agent Card

```typescript
// src/components/AgentCard.tsx
import React, { useEffect, useState } from 'react';
import { Activity, Clock, Zap } from 'lucide-react';

interface Agent {
  id: string;
  status: 'active' | 'idle' | 'stopped';
  sessionId: string;
  toolCount: number;
  lastActivity: string;
}

export const AgentCard: React.FC<{ agentId: string }> = ({ agentId }) => {
  const [agent, setAgent] = useState<Agent | null>(null);

  useEffect(() => {
    // Fetch initial agent data
    fetch(`/api/agents/${agentId}`)
      .then(res => res.json())
      .then(setAgent);

    // Subscribe to WebSocket updates
    const ws = new WebSocket('ws://localhost:3001/ws');
    ws.onopen = () => {
      ws.send(JSON.stringify({
        type: 'subscribe',
        channels: ['agents'],
        filter: { agentId }
      }));
    };

    ws.onmessage = (event) => {
      const update = JSON.parse(event.data);
      if (update.type === 'agent_status_change' && update.data.agentId === agentId) {
        setAgent(prev => prev ? { ...prev, ...update.data } : null);
      }
    };

    return () => ws.close();
  }, [agentId]);

  if (!agent) return <div className="animate-pulse">Loading...</div>;

  const statusColors = {
    active: 'bg-green-500',
    idle: 'bg-yellow-500',
    stopped: 'bg-gray-500'
  };

  return (
    <div className="bg-gray-800 rounded-lg p-4 border border-gray-700">
      <div className="flex items-center justify-between mb-3">
        <h3 className="text-lg font-semibold text-white">{agent.id}</h3>
        <span className={`h-3 w-3 rounded-full ${statusColors[agent.status]}`} />
      </div>
      <div className="space-y-2 text-sm text-gray-400">
        <div className="flex items-center gap-2">
          <Activity size={16} />
          <span>Session: {agent.sessionId}</span>
        </div>
        <div className="flex items-center gap-2">
          <Zap size={16} />
          <span>{agent.toolCount} tools used</span>
        </div>
        <div className="flex items-center gap-2">
          <Clock size={16} />
          <span>Last: {new Date(agent.lastActivity).toLocaleTimeString()}</span>
        </div>
      </div>
    </div>
  );
};
```

### Express: Hook Event Handler

```javascript
// server/routes/events.js
const express = require('express');
const router = express.Router();
const db = require('../db');
const wsBroadcast = require('../websocket').broadcast;

router.post('/', (req, res) => {
  const { type, sessionId, agentId, toolName, timestamp, metadata } = req.body;

  // Validate required fields
  if (!type || !sessionId || !agentId || !timestamp) {
    return res.status(400).json({ error: 'Missing required fields' });
  }

  try {
    // Insert event into SQLite
    const insert = db.prepare(`
      INSERT INTO events (type, session_id, agent_id, tool_name, timestamp, metadata)
      VALUES (?, ?, ?, ?, ?, ?)
    `);
    
    const result = insert.run(
      type,
      sessionId,
      agentId,
      toolName || null,
      timestamp,
      JSON.stringify(metadata || {})
    );

    // Broadcast to WebSocket clients
    wsBroadcast({
      type: `event_${type}`,
      data: {
        id: result.lastInsertRowid,
        ...req.body
      }
    });

    res.status(201).json({ 
      success: true, 
      eventId: result.lastInsertRowid 
    });
  } catch (error) {
    console.error('Error storing event:', error);
    res.status(500).json({ error: 'Failed to store event' });
  }
});

module.exports = router;
```

### SQLite: Session Query

```javascript
// server/db/queries.js
const db = require('./index');

function getActiveSessions() {
  const query = db.prepare(`
    SELECT 
      s.id,
      s.agent_id,
      s.start_time,
      COUNT(DISTINCT e.tool_name) as unique_tools,
      COUNT(*) as event_count,
      MAX(e.timestamp) as last_activity
    FROM sessions s
    LEFT JOIN events e ON s.id = e.session_id
    WHERE s.end_time IS NULL
    GROUP BY s.id
    ORDER BY s.start_time DESC
  `);

  return query.all();
}

function getToolStats(sessionId) {
  const query = db.prepare(`
    SELECT 
      tool_name,
      COUNT(*) as count,
      AVG(json_extract(metadata, '$.duration_ms')) as avg_duration,
      SUM(CASE WHEN json_extract(metadata, '$.success') = 1 THEN 1 ELSE 0 END) as success_count
    FROM events
    WHERE session_id = ? AND type = 'tool_use'
    GROUP BY tool_name
    ORDER BY count DESC
  `);

  return query.all(sessionId);
}

module.exports = { getActiveSessions, getToolStats };
```

### Python: Hook Handler Script

```python
#!/usr/bin/env python3
# hook_handler.py
import sys
import json
import os
from datetime import datetime
import requests

DASHBOARD_URL = os.getenv("DASHBOARD_URL", "http://localhost:3000")

def send_event(event_type, session_id, agent_id, tool_name=None, metadata=None):
    payload = {
        "type": event_type,
        "sessionId": session_id,
        "agentId": agent_id,
        "toolName": tool_name,
        "timestamp": datetime.utcnow().isoformat() + "Z",
        "metadata": metadata or {}
    }
    
    try:
        response = requests.post(
            f"{DASHBOARD_URL}/api/events",
            json=payload,
            timeout=5,
            headers={"Content-Type": "application/json"}
        )
        response.raise_for_status()
        print(f"✓ Event sent: {event_type}", file=sys.stderr)
    except requests.exceptions.RequestException as e:
        print(f"✗ Failed to send event: {e}", file=sys.stderr)
        sys.exit(1)

def main():
    # Read hook data from stdin (Claude Code hook protocol)
    try:
        hook_data = json.loads(sys.stdin.read())
    except json.JSONDecodeError:
        print("✗ Invalid JSON input", file=sys.stderr)
        sys.exit(1)
    
    event_type = hook_data.get("event", "tool_use")
    session_id = hook_data.get("sessionId")
    agent_id = hook_data.get("agentId", "agent_main")
    tool_name = hook_data.get("toolName")
    metadata = hook_data.get("metadata", {})
    
    if not session_id:
        print("✗ Missing sessionId", file=sys.stderr)
        sys.exit(1)
    
    send_event(event_type, session_id, agent_id, tool_name, metadata)

if __name__ == "__main__":
    if "--test" in sys.argv:
        # Test mode: send a dummy event
        send_event(
            "tool_use",
            "sess_test_" + datetime.now().strftime("%Y%m%d%H%M%S"),
            "agent_test",
            "file_read",
            {"path": "/test/file.txt", "duration_ms": 42}
        )
    else:
        main()
```

## Common Patterns

### 1. Real-Time Dashboard Updates

Use React hooks + WebSocket to keep UI in sync:

```typescript
// src/hooks/useRealtimeData.ts
import { useEffect, useState } from 'react';

export function useRealtimeData<T>(endpoint: string, wsChannel: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // Initial fetch
    fetch(endpoint)
      .then(res => res.json())
      .then(initial => {
        setData(initial);
        setLoading(false);
      });

    // WebSocket updates
    const ws = new WebSocket('ws://localhost:3001/ws');
    ws.onopen = () => {
      ws.send(JSON.stringify({ type: 'subscribe', channels: [wsChannel] }));
    };

    ws.onmessage = (event) => {
      const update = JSON.parse(event.data);
      if (update.type.startsWith(wsChannel)) {
        setData(prev => ({ ...prev, ...update.data }));
      }
    };

    return () => ws.close();
  }, [endpoint, wsChannel]);

  return { data, loading };
}

// Usage in component
const { data: sessions, loading } = useRealtimeData<Session[]>(
  '/api/sessions',
  'sessions'
);
```

### 2. Tool Usage Analytics

Query and visualize tool patterns:

```javascript
// server/routes/analytics.js
router.get('/tools/heatmap', (req, res) => {
  const days = parseInt(req.query.days, 10) || 7;
  
  const query = db.prepare(`
    SELECT 
      tool_name,
      strftime('%Y-%m-%d', timestamp) as date,
      COUNT(*) as count
    FROM events
    WHERE type = 'tool_use' 
      AND timestamp >= datetime('now', '-${days} days')
    GROUP BY tool_name, date
    ORDER BY date ASC, count DESC
  `);

  const rows = query.all();
  
  // Transform to heatmap format
  const heatmap = rows.reduce((acc, row) => {
    if (!acc[row.tool_name]) acc[row.tool_name] = [];
    acc[row.tool_name].push({ date: row.date, count: row.count });
    return acc;
  }, {});

  res.json({ heatmap, days });
});
```

### 3. Subagent Orchestration Flow

Generate Mermaid diagrams for subagent spawning:

```typescript
// src/utils/mermaidFlow.ts
export function generateSubagentFlow(session: Session): string {
  const events = session.events.filter(e => e.type === 'subagent_spawn');
  
  let mermaid = 'graph TD\n';
  mermaid += `  A[${session.agentId}]\n`;
  
  events.forEach((event, idx) => {
    const nodeId = String.fromCharCode(66 + idx); // B, C, D...
    mermaid += `  A -->|spawn| ${nodeId}[${event.subagentId}]\n`;
    mermaid += `  ${nodeId} -.->|${event.metadata.purpose}| A\n`;
  });
  
  return mermaid;
}

// Render in React component
import mermaid from 'mermaid';

useEffect(() => {
  mermaid.initialize({ startOnLoad: true, theme: 'dark' });
  mermaid.contentLoaded();
}, []);

return <div className="mermaid">{generateSubagentFlow(session)}</div>;
```

### 4. Notification System

Subscribe to browser push notifications:

```typescript
// src/utils/notifications.ts
export async function subscribeToPush() {
  if (!('serviceWorker' in navigator) || !('PushManager' in window)) {
    throw new Error('Push notifications not supported');
  }

  const registration = await navigator.serviceWorker.register('/sw.js');
  
  const subscription = await registration.pushManager.subscribe({
    userVisibleOnly: true,
    applicationServerKey: urlBase64ToUint8Array(VAPID_PUBLIC_KEY)
  });

  // Send subscription to server
  await fetch('/api/notifications/subscribe', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(subscription)
  });

  return subscription;
}

function urlBase64ToUint8Array(base64String: string) {
  const padding = '='.repeat((4 - base64String.length % 4) % 4);
  const base64 = (base64String + padding).replace(/-/g, '+').replace(/_/g, '/');
  const rawData = window.atob(base64);
  return Uint8Array.from([...rawData].map(char => char.charCodeAt(0)));
}
```

### 5. Data Retention Policy

Automated cleanup of old sessions:

```javascript
// server/jobs/retention.js
const cron = require('node-cron');
const db = require('../db');
const config = require('../config');

function setupRetentionJob() {
  if (!config.retention.enabled) return;

  // Run daily at 2 AM
  cron.schedule('0 2 * * *', () => {
    console.log('Running retention policy...');
    
    const cutoffDate = new Date();
    cutoffDate.setDate(cutoffDate.getDate() - config.retention.retentionDays);
    
    const deleteStmt = db.prepare(`
      DELETE FROM events 
      WHERE timestamp < ?
    `);
    
    const result = deleteStmt.run(cutoffDate.toISOString());
    console.log(`Deleted ${result.changes} old events`);
    
    // Vacuum to reclaim space
    db.exec('VACUUM');
  });
}

module.exports = { setupRetentionJob };
```

## Desktop Apps

### macOS App

Build and package the Electron app for macOS:

```bash
# Development
npm run dev:electron

# Production build (creates DMG installer)
npm run build:mac
# Output: dist/Agent-Monitor-1.0.0-arm64.dmg
#         dist/Agent-Monitor-1.0.0-x64.dmg

# Sign for distribution (requires Apple Developer ID)
export APPLE_ID="your-apple-id@example.com"
export APPLE_ID_PASSWORD="app-specific-password"
export APPLE_TEAM_ID="YOUR_TEAM_ID"

npm run build:mac
```

The macOS app includes:
- Native menu bar integration
- Launch at login (SMAppService)
- Dock badge notifications
- Touch Bar support
- Code signing + notarization ready

### Windows App

Build and package for Windows:

```bash
# Development
npm run dev:electron

# Production build (creates NSIS installer + portable ZIP)
npm run build:win
# Output: dist/Agent-Monitor-Setup-1.0.0.exe
#         dist/Agent-Monitor-1.0.0-win-x64.zip

# Sign for distribution (requires code signing certificate)
export WIN_CSC_LINK="path/to/certificate.pfx"
export WIN_CSC_KEY_PASSWORD="certificate-password"

npm run build:win
```

Features:
- System tray integration
- Auto-update via electron-updater
- Windows installer (NSIS) + portable version
- Taskbar progress indicators

### Electron Configuration

```javascript
// electron/main.js
const { app, BrowserWindow, Tray, Menu } = require('electron');
const path = require('path');
const { spawn } = require('child_process');

let mainWindow;
let tray;
let serverProcess;

function createWindow() {
  mainWindow = new BrowserWindow({
    width: 1400,
    height: 900,
    webPreferences: {
      nodeIntegration: false,
      contextIsolation: true,
      preload: path.join(__dirname, 'preload.js')
    },
    icon: path.join(__dirname, 'icon.png')
  });

  // Start backend server
  serverProcess = spawn('node', ['server/index.js'], {
    cwd: path.join(__dirname, '..'),
    env: { ...process.env, ELECTRON_MODE: '1' }
  });

  // Load dashboard
  if (process.env.NODE_ENV === 'development') {
    mainWindow.loadURL('http://localhost:5173');
  } else {
    mainWindow.loadFile(path.join(__dirname, '../dist/index.html'));
  }

  // System tray
  tray = new Tray(path.join(__dirname, 'icon-tray.png'));
  const contextMenu = Menu.buildFromTemplate([
    { label: 'Show Dashboard', click: () => mainWindow.show() },
    { label: 'Quit', click: () => app.quit() }
  ]);
  tray.setContextMenu(contextMenu);
}

app.whenReady().then(createWindow);

app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') {
    if (serverProcess) serverProcess.kill();
    app.quit();
  }
});

app.on('before-quit', () => {
  if (serverProcess) serverProcess.kill();
});
```

## Deployment

### Docker

```bash
# Build image
docker build -t claude-agent-monitor .

# Run container
docker run -d \
  --name agent-monitor \
  -p 3000:3000 \
  -p 3001:3001 \
  -v $(pwd)/data:/app/data \
  -e DASHBOARD_URL=http://localhost:3000 \
  claude-agent-monitor

# Docker Compose
docker-compose up -d
```

### Kubernetes

```bash
# Deploy with Helm
helm install agent-monitor ./deploy/helm \
  --set image.tag=latest \
  --set service.type=LoadBalancer \
  --set persistence.enabled=true \
  --set persistence.size=10Gi

# Or use Kustomize
kubectl apply -k deploy/k8s/overlays/production
```

### Prometheus + Grafana

```bash
# Enable Prometheus metrics
export METRICS_ENABLED=true
export METRICS_PORT=9090

npm start

# Metrics exposed at http://localhost:9090/metrics

# Import Grafana dashboard
# File: deploy/grafana/dashboard.json
```

Key metrics:
- `agent_sessions_active` — Current active sessions
- `agent_events_total` — Total events received (by type)
- `agent_tool_invocations_total` — Tool usage (by tool name)
- `agent_websocket_clients` — Connected WebSocket clients
- `agent_db_size_bytes` — SQLite database size

## Troubleshooting

### WebSocket Connection Fails

```bash
# Check if WebSocket server is running
curl -i -N -H "Connection: Upgrade" \
  -H "Upgrade: websocket" \
  -H "Sec-WebSocket-Version: 13" \
  -H "Sec-WebSocket-Key: test" \
  http://localhost:3001/ws
```

**Fix**: Ensure `WS_PORT` matches in both server config and React client (`src/utils/websocket.ts`).

### Hook Events Not Appearing

1. Verify hook handler is executable:

```bash
chmod +x ~/.claude-code/hooks/hook_handler.py
```

2. Test hook manually:

```bash
echo '{"event":"tool_use","sessionId":"test123","agentId":"agent_main","toolName":"file_read"}' | \
  python3 ~/.claude-code/hooks/hook_handler.py
```

3. Check Claude Code logs:

```bash
tail -f ~/.claude-code/logs/hooks.log
```

### SQLite WAL Mode Issues

If you see `database is locked` errors:

```javascript
// server/db/index.js
const db = require('better-sqlite3')('./data/agent-monitor.db', {
  timeout: 5000,
  verbose: console.log
});

db.pragma('journal_mode = WAL');
db.pragma('busy_timeout = 5000');
```

Ensure no other process is holding a SHARED or EXCLUSIVE lock.

### Missing VAPID Keys

Generate new VAPID key pair:

```bash
npm run generate-vapid
# Outputs VAPID_PUBLIC_KEY and VAPID_PRIVATE_KEY
# Add to .env file
```

### Desktop App Won't Launch

**macOS**: Check security preferences:

```bash
