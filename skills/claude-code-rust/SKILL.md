---
name: claude-code-rust
description: High-performance Rust implementation of Claude Code with 2.5x faster startup, 97% smaller binary, and full MCP/REPL/plugin support
triggers:
  - use claude code rust
  - install claude code rust version
  - set up rust claude code
  - configure claude code rust mcp
  - run claude code repl in rust
  - create claude code plugin rust
  - optimize claude code with rust
  - migrate to claude code rust
---

# Claude Code Rust

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

Claude Code Rust is a complete Rust rewrite of Anthropic's Claude Code, delivering 2.5x faster startup (63ms vs 158ms), 97% smaller binary size (5MB vs 164MB), and zero runtime dependencies while maintaining 100% feature compatibility. It includes CLI tools, REPL mode, MCP server implementation, plugin system, and voice input support.

## Installation

### From Source (Recommended)

**Linux/macOS:**
```bash
# Clone repository
git clone https://github.com/lorryjovens-hub/claude-code-rust.git
cd claude-code-rust

# Build release binary
cargo build --release

# Install to system
sudo cp target/release/claude-code /usr/local/bin/
# or use install script
chmod +x scripts/install-linux.sh
./scripts/install-linux.sh
```

**Windows (PowerShell):**
```powershell
# Clone repository
git clone https://github.com/lorryjovens-hub/claude-code-rust.git
cd claude-code-rust

# Build release binary
cargo build --release

# Run install script (copies to Program Files)
.\scripts\install-windows.ps1
```

### Quick Build

```bash
# Development build (faster compilation)
cargo build

# Optimized release build
cargo build --release

# Run without installing
cargo run -- --help
```

## Configuration

### API Configuration

Create `~/.claude-code/config.toml`:

```toml
[api]
# Anthropic API
provider = "anthropic"
api_key_env = "ANTHROPIC_API_KEY"  # Reference env var
model = "claude-3-5-sonnet-20241022"
max_tokens = 4096

# Alternative: DeepSeek API
# provider = "deepseek"
# api_key_env = "DEEPSEEK_API_KEY"
# model = "deepseek-chat"
```

Set environment variable:
```bash
export ANTHROPIC_API_KEY="your-key-here"
# or for DeepSeek
export DEEPSEEK_API_KEY="your-key-here"
```

### MCP Server Configuration

Create `~/.claude-code/mcp-config.json`:

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "mcp-server-filesystem",
      "args": ["/path/to/workspace"],
      "env": {}
    },
    "git": {
      "command": "mcp-server-git",
      "args": [],
      "env": {
        "GIT_EDITOR": "vim"
      }
    }
  }
}
```

## CLI Usage

### Basic Commands

```bash
# Single query
claude-code "explain this code"

# With file context
claude-code "refactor this function" -f src/main.rs

# Interactive REPL mode
claude-code --repl

# Voice input mode
claude-code --voice

# Display version
claude-code --version

# Show help
claude-code --help
```

### REPL Mode

```bash
# Start interactive session
claude-code --repl

# In REPL:
> analyze the architecture
> /add src/api/client.rs      # Add file to context
> /context                    # Show current context
> /history                    # View conversation history
> /clear                      # Clear context
> /save session.json          # Save session
> /load session.json          # Restore session
> /exit                       # Exit REPL
```

### Memory Management

```bash
# Session-based memory
claude-code --repl --session my-project

# View memory stats
claude-code --memory-stats

# Consolidate memory (reduces token usage)
claude-code --consolidate-memory
```

## MCP Server Usage

### Starting MCP Server

```rust
use claude_code_rust::mcp::Server;
use claude_code_rust::config::McpConfig;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Load configuration
    let config = McpConfig::load()?;
    
    // Initialize server
    let server = Server::new(config);
    
    // Start server on stdio
    server.serve_stdio().await?;
    
    Ok(())
}
```

### Registering Custom Tools

```rust
use claude_code_rust::mcp::{Tool, ToolRegistry};
use serde_json::Value;

// Define custom tool
struct MyTool;

impl Tool for MyTool {
    fn name(&self) -> &str {
        "my_custom_tool"
    }
    
    fn description(&self) -> &str {
        "Performs custom analysis"
    }
    
    fn input_schema(&self) -> Value {
        serde_json::json!({
            "type": "object",
            "properties": {
                "input": {
                    "type": "string",
                    "description": "Input to analyze"
                }
            },
            "required": ["input"]
        })
    }
    
    async fn execute(&self, args: Value) -> Result<Value, Box<dyn std::error::Error>> {
        let input = args["input"].as_str().unwrap();
        // Process input
        Ok(serde_json::json!({
            "result": format!("Analyzed: {}", input)
        }))
    }
}

// Register tool
let mut registry = ToolRegistry::new();
registry.register(Box::new(MyTool));
```

### Resource Management

```rust
use claude_code_rust::mcp::{Resource, ResourceManager};

// Define custom resource
struct ProjectResource {
    path: String,
}

impl Resource for ProjectResource {
    fn uri(&self) -> String {
        format!("file://{}", self.path)
    }
    
    fn name(&self) -> String {
        "Project Files".to_string()
    }
    
    fn mime_type(&self) -> String {
        "text/plain".to_string()
    }
    
    async fn read(&self) -> Result<String, Box<dyn std::error::Error>> {
        std::fs::read_to_string(&self.path)
            .map_err(|e| e.into())
    }
}

// Use resource
let resource = ProjectResource {
    path: "src/main.rs".to_string()
};
let content = resource.read().await?;
```

## Plugin System

### Creating a Plugin

```rust
// plugins/my_plugin/src/lib.rs
use claude_code_rust::plugins::{Plugin, PluginContext};
use async_trait::async_trait;

pub struct MyPlugin;

#[async_trait]
impl Plugin for MyPlugin {
    fn name(&self) -> &str {
        "my-plugin"
    }
    
    fn version(&self) -> &str {
        "1.0.0"
    }
    
    async fn initialize(&mut self, ctx: &PluginContext) -> Result<(), Box<dyn std::error::Error>> {
        // Register custom commands
        ctx.register_command("greet", |args| {
            Box::pin(async move {
                Ok(format!("Hello, {}!", args.get("name").unwrap_or(&"World".to_string())))
            })
        });
        
        // Register hooks
        ctx.register_hook("before_query", |query| {
            Box::pin(async move {
                println!("Processing query: {}", query);
                Ok(query)
            })
        });
        
        Ok(())
    }
}

// Export plugin
#[no_mangle]
pub extern "C" fn _plugin_create() -> *mut dyn Plugin {
    Box::into_raw(Box::new(MyPlugin))
}
```

### Loading Plugins

```bash
# Install plugin
claude-code --install-plugin ./my-plugin.so

# List installed plugins
claude-code --list-plugins

# Use plugin command
claude-code --plugin my-plugin greet --name Alice
```

## API Client Usage

### Making API Calls

```rust
use claude_code_rust::api::{ApiClient, Message, Role};

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Initialize client
    let client = ApiClient::from_env()?;
    
    // Create messages
    let messages = vec![
        Message {
            role: Role::User,
            content: "Explain async/await in Rust".to_string(),
        }
    ];
    
    // Send request
    let response = client.create_message(messages).await?;
    
    println!("{}", response.content);
    
    Ok(())
}
```

### Streaming Responses

```rust
use claude_code_rust::api::ApiClient;
use futures::StreamExt;

let mut stream = client.create_message_stream(messages).await?;

while let Some(chunk) = stream.next().await {
    match chunk {
        Ok(content) => print!("{}", content),
        Err(e) => eprintln!("Stream error: {}", e),
    }
}
```

## Advanced Features

### Voice Input

```bash
# Start voice-enabled REPL
claude-code --voice

# Use keyboard shortcut in REPL
# Press SPACE to start recording
# Release SPACE to stop and transcribe
```

### SSH Remote Execution

```rust
use claude_code_rust::advanced::ssh::SshClient;

let client = SshClient::connect("user@host:22", "~/.ssh/id_rsa").await?;
let result = client.execute("ls -la").await?;
println!("{}", result);
```

### Project Initialization

```bash
# Initialize new project with AI assistance
claude-code --init my-new-project

# Specify template
claude-code --init my-app --template web-server

# Interactive mode
claude-code --init --interactive
```

## Performance Optimization

### Configuration Tuning

```toml
# ~/.claude-code/config.toml
[performance]
# Memory consolidation threshold (messages)
consolidation_threshold = 20

# Maximum context window (tokens)
max_context_tokens = 100000

# Parallel tool execution
parallel_tools = true

# Cache responses
response_cache = true
cache_ttl_seconds = 3600
```

### Memory Usage

```rust
use claude_code_rust::memory::{Session, ConsolidationStrategy};

let mut session = Session::new("my-session");

// Add messages
session.add_message(message);

// Consolidate when needed (reduces token usage)
if session.message_count() > 20 {
    session.consolidate(ConsolidationStrategy::Summary).await?;
}

// Persist to disk
session.save("~/.claude-code/sessions/my-session.json")?;
```

## Common Patterns

### Error Handling

```rust
use claude_code_rust::error::{Error, Result};

fn process_query(query: &str) -> Result<String> {
    if query.is_empty() {
        return Err(Error::InvalidInput("Query cannot be empty".to_string()));
    }
    
    // Process query
    Ok("Result".to_string())
}

// Usage
match process_query(input) {
    Ok(result) => println!("{}", result),
    Err(Error::ApiError(e)) => eprintln!("API error: {}", e),
    Err(Error::ConfigError(e)) => eprintln!("Config error: {}", e),
    Err(e) => eprintln!("Error: {}", e),
}
```

### Context Management

```rust
use claude_code_rust::memory::Context;

let mut context = Context::new();

// Add files
context.add_file("src/main.rs").await?;
context.add_directory("src/api", true).await?; // recursive

// Add inline content
context.add_snippet("Custom context", "fn example() {}");

// Limit token count
context.set_max_tokens(50000);
let truncated = context.truncate()?;

// Get formatted context
let formatted = context.to_string();
```

## Troubleshooting

### API Key Issues

```bash
# Verify API key is set
echo $ANTHROPIC_API_KEY

# Test connection
claude-code "test" --debug

# Check config
cat ~/.claude-code/config.toml
```

### MCP Server Not Starting

```bash
# Check MCP configuration
cat ~/.claude-code/mcp-config.json

# Verify server executables exist
which mcp-server-filesystem

# Run with debug logging
RUST_LOG=debug claude-code --repl
```

### Build Errors

```bash
# Update Rust toolchain
rustup update stable

# Clean build cache
cargo clean

# Rebuild with verbose output
cargo build --release --verbose

# Check for missing system dependencies
# Linux: sudo apt-get install libssl-dev pkg-config
# macOS: brew install openssl
```

### Performance Issues

```bash
# Enable optimization flags
export RUSTFLAGS="-C target-cpu=native"

# Profile binary size
cargo bloat --release

# Check memory usage
/usr/bin/time -v ./target/release/claude-code --help

# Reduce context size
claude-code --max-context 50000
```

### Plugin Errors

```bash
# Verify plugin compatibility
claude-code --verify-plugin ./my-plugin.so

# Check plugin logs
cat ~/.claude-code/plugins/my-plugin.log

# Rebuild plugin
cd my-plugin && cargo build --release

# Reinstall
claude-code --uninstall-plugin my-plugin
claude-code --install-plugin ./target/release/libmy_plugin.so
```

## Environment Variables

```bash
# API Configuration
export ANTHROPIC_API_KEY="sk-ant-..."
export DEEPSEEK_API_KEY="sk-..."

# Logging
export RUST_LOG=debug              # debug, info, warn, error
export RUST_BACKTRACE=1            # Enable backtraces

# Performance
export CLAUDE_CODE_CACHE_DIR="~/.cache/claude-code"
export CLAUDE_CODE_MAX_WORKERS=4   # Parallel tool workers

# MCP
export MCP_CONFIG_PATH="~/.claude-code/mcp-config.json"
```

## Integration Examples

### VS Code Extension

```typescript
// extension.ts
import { spawn } from 'child_process';

const claudeCode = spawn('claude-code', ['--repl']);

claudeCode.stdin.write('analyze this code\n');
claudeCode.stdout.on('data', (data) => {
  console.log(`Response: ${data}`);
});
```

### CI/CD Pipeline

```yaml
# .github/workflows/ai-review.yml
name: AI Code Review
on: [pull_request]
jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Install Claude Code Rust
        run: |
          curl -L https://github.com/lorryjovens-hub/claude-code-rust/releases/latest/download/claude-code-linux-x64 -o /usr/local/bin/claude-code
          chmod +x /usr/local/bin/claude-code
      - name: Review PR
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          git diff origin/main...HEAD | claude-code "review this PR diff"
```
