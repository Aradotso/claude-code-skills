---
name: claude-code-security-review-action
description: AI-powered security review GitHub Action using Claude to analyze code changes for vulnerabilities in pull requests
triggers:
  - set up security review action
  - configure claude security scanning
  - add automated security analysis to PR
  - create security review workflow
  - scan code for security vulnerabilities with claude
  - integrate claude code security review
  - configure security github action
  - add ai security scanning to repository
---

# Claude Code Security Review Action

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## What It Does

Claude Code Security Review is a GitHub Action that uses Anthropic's Claude AI to perform intelligent, context-aware security analysis on pull requests. Unlike traditional SAST tools that rely on pattern matching, it understands code semantics to detect vulnerabilities like SQL injection, XSS, authentication flaws, cryptographic issues, and business logic vulnerabilities while minimizing false positives.

## Installation

### Basic GitHub Action Setup

Create `.github/workflows/security.yml`:

```yaml
name: Security Review

permissions:
  pull-requests: write  # Required for PR comments
  contents: read

on:
  pull_request:

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.head.sha || github.sha }}
          fetch-depth: 2
      
      - uses: anthropics/claude-code-security-review@main
        with:
          comment-pr: true
          claude-api-key: ${{ secrets.CLAUDE_API_KEY }}
```

### API Key Setup

1. Get a Claude API key from [Anthropic Console](https://console.anthropic.com/)
2. Ensure the key has both Claude API and Claude Code usage enabled
3. Add to GitHub repository secrets as `CLAUDE_API_KEY`:
   - Repository Settings → Secrets and variables → Actions → New repository secret

### Security Hardening

For public repositories, enable maintainer approval for external contributors:

Repository Settings → Actions → General → Fork pull request workflows → "Require approval for all outside collaborators"

This prevents prompt injection attacks from untrusted PRs.

## Configuration Options

### Complete Configuration Example

```yaml
name: Security Review

permissions:
  pull-requests: write
  contents: read

on:
  pull_request:
    branches: [main, develop]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.head.sha || github.sha }}
          fetch-depth: 2
      
      - uses: anthropics/claude-code-security-review@main
        with:
          claude-api-key: ${{ secrets.CLAUDE_API_KEY }}
          comment-pr: true
          upload-results: true
          exclude-directories: "tests/,docs/,examples/"
          claude-model: "claude-opus-4-1-20250805"
          claudecode-timeout: "20"
          run-every-commit: false
          false-positive-filtering-instructions: ".github/security/fp-filter.txt"
          custom-security-scan-instructions: ".github/security/custom-rules.txt"
```

### Input Parameters

| Input | Description | Default | Required |
|-------|-------------|---------|----------|
| `claude-api-key` | Anthropic API key (needs Claude API + Claude Code access) | - | Yes |
| `comment-pr` | Post findings as PR comments | `true` | No |
| `upload-results` | Upload results as workflow artifacts | `true` | No |
| `exclude-directories` | Comma-separated directories to skip | - | No |
| `claude-model` | Claude model to use | `claude-opus-4-1-20250805` | No |
| `claudecode-timeout` | Analysis timeout in minutes | `20` | No |
| `run-every-commit` | Skip cache, run on every commit | `false` | No |
| `false-positive-filtering-instructions` | Path to FP filter instructions file | - | No |
| `custom-security-scan-instructions` | Path to custom audit rules file | - | No |

### Output Parameters

Access outputs in subsequent workflow steps:

```yaml
- uses: anthropics/claude-code-security-review@main
  id: security-scan
  with:
    claude-api-key: ${{ secrets.CLAUDE_API_KEY }}

- name: Check findings
  run: |
    echo "Found ${{ steps.security-scan.outputs.findings-count }} security issues"
    echo "Results saved to ${{ steps.security-scan.outputs.results-file }}"
```

## Custom Security Rules

### Custom Scan Instructions

Create `.github/security/custom-rules.txt`:

```text
Additional security requirements for this project:

1. All database queries must use parameterized queries or an ORM
2. API endpoints handling PII must log access with user ID and timestamp
3. All file uploads must validate MIME type server-side
4. Authentication tokens must expire within 15 minutes
5. All external API calls must include timeout limits (max 30 seconds)
6. Cryptographic operations must use approved algorithms: AES-256, RSA-4096, SHA-256
7. User input in log messages must be sanitized to prevent log injection
```

Reference in workflow:

```yaml
- uses: anthropics/claude-code-security-review@main
  with:
    claude-api-key: ${{ secrets.CLAUDE_API_KEY }}
    custom-security-scan-instructions: ".github/security/custom-rules.txt"
```

### Custom False Positive Filtering

Create `.github/security/fp-filter.txt`:

```text
False positive filtering rules:

1. Ignore rate limiting issues in test files
2. Ignore SQL injection warnings for queries using our SafeQuery wrapper class
3. Ignore hardcoded credentials in files matching pattern: *_fixture.py
4. Accept usage of MD5 for non-security purposes (ETags, cache keys)
5. Allow eval() usage in our DSL parser module (already sandboxed)
```

Reference in workflow:

```yaml
- uses: anthropics/claude-code-security-review@main
  with:
    claude-api-key: ${{ secrets.CLAUDE_API_KEY }}
    false-positive-filtering-instructions: ".github/security/fp-filter.txt"
```

## Advanced Workflow Patterns

### Conditional Execution Based on Findings

```yaml
jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.head.sha || github.sha }}
          fetch-depth: 2
      
      - uses: anthropics/claude-code-security-review@main
        id: security
        with:
          claude-api-key: ${{ secrets.CLAUDE_API_KEY }}
      
      - name: Fail if critical findings
        if: steps.security.outputs.findings-count > 0
        run: |
          echo "::error::Found ${{ steps.security.outputs.findings-count }} security issues"
          exit 1
      
      - name: Notify security team
        if: steps.security.outputs.findings-count > 0
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: '@security-team Please review security findings'
            })
```

### Multi-Environment Analysis

```yaml
jobs:
  security-staging:
    runs-on: ubuntu-latest
    if: github.base_ref == 'staging'
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.head.sha }}
          fetch-depth: 2
      
      - uses: anthropics/claude-code-security-review@main
        with:
          claude-api-key: ${{ secrets.CLAUDE_API_KEY }}
          custom-security-scan-instructions: ".github/security/staging-rules.txt"

  security-production:
    runs-on: ubuntu-latest
    if: github.base_ref == 'main'
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.head.sha }}
          fetch-depth: 2
      
      - uses: anthropics/claude-code-security-review@main
        with:
          claude-api-key: ${{ secrets.CLAUDE_API_KEY }}
          custom-security-scan-instructions: ".github/security/production-rules.txt"
          run-every-commit: true  # Stricter for production
```

### Scheduled Security Audits

```yaml
name: Weekly Security Audit

on:
  schedule:
    - cron: '0 2 * * 1'  # Every Monday at 2 AM
  workflow_dispatch:

jobs:
  full-audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: anthropics/claude-code-security-review@main
        with:
          claude-api-key: ${{ secrets.CLAUDE_API_KEY }}
          comment-pr: false
          upload-results: true
      
      - name: Create issue if findings
        uses: actions/github-script@v7
        if: steps.security.outputs.findings-count > 0
        with:
          script: |
            const fs = require('fs');
            const results = fs.readFileSync('${{ steps.security.outputs.results-file }}', 'utf8');
            github.rest.issues.create({
              owner: context.repo.owner,
              repo: context.repo.repo,
              title: 'Weekly Security Audit Findings',
              body: '```json\n' + results + '\n```',
              labels: ['security', 'audit']
            })
```

## Claude Code Integration

### Built-in `/security-review` Command

Claude Code includes a built-in `/security-review` slash command for interactive security analysis:

```bash
# In Claude Code, run:
/security-review
```

This performs the same analysis as the GitHub Action but in your local environment.

### Customizing the Slash Command

1. Copy the command template to your project:

```bash
mkdir -p .claude/commands
curl -o .claude/commands/security-review.md \
  https://raw.githubusercontent.com/anthropics/claude-code-security-review/main/.claude/commands/security-review.md
```

2. Edit `.claude/commands/security-review.md` to customize:

```markdown
---
name: security-review
description: Comprehensive security review of code changes
---

# Security Review

Perform a thorough security analysis of all pending changes.

## Custom Rules for This Project

- All API endpoints must validate authentication tokens
- Database queries must use parameterized statements
- User input must be sanitized before logging
- File uploads require MIME type validation

## Analysis Steps

1. Identify all modified files
2. Analyze each change for security vulnerabilities
3. Check against project-specific security requirements
4. Generate findings with severity ratings and remediation guidance
```

## Local Development & Testing

### Running Locally Against a PR

```bash
# Clone the repository
git clone https://github.com/anthropics/claude-code-security-review.git
cd claude-code-security-review

# Install dependencies
pip install -r claudecode/requirements.txt

# Set environment variables
export ANTHROPIC_API_KEY="your-api-key"
export GITHUB_TOKEN="your-github-token"

# Run against a specific PR
python claudecode/evals/run_eval.py \
  --repo owner/repo \
  --pr-number 123 \
  --output results.json
```

### Running Tests

```bash
# Run all tests
pytest claudecode -v

# Run specific test suite
pytest claudecode/test_findings_filter.py -v

# Run with coverage
pytest claudecode --cov=claudecode --cov-report=html
```

## Vulnerability Detection Capabilities

The action detects:

- **Injection Attacks**: SQL, command, LDAP, XPath, NoSQL, XXE
- **Auth/Authz Issues**: Broken authentication, privilege escalation, IDOR, session flaws
- **Data Exposure**: Hardcoded secrets, sensitive logging, PII violations
- **Crypto Issues**: Weak algorithms, poor key management, insecure RNG
- **Input Validation**: Missing validation, improper sanitization, buffer overflows
- **Business Logic**: Race conditions, TOCTOU issues
- **XSS**: Reflected, stored, DOM-based
- **Configuration**: Insecure defaults, missing headers, permissive CORS
- **Supply Chain**: Vulnerable dependencies, typosquatting

## Troubleshooting

### Action Fails with "API Key Invalid"

Ensure your API key has both Claude API and Claude Code access enabled in the Anthropic Console.

### No Findings Posted to PR

Check workflow logs for:
- Permission errors (needs `pull-requests: write`)
- API rate limits
- Network connectivity issues

Verify permissions:

```yaml
permissions:
  pull-requests: write
  contents: read
```

### High False Positive Rate

1. Add custom filtering instructions
2. Exclude test/example directories
3. Tune the model or timeout settings

```yaml
- uses: anthropics/claude-code-security-review@main
  with:
    claude-api-key: ${{ secrets.CLAUDE_API_KEY }}
    exclude-directories: "tests/,examples/,docs/"
    false-positive-filtering-instructions: ".github/security/fp-filter.txt"
```

### Timeout Errors

Increase the timeout for large PRs:

```yaml
- uses: anthropics/claude-code-security-review@main
  with:
    claude-api-key: ${{ secrets.CLAUDE_API_KEY }}
    claudecode-timeout: "30"  # 30 minutes
```

### Results Not Uploaded

Ensure artifact upload is enabled:

```yaml
- uses: anthropics/claude-code-security-review@main
  with:
    claude-api-key: ${{ secrets.CLAUDE_API_KEY }}
    upload-results: true
```

Access artifacts in Actions tab → Workflow run → Artifacts section.

### Cache Issues on Multi-Commit PRs

Force analysis on every commit:

```yaml
- uses: anthropics/claude-code-security-review@main
  with:
    claude-api-key: ${{ secrets.CLAUDE_API_KEY }}
    run-every-commit: true
```

Note: This may increase false positives but ensures comprehensive coverage.
