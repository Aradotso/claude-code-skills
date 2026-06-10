---
name: claude-code-prompts
description: Expert in using Claude Code prompt templates for AI coding agents — system prompts, tool prompts, agent delegation, memory management, and multi-agent coordination
triggers:
  - "use claude code prompts"
  - "set up coding agent prompts"
  - "implement agent delegation patterns"
  - "add memory management to my agent"
  - "create multi-agent coordinator"
  - "structure agent system prompt"
  - "implement verification agent"
  - "add cursor skills from claude-code-prompts"
---

# Claude Code Prompts Skill

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

Expert skill for using the claude-code-prompts repository — independently authored prompt templates for building AI coding agents with proper system prompts, tool routing, agent delegation, memory management, and multi-agent coordination patterns.

## What This Project Does

`claude-code-prompts` provides production-ready prompt templates implementing the behavioral patterns observed in Claude Code:

- **System prompts** — agent identity, safety rules, tool routing, output format
- **Tool prompts** — shell, file operations, search, web, agent launcher, user interaction
- **Agent prompts** — specialized subagents (explorer, architect, verifier, docs)
- **Memory prompts** — conversation summarization, session notes, memory extraction/consolidation
- **Coordinator prompt** — multi-agent orchestration with delegation and synthesis
- **Utility prompts** — session titles, tool summaries, away recaps, next-action suggestions
- **Pattern analyses** — deep dives into each pattern with reusable templates
- **Cursor skills** — drop-in skills implementing key patterns

All prompts are independently authored, informed by studying how Claude Code works in practice.

## Installation

### Clone the Repository

```bash
git clone https://github.com/repowise-dev/claude-code-prompts.git
cd claude-code-prompts
```

### Install Cursor Skills (Optional)

For Cursor IDE users:

```bash
# Copy skills to Cursor's skills directory
cp -r skills/* ~/.cursor/skills-cursor/

# Or create symlinks for auto-updates
ln -s "$(pwd)/skills/coding-agent-standards" ~/.cursor/skills-cursor/coding-agent-standards
ln -s "$(pwd)/skills/verification-agent" ~/.cursor/skills-cursor/verification-agent
ln -s "$(pwd)/skills/prompt-architect" ~/.cursor/skills-cursor/prompt-architect
```

Skills will be available immediately in Cursor — no restart required.

## Project Structure

```
claude-code-prompts/
├── complete-prompts/          # Ready-to-use complete prompts
│   ├── system-prompt.md       # Main agent system prompt
│   ├── coordinator-prompt.md  # Multi-agent coordinator
│   ├── tool-prompts/          # 11 tool-specific prompts
│   ├── agent-prompts/         # 5 specialized agent prompts
│   ├── memory-prompts/        # 4 memory management prompts
│   └── utility-prompts/       # 4 helper prompts
├── patterns/                  # Pattern analysis and templates
│   ├── 01-system-prompt-architecture.md
│   ├── 02-core-behavioral-rules.md
│   ├── 03-safety-and-risk-assessment.md
│   ├── 04-tool-specific-instructions.md
│   ├── 05-agent-delegation.md
│   ├── 06-verification-and-testing.md
│   ├── 07-memory-and-context.md
│   ├── 08-multi-agent-coordination.md
│   └── 09-auxiliary-prompts.md
└── skills/                    # Cursor IDE skills
    ├── coding-agent-standards/
    ├── verification-agent/
    └── prompt-architect/
```

## Key Usage Patterns

### 1. Using the System Prompt

The system prompt defines agent identity, safety rules, and tool routing:

```markdown
# In your agent configuration, use complete-prompts/system-prompt.md

# Replace placeholders:
{{AGENT_NAME}} → "CodeAssistant"
{{ALLOWED_OPERATIONS}} → "file read/write, shell execution, web search"
{{RESTRICTED_OPERATIONS}} → "system-wide package changes, database migrations"
{{PROJECT_CONVENTIONS}} → "TypeScript strict mode, React functional components"
```

**Key sections to customize:**

- **Identity & Purpose** — what your agent does
- **Permitted Operations** — allowed/restricted actions
- **Code Style Preferences** — language conventions, formatting
- **Safety & Risk Assessment** — reversibility tiers, destructive action gates
- **Tool Routing** — when to use each tool

### 2. Implementing Tool Prompts

Each tool has specific usage patterns and constraints:

```markdown
# File Edit Pattern (complete-prompts/tool-prompts/file-edit.md)

When editing files:
1. Read the file first to verify content
2. Use exact string matching (no regex)
3. Ensure OLD_TEXT appears exactly once
4. Show minimal context diffs
5. Never edit generated/vendor files

# Shell Execution Pattern (complete-prompts/tool-prompts/shell-execution.md)

Before shell commands:
1. Check if operation is reversible
2. Use git safety for code changes
3. Show command + expected outcome
4. Prefer atomic operations
```

### 3. Agent Delegation Pattern

Spawn specialized subagents for complex tasks:

```markdown
# Pattern from complete-prompts/tool-prompts/task-management.md

When to delegate:
- Research tasks → General Purpose Agent
- Codebase exploration → Code Explorer Agent
- Design planning → Solution Architect Agent
- Testing/validation → Verification Specialist Agent
- Documentation → Documentation Guide Agent

# Example delegation
Task: "Verify the authentication flow"
→ Launch Verification Specialist Agent
→ Provide: entry points, expected behavior, edge cases
→ Expect: PASS/FAIL/PARTIAL verdict with evidence
```

### 4. Memory Management Pattern

Implement context compression for long sessions:

```markdown
# Pattern from complete-prompts/memory-prompts/conversation-summary.md

Memory structure (9 sections):
1. Session context (1 line)
2. Key decisions made
3. Problems solved
4. Active tasks
5. Blocked/pending work
6. Important discoveries
7. User preferences revealed
8. Errors/warnings to remember
9. Next session priorities

Update memory when:
- Context window > 80% full
- Major decision points
- Session handoffs
- User requests recap
```

### 5. Multi-Agent Coordination

Orchestrate multiple workers with the coordinator pattern:

```markdown
# Pattern from complete-prompts/coordinator-prompt.md

Coordinator responsibilities:
1. Decompose complex requests into subtasks
2. Assign tasks to specialized workers
3. Monitor progress and handle blockers
4. Synthesize worker results into coherent response
5. Verify completeness before returning to user

# Example workflow
User request: "Refactor auth system and add OAuth"
→ Coordinator creates plan
→ Architect Agent: design new auth architecture
→ Explorer Agent: map current auth implementation
→ General Agent: implement OAuth integration
→ Verifier Agent: test all auth flows
→ Coordinator: synthesize results, verify completeness
```

### 6. Verification Agent Pattern

Implement adversarial testing:

```markdown
# Pattern from skills/verification-agent/SKILL.md

Verification approach:
1. Understand expected behavior
2. Identify verification strategy (see strategies.md)
3. Design tests that could fail
4. Execute verification
5. Return verdict: PASS / FAIL / PARTIAL

# Example strategies
- Static analysis (code review without execution)
- Dynamic testing (run tests, inspect output)
- Integration testing (test component interactions)
- Boundary testing (edge cases, error conditions)
- Regression testing (compare before/after)
```

## Real Code Examples

### Example 1: Setting Up a Basic Coding Agent

```python
# agent_config.py
import anthropic
from pathlib import Path

# Load system prompt template
system_prompt_path = Path("claude-code-prompts/complete-prompts/system-prompt.md")
system_prompt = system_prompt_path.read_text()

# Customize for your project
system_prompt = system_prompt.replace("{{AGENT_NAME}}", "PythonAssistant")
system_prompt = system_prompt.replace("{{ALLOWED_OPERATIONS}}", 
    "file read/write, shell execution (python, pytest, git), web search")
system_prompt = system_prompt.replace("{{RESTRICTED_OPERATIONS}}", 
    "system package installation, database operations")
system_prompt = system_prompt.replace("{{PROJECT_CONVENTIONS}}", 
    "Python 3.11+, type hints required, pytest for testing, black formatting")

# Initialize agent
client = anthropic.Anthropic(api_key=os.environ.get("ANTHROPIC_API_KEY"))

def create_agent_message(user_request: str) -> str:
    message = client.messages.create(
        model="claude-3-7-sonnet-20250219",
        max_tokens=4096,
        system=system_prompt,
        messages=[{"role": "user", "content": user_request}]
    )
    return message.content[0].text
```

### Example 2: Implementing Memory Management

```python
# memory_manager.py
from pathlib import Path
from typing import List, Dict

class AgentMemory:
    def __init__(self, session_id: str):
        self.session_id = session_id
        self.memory_template = Path(
            "claude-code-prompts/complete-prompts/memory-prompts/conversation-summary.md"
        ).read_text()
        self.memory_file = Path(f".agent_memory/{session_id}.md")
        self.memory_file.parent.mkdir(exist_ok=True)
    
    def should_summarize(self, messages: List[Dict]) -> bool:
        """Check if context window is getting full"""
        total_tokens = sum(len(m.get("content", "")) for m in messages)
        return total_tokens > 150000  # ~80% of 200k context window
    
    def create_summary(self, messages: List[Dict], client) -> str:
        """Generate 9-section memory summary"""
        summary_request = f"""
        Using this template, summarize our conversation:
        
        {self.memory_template}
        
        Recent messages:
        {messages[-20:]}  # Last 20 messages for context
        """
        
        response = client.messages.create(
            model="claude-3-7-sonnet-20250219",
            max_tokens=2000,
            messages=[{"role": "user", "content": summary_request}]
        )
        
        summary = response.content[0].text
        self.memory_file.write_text(summary)
        return summary
    
    def load_memory(self) -> str:
        """Load existing memory for new session"""
        if self.memory_file.exists():
            return self.memory_file.read_text()
        return ""
```

### Example 3: Multi-Agent Coordinator

```python
# coordinator.py
from typing import List, Dict, Literal
from pathlib import Path
import anthropic

AgentType = Literal["explorer", "architect", "verifier", "general"]

class AgentCoordinator:
    def __init__(self, api_key: str):
        self.client = anthropic.Anthropic(api_key=api_key)
        self.coordinator_prompt = Path(
            "claude-code-prompts/complete-prompts/coordinator-prompt.md"
        ).read_text()
        self.agent_prompts = self._load_agent_prompts()
    
    def _load_agent_prompts(self) -> Dict[AgentType, str]:
        """Load specialized agent prompts"""
        base_path = Path("claude-code-prompts/complete-prompts/agent-prompts")
        return {
            "explorer": (base_path / "code-explorer.md").read_text(),
            "architect": (base_path / "solution-architect.md").read_text(),
            "verifier": (base_path / "verification-specialist.md").read_text(),
            "general": (base_path / "general-purpose.md").read_text(),
        }
    
    def execute_complex_task(self, user_request: str) -> str:
        """Coordinate multiple agents for complex task"""
        # 1. Coordinator creates plan
        plan = self._create_plan(user_request)
        
        # 2. Execute subtasks with specialized agents
        results = []
        for subtask in plan["subtasks"]:
            agent_type = subtask["agent_type"]
            result = self._execute_subtask(
                agent_type=agent_type,
                task=subtask["description"],
                context=subtask.get("context", "")
            )
            results.append({
                "subtask": subtask["description"],
                "result": result,
                "agent": agent_type
            })
        
        # 3. Coordinator synthesizes results
        final_response = self._synthesize_results(user_request, results)
        return final_response
    
    def _create_plan(self, user_request: str) -> Dict:
        """Use coordinator to decompose request"""
        response = self.client.messages.create(
            model="claude-3-7-sonnet-20250219",
            max_tokens=2000,
            system=self.coordinator_prompt,
            messages=[{
                "role": "user",
                "content": f"Create execution plan for: {user_request}"
            }]
        )
        # Parse response into structured plan
        # (implementation depends on your response format)
        return self._parse_plan(response.content[0].text)
    
    def _execute_subtask(self, agent_type: AgentType, task: str, context: str) -> str:
        """Execute subtask with specialized agent"""
        agent_prompt = self.agent_prompts[agent_type]
        
        response = self.client.messages.create(
            model="claude-3-7-sonnet-20250219",
            max_tokens=4000,
            system=agent_prompt,
            messages=[{
                "role": "user",
                "content": f"Context: {context}\n\nTask: {task}"
            }]
        )
        return response.content[0].text
    
    def _synthesize_results(self, original_request: str, results: List[Dict]) -> str:
        """Coordinator synthesizes worker results"""
        synthesis_request = f"""
        Original request: {original_request}
        
        Worker results:
        {chr(10).join(f"- [{r['agent']}] {r['subtask']}: {r['result']}" for r in results)}
        
        Synthesize into coherent response that fully addresses the user's request.
        """
        
        response = self.client.messages.create(
            model="claude-3-7-sonnet-20250219",
            max_tokens=4000,
            system=self.coordinator_prompt,
            messages=[{"role": "user", "content": synthesis_request}]
        )
        return response.content[0].text
```

### Example 4: Verification Agent Implementation

```python
# verifier.py
from pathlib import Path
from typing import Literal
import anthropic

Verdict = Literal["PASS", "FAIL", "PARTIAL"]

class VerificationAgent:
    def __init__(self, api_key: str):
        self.client = anthropic.Anthropic(api_key=api_key)
        self.verifier_prompt = Path(
            "claude-code-prompts/complete-prompts/agent-prompts/verification-specialist.md"
        ).read_text()
        self.strategies = Path(
            "claude-code-prompts/skills/verification-agent/strategies.md"
        ).read_text()
    
    def verify(
        self, 
        description: str, 
        implementation: str, 
        expected_behavior: str
    ) -> Dict[str, any]:
        """
        Verify implementation matches expected behavior
        
        Returns:
            {
                "verdict": "PASS" | "FAIL" | "PARTIAL",
                "evidence": str,
                "issues": List[str],
                "recommendations": List[str]
            }
        """
        verification_request = f"""
        Verify this implementation using adversarial testing approach.
        
        Available strategies:
        {self.strategies}
        
        Description: {description}
        Expected behavior: {expected_behavior}
        Implementation: {implementation}
        
        Provide:
        1. Chosen verification strategy
        2. Test design that could fail
        3. Execution results
        4. Verdict: PASS / FAIL / PARTIAL
        5. Evidence supporting verdict
        6. Issues found (if any)
        7. Recommendations
        """
        
        response = self.client.messages.create(
            model="claude-3-7-sonnet-20250219",
            max_tokens=4000,
            system=self.verifier_prompt,
            messages=[{"role": "user", "content": verification_request}]
        )
        
        return self._parse_verification_result(response.content[0].text)
    
    def _parse_verification_result(self, response: str) -> Dict:
        """Parse verification response into structured result"""
        # Extract verdict (PASS/FAIL/PARTIAL)
        verdict = "PARTIAL"  # default
        if "VERDICT: PASS" in response:
            verdict = "PASS"
        elif "VERDICT: FAIL" in response:
            verdict = "FAIL"
        
        return {
            "verdict": verdict,
            "evidence": response,
            "issues": self._extract_issues(response),
            "recommendations": self._extract_recommendations(response)
        }
```

## Configuration

### Customizing System Prompt

Key placeholders to replace in `complete-prompts/system-prompt.md`:

```markdown
{{AGENT_NAME}} — Your agent's name (e.g., "CodeAssistant", "PythonHelper")
{{ALLOWED_OPERATIONS}} — Permitted operations (e.g., "file read/write, shell, web search")
{{RESTRICTED_OPERATIONS}} — Forbidden operations (e.g., "database changes, system installs")
{{PROJECT_CONVENTIONS}} — Code style rules (e.g., "TypeScript strict, ESLint, Prettier")
{{RISK_POLICY}} — Risk assessment rules (e.g., "require approval for production deploys")
```

### Customizing Tool Prompts

Each tool prompt in `complete-prompts/tool-prompts/` can be customized:

```markdown
# shell-execution.md
{{SANDBOX_PATH}} — Safe execution directory
{{GIT_WORKFLOW}} — Your git commit/PR workflow
{{ALLOWED_COMMANDS}} — Permitted shell commands
{{BLOCKED_COMMANDS}} — Forbidden commands (rm -rf, sudo, etc.)

# file-edit.md
{{MAX_FILE_SIZE}} — Maximum file size for edits
{{EXCLUDED_PATTERNS}} — Files to never edit (vendor/, dist/, etc.)

# web-search.md
{{SEARCH_API}} — Your search provider (Brave, Google, etc.)
{{CITATION_FORMAT}} — How to format source citations
```

### Customizing Agent Prompts

Specialized agent prompts in `complete-prompts/agent-prompts/`:

```markdown
# verification-specialist.md
{{TESTING_FRAMEWORK}} — Your test framework (pytest, jest, etc.)
{{COVERAGE_THRESHOLD}} — Minimum test coverage
{{CRITICAL_PATHS}} — Paths requiring extra verification

# solution-architect.md
{{ARCHITECTURE_STYLE}} — Your preferred architecture (microservices, monolith, etc.)
{{TECH_STACK}} — Allowed technologies
{{DESIGN_PRINCIPLES}} — SOLID, DRY, etc.
```

## Common Patterns

### Pattern: Progressive Enhancement

Start simple, add complexity only when needed:

```python
# 1. Start with basic system prompt
agent = BasicAgent(system_prompt="complete-prompts/system-prompt.md")

# 2. Add memory when sessions get long
if session_length > 100:
    agent.enable_memory("complete-prompts/memory-prompts/conversation-summary.md")

# 3. Add delegation for complex tasks
if task_complexity > threshold:
    agent.enable_delegation("complete-prompts/agent-prompts/")

# 4. Add coordination for multi-step workflows
if needs_coordination:
    coordinator = AgentCoordinator("complete-prompts/coordinator-prompt.md")
```

### Pattern: Safety Gates

Implement risk assessment before destructive operations:

```python
# From complete-prompts/system-prompt.md
def assess_operation_risk(operation: str, target: str) -> str:
    """
    Risk tiers:
    - REVERSIBLE: can undo (git operations, file edits)
    - SEMI_REVERSIBLE: can restore from backup
    - IRREVERSIBLE: cannot undo (rm -rf, database drops)
    """
    if operation in ["delete", "drop", "truncate"]:
        if "production" in target or "main" in target:
            return "IRREVERSIBLE"
        return "SEMI_REVERSIBLE"
    
    if operation in ["edit", "commit", "create"]:
        return "REVERSIBLE"
    
    return "SEMI_REVERSIBLE"

# Require user confirmation for high-risk operations
risk = assess_operation_risk(operation, target)
if risk == "IRREVERSIBLE":
    user_approval = ask_user(f"Confirm {operation} on {target}? [y/N]")
    if user_approval != "y":
        abort()
```

### Pattern: Tool Routing

Route requests to appropriate tools based on intent:

```python
# From complete-prompts/system-prompt.md
def route_to_tool(user_intent: str) -> str:
    """Route user intent to appropriate tool"""
    intent_patterns = {
        "read": ["file_read"],
        "search": ["search_grep", "search_glob"],
        "modify": ["file_edit"],
        "create": ["file_write"],
        "execute": ["shell_execution"],
        "research": ["web_search", "web_fetch"],
        "complex_task": ["agent_launcher"],
        "clarification": ["ask_user"],
        "planning": ["plan_mode"]
    }
    
    # Match intent to tool (simplified)
    for intent, tools in intent_patterns.items():
        if intent in user_intent.lower():
            return tools[0]
    
    return "ask_user"  # default to asking for clarification
```

## Troubleshooting

### Issue: Agent is Too Verbose

**Solution**: Emphasize efficiency rules in system prompt

```markdown
# Add to system prompt
OUTPUT EFFICIENCY:
- Lead with action, not preamble
- No "Certainly!", "Great question!", etc.
- Show code diffs, not full files
- Limit explanations to what's non-obvious
- Use bullet points over paragraphs
```

### Issue: Agent Makes Unnecessary Changes

**Solution**: Strengthen anti-over-engineering rules

```markdown
# Add to system prompt
CORE BEHAVIORAL RULES:
- Make minimal necessary changes only
- Don't refactor code that isn't broken
- Don't optimize prematurely
- Don't add features unless requested
- Ask before expanding scope
```

### Issue: Memory Not Persisting Across Sessions

**Solution**: Implement proper memory loading

```python
# Load memory at session start
def start_session(session_id: str, memory_manager: AgentMemory):
    previous_memory = memory_manager.load_memory()
    
    system_prompt = base_system_prompt + f"""
    
    PREVIOUS SESSION MEMORY:
    {previous_memory}
    
    Continue from where we left off.
    """
    return system_prompt
```

### Issue: Verification Agent Too Lenient

**Solution**: Use adversarial verification approach

```markdown
# In verification request
Your goal is to FIND BUGS, not confirm the code works.
Design tests that are LIKELY TO FAIL.
Be suspicious of edge cases, error handling, race conditions.
Return PASS only if you genuinely cannot break it.
```

### Issue: Multi-Agent Coordination Inefficient

**Solution**: Implement proper synthesis pattern

```python
# Coordinator must synthesize, not just concatenate
def synthesize_results(results: List[Dict]) -> str:
    """
    BAD: return "\n".join(r["result"] for r in results)
    
    GOOD: Extract key insights, resolve conflicts, 
          create coherent narrative that addresses original request
    """
    synthesis_request = f"""
    Synthesize these worker results into a single coherent response:
    - Identify common themes
    - Resolve any conflicts between workers
    - Fill gaps where workers didn't fully address the request
    - Present as unified solution, not list of agent outputs
    
    Results: {results}
    """
    # ... coordinator generates synthesis
```

### Issue: Tool Prompts Too Generic

**Solution**: Customize per your stack and workflow

```markdown
# Example: Customize shell-execution.md for your team
ALLOWED COMMANDS:
- npm/pnpm (package management)
- git (version control)
- pytest (testing)
- docker-compose (local services)

BLOCKED COMMANDS:
- sudo (system changes)
- rm -rf (destructive)
- pip install --global (system packages)
- kubectl (production deploys)

GIT WORKFLOW:
1. Create feature branch
2. Make changes
3. Run tests locally
4. Commit with conventional commit message
5. Push and create PR (don't merge directly)
```

## Integration with Other Tools

### With RepoWise

Combine prompts (behavior) with RepoWise (codebase knowledge):

```python
import repowise

# 1. Generate codebase intelligence with RepoWise
repo = repowise.analyze("path/to/repo")
architecture = repo.get_architecture_wiki()
dependencies = repo.get_dependency_graph()

# 2. Inject into agent context
system_prompt = base_system_prompt + f"""

CODEBASE CONTEXT:
{architecture}

DEPENDENCY GRAPH:
{dependencies}

Use this knowledge to understand impact of changes before making them.
"""
```

### With MCP Servers

Use prompts with Model Context Protocol servers:

```python
# Claude Desktop config with claude-code-prompts
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/code"],
      "systemPrompt": "file://path/to/claude-code-prompts/complete-prompts/tool-prompts/file-edit.md"
    }
  }
}
```

## Best Practices

1. **Start with system prompt** — establishes agent identity and core behaviors
2. **Customize for your stack** — replace all `{{PLACEHOLDERS}}` with your specifics
3. **Add tools incrementally** — start with file ops + shell, add others as needed
4. **Implement safety gates** — risk assessment before destructive operations
5. **Use delegation sparingly** — only for truly complex multi-step tasks
6. **Implement memory early** — before context window becomes a problem
7. **Test with verification agent** — use adversarial approach to find bugs
8. **Pair with RepoWise** — give your agent actual codebase understanding

## Related Skills

- [RepoWise](https://github.com/repowise-dev/repowise) — codebase intelligence for AI agents
- [MCP Servers](https://modelcontextprotocol.io/) — Model Context Protocol integrations

---

**Built by the RepoWise team** — [repowise.dev](https://repowise.dev)
