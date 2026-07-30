---
name: claude-code-migration-kit
description: Run large-scale language migrations with Claude Code using structured prompts, dependency mapping, and adversarial review
triggers:
  - migrate codebase to another language
  - run language migration with claude
  - translate code from one language to another
  - port codebase to new language
  - setup code migration workflow
  - create migration rulebook
  - generate dependency map for migration
  - stress test migration rules
---

# Claude Code Migration Kit

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

A structured framework for running large-scale, total language migrations using Claude Code. This kit provides prompts, templates, and scripts for **structure-preserving migrations** (same architecture, new language) where the entire codebase crosses over and the old language is deleted.

## What It Does

- Provides 8 sequenced prompts (feasibility → post-parity) for methodical migration
- Generates dependency maps to order translation work
- Creates rulebooks that decide every translation question once
- Implements adversarial review (implementer + 2 reviewers + fixer per unit)
- Uses parallel agent workflows with explicit sign-off gates
- Validates through parity testing against original codebase

**Core doctrine:** You don't fix the code — you fix the process that produced the code.

## Installation

```bash
# Clone inside or adjacent to the repo you're migrating
git clone https://github.com/anthropics/code-migration-kit-with-claude-code ./migration-kit

# Install as a Claude Code skill (optional)
cp -r migration-kit/skill ~/.claude/skills/code-migration

# Copy the operating manual to target repo
cp migration-kit/CLAUDE.md ./CLAUDE.md

# Create migration workspace
mkdir -p migration
```

## The Six-Step Process

### Step 0: Feasibility Assessment

**Always start here.** Paste `prompts/00-feasibility.md` with placeholders filled:

```markdown
Source language: [Python]
Target language: [Rust]
Repository path: [./src]
Approximate file count: [450]
Primary reason for migration: [memory safety + performance]
```

The feasibility prompt produces:
- Case for staying vs. migrating
- Structure-preserving vs. redesign recommendation
- Verification cost estimate
- Custom six-step sketch for your repo
- Go/no-go verdict

**If redesigning instead of structure-preserving:**
- Rulebook becomes a design document
- Bakeoff step is invalid (use adversarial design review + disposable full runs)
- Unit of work is module/subsystem, not file
- Behavior matching still works unchanged

### Step 0b: Judge Setup (Critical)

**You need a judge before Step 1.** If your test suite imports internals that will die with the old language, run `prompts/00b-judge-setup.md`:

```markdown
I need to build a portable parity harness because our tests import [language]-specific internals.

Source language: [Python]
Target language: [Rust]
Test suite path: [./tests]
Public API surface: [CLI + HTTP API]
```

The judge must be:
- Validated against the original code (zero failures)
- Validated against deliberately broken code (catches intentional bugs)
- Kept running throughout the migration
- Language-agnostic (no imports of source internals)

### Step 1: Create Map and Rules

Three parallel artifacts:

**1. Dependency Map**

```python
# For Python projects
python migration-kit/scripts/depmap_python.py ./src > migration/depmap.json

# For JavaScript/TypeScript
node migration-kit/scripts/depmap_js.mjs ./src > migration/depmap.json

# For C/C++ headers
python migration-kit/scripts/depmap_c_headers.py ./include > migration/depmap.json
```

The map provides:
- File-level dependency ordering (leaves to root)
- Package-level cycle detection
- Translation queue foundation

**2. Rulebook**

```bash
# Copy template
cp migration-kit/templates/RULEBOOK.md ./migration/RULEBOOK.md
```

Use `prompts/01-create-rulebook.md` to draft it. The rulebook decides:
- How each source construct translates to target
- Naming conventions (snake_case → camelCase, etc.)
- Error handling patterns
- Memory management (if applicable)
- Testing approach per translated unit

**Meta-rule:** If two agents could answer differently, it goes in the rulebook.

Example rulebook entry:

```markdown
## Error Handling

**Source (Python):**
```python
def parse_config(path):
    try:
        return json.load(open(path))
    except FileNotFoundError:
        return {}
```

**Target (Rust):**
```rust
fn parse_config(path: &Path) -> Result<Config, ConfigError> {
    let contents = fs::read_to_string(path)
        .map_err(|e| ConfigError::ReadFailed(path.to_owned(), e))?;
    serde_json::from_str(&contents)
        .map_err(ConfigError::ParseFailed)
}
```

**Rule:** All Python exceptions become Result<T, E>. Map errors to domain-specific error types.
```

**3. Gap Inventory**

Use `prompts/02-gap-inventory.md`:

```bash
# Creates migration/inventory.tsv
```

A flat table of every site where the target language demands explicit decisions:
- Ownership annotations
- Lifetime parameters
- Nullability markers
- Interface contracts
- Concurrency primitives

Implementers grep it; nobody reads it cover-to-cover.

**Generate manifest:**

```python
# After dependency map exists
python migration-kit/scripts/make_manifest.py \
  migration/depmap.json \
  > migration/manifest.tsv
```

### Step 2: Stress-Test Rules

**Before any fan-out**, run `prompts/03-stress-test.md`:

**Bakeoff:**
- Two translators in separate contexts
- One follows rulebook, one doesn't know it exists
- Diff inspector turns every difference into a verdict on a rule
- Amendments queued for human approval, never self-applied

**Pilot:**
- Run production pipeline exactly as Step 3 will
- Pick 3-5 nasty files (deepest dependencies, most complex)
- Grade on obedience to rules, not output quality
- Install `.claude/settings.json` BEFORE this pilot:

```bash
cp migration-kit/templates/settings.json ./.claude/settings.json
```

**Critical:** `settings.json` must exist before Step 2 pilot and remain active through Step 4. See `templates/settings.README.md` for the rationale.

### Step 3: Translate Everything

Install queue runner:

```bash
# Make executable
chmod +x migration-kit/scripts/queue_runner.mjs
```

Kick off with `prompts/04-translation-kickoff.md`:

```markdown
Translate the codebase using the established rulebook.

Manifest: migration/manifest.tsv
Rulebook: migration/RULEBOOK.md
Output directory: migration/translated/
Settings: .claude/settings.json (denies active)

For each file in manifest order:
1. Implementer translates (follows rulebook exactly)
2. Adversarial reviewer 1 (checks rule compliance)
3. Adversarial reviewer 2 (checks mistake class X)
4. Fixer (applies amendments)
5. Write to migration/translated/<target_path>
```

**Don't run the compiler yet.** Settings bans:
- Test execution
- Build commands
- File operations outside `migration/translated/`

The queue runner processes `migration/manifest.tsv`:

```javascript
// Queue runner handles resume automatically
// Stop anytime, restart with same command
node migration-kit/scripts/queue_runner.mjs \
  --manifest migration/manifest.tsv \
  --output migration/translated \
  --rulebook migration/RULEBOOK.md
```

### Step 4: Compile

Use `prompts/05-survey-build.md` to run one **survey build**:

```bash
# Start build daemon (human runs once)
./migration-kit/scripts/build_daemon.sh \
  migration/translated \
  "cargo build --all 2>&1" \
  migration/build-output
```

The daemon:
- Watches `migration/translated/`
- Reruns build on changes
- Emits numbered error files: `migration/build-output-r1.txt`, `r2.txt`, etc.
- Slices errors by module (leaves to root)

Fixers work **without compiler access**:

```markdown
Fix compilation errors from migration/build-output-r{N}.txt

Rules:
- No running builds yourself (daemon owns it)
- Read numbered error file
- Fix issues in dependency order
- Write fixes to migration/translated/
- Daemon reruns automatically
- Consume next numbered file

Repeat until clean build.
```

**If target typecheck is cheap (TypeScript, Go):**

This step dissolves into Step 3 — edit `.claude/settings.json` to remove typecheck denies, run typechecker inside each unit's loop instead of batching.

### Step 5: Run It

```bash
# Hello world
./migration/translated/bin/hello

# Smallest end-to-end command
./migration/translated/bin/app --version

# Smoke tests (cheap proofs before expensive ones)
./migration/translated/bin/app test-basic-operation
```

### Step 6: Match Behavior

Your judge from Step 0b decides the gate:

**If tests hit public surface (CLI/API):**

```bash
# Run new tests against new code
pytest migration/translated/tests/

# Triage failures by running against old code
pytest tests/  # Original suite on original code

# Classify: regression / inherited / environment
# Burn down regression queue
```

**If using parity harness:**

```bash
# Run harness against both
python migration/parity_harness.py --target old > old_output.json
python migration/parity_harness.py --target new > new_output.json

# Diff results
diff old_output.json new_output.json
```

**Done gate:**
- Every parity test passes
- Original suite re-run on original code with zero inherited failures
- Both counts documented in final report

**After parity**, use `prompts/06-post-parity.md`:

```markdown
Burn down deferred markers:
- BUG(port): [count from grep]
- TODO(port): [count from grep]
- PERF(port): [count from grep]

Each fix:
- Own flagged commit
- Proved by parity re-run
- Documents why it was deferred
```

## Configuration

**Settings File (`.claude/settings.json`):**

```json
{
  "deny_operations": {
    "test_execution": true,
    "build_commands": true,
    "file_operations_outside_workspace": true
  },
  "migration": {
    "workspace": "migration/translated",
    "rulebook": "migration/RULEBOOK.md",
    "manifest": "migration/manifest.tsv"
  }
}
```

**Timeline:**
- Installed before Step 2 pilot
- Active through Step 4
- Test denies re-activated for Step 6 fix loops
- If Step 4 dissolves into Step 3 (cheap typecheck), remove typecheck denies

## Real Code Example: Python → Rust Migration

**Original Python (src/parser.py):**

```python
class ConfigParser:
    def __init__(self, path):
        self.path = path
        self.data = {}
    
    def load(self):
        try:
            with open(self.path) as f:
                self.data = json.load(f)
        except FileNotFoundError:
            self.data = {"default": True}
        return self.data
    
    def get(self, key, default=None):
        return self.data.get(key, default)
```

**Rulebook Entry:**

```markdown
## Class Translation

**Python classes → Rust structs + impl blocks**
- `__init__` → `new()` constructor
- Instance methods → `&self` methods
- Exceptions → Result<T, E>
- Optional args → Option<T>
```

**Translated Rust (migration/translated/src/parser.rs):**

```rust
use std::collections::HashMap;
use std::fs;
use std::path::{Path, PathBuf};
use serde_json;

#[derive(Debug)]
pub enum ConfigError {
    ReadFailed(PathBuf, std::io::Error),
    ParseFailed(serde_json::Error),
}

pub struct ConfigParser {
    path: PathBuf,
    data: HashMap<String, serde_json::Value>,
}

impl ConfigParser {
    pub fn new(path: impl AsRef<Path>) -> Self {
        Self {
            path: path.as_ref().to_owned(),
            data: HashMap::new(),
        }
    }
    
    pub fn load(&mut self) -> Result<&HashMap<String, serde_json::Value>, ConfigError> {
        match fs::read_to_string(&self.path) {
            Ok(contents) => {
                self.data = serde_json::from_str(&contents)
                    .map_err(ConfigError::ParseFailed)?;
            }
            Err(e) if e.kind() == std::io::ErrorKind::NotFound => {
                self.data.insert("default".to_string(), serde_json::json!(true));
            }
            Err(e) => return Err(ConfigError::ReadFailed(self.path.clone(), e)),
        }
        Ok(&self.data)
    }
    
    pub fn get(&self, key: &str) -> Option<&serde_json::Value> {
        self.data.get(key)
    }
}
```

## Common Patterns

### Resumable Workflows

Every queue is defined by what exists on disk:

```bash
# Stop anytime (Ctrl+C)
# Resume by re-running same command
node migration-kit/scripts/queue_runner.mjs --manifest migration/manifest.tsv
```

Stopping is free. Resuming is re-invocation, not recovery.

### Sign-Off Gates

Prompts end with gates, not auto-continue:

```markdown
=== GATE ===
Sign-off required to proceed to Step 3.

Evidence:
- Bakeoff diff: migration/bakeoff-diff.md
- Pilot results: migration/pilot-results.md
- Proposed amendments: migration/rule-amendments.md

Your approval kicks off Step 3 translation fan-out.
```

**Your sign-off = starting the next prompt.**

### Adversarial Review

Each unit gets 3 reviewers with different mandates:

```python
# In translation loop
reviewers = [
    {"role": "rule_compliance", "rejects_on": "any rulebook deviation"},
    {"role": "safety", "rejects_on": "memory unsafety, data races"},
    {"role": "performance", "rejects_on": "allocations in hot path"},
]
```

### Dependency Map Usage

```python
# Check if file is ready to translate
import json

depmap = json.load(open("migration/depmap.json"))
file_deps = depmap["files"]["src/module.py"]["dependencies"]

all_translated = all(
    os.path.exists(f"migration/translated/{dep}")
    for dep in file_deps
)
```

## Troubleshooting

### "Feasibility prompt says don't migrate"

**Don't migrate.** Valid outcomes:
- Stay on current language
- Incremental adoption (TypeScript superset model)
- Rewrite from scratch (if redesigning heavily)

The kit is for total, structure-preserving migrations. If that's not your case, the ROI isn't there.

### "Tests fail but they also fail on original code"

These are **inherited failures**, not regressions:

```bash
# Classify each failure
pytest tests/test_config.py::test_parse  # On new code: FAIL
pytest tests/test_config.py::test_parse  # On old code: FAIL → INHERITED

# Document in migration/inherited-failures.md
# Don't block parity gate on these
```

The done-gate explicitly requires: "original suite re-run on original code with zero inherited failures."

### "Parity harness reports all divergences"

**Debug the referee first.** In early testing, all reported divergences traced to comparator bugs (whitespace handling, JSON.stringify converting NaN to null), not the port.

```python
# Validate referee against deliberately broken code
# BEFORE trusting its verdicts
def test_referee_catches_bugs():
    broken_impl = introduce_bug(correct_impl)
    assert referee.compare(broken_impl, correct_impl) != "PASS"
```

### "Build errors repeat across modules"

**Indict the rule, not the code:**

```markdown
Recurring error: "lifetime parameter required but not in rulebook"

Action:
1. Add to migration/rule-amendments.md
2. Update RULEBOOK.md with lifetime rules
3. Re-run affected translations (grep manifest for pattern)
4. DO NOT fix individual files without fixing the rule
```

### "Dependency map has cycles"

File-level cycles are rare but package-level cycles are common:

```bash
# Check both levels
python migration-kit/scripts/depmap_python.py ./src --check-cycles

# Output shows:
# File cycles: 0
# Package cycles: 3 (details in depmap.json)
```

**For package cycles:** Break them in the target language's module system first (Rust: pub(crate), careful re-exports), then translate.

### "Queue runner stuck on one file"

```bash
# Check migration/queue_runner.log
tail -f migration/queue_runner.log

# Skip problematic file temporarily
echo "src/broken.py" >> migration/skip_list.txt
node migration-kit/scripts/queue_runner.mjs --skip-list migration/skip_list.txt

# Come back to it after understanding the pattern
```

### "Settings.json never installed, Step 3 ran unbounded"

This happened in early testing. The dissolve (removing typecheck denies) silently proceeded with no guardrails.

**Fix:**

```bash
# Install now
cp migration-kit/templates/settings.json ./.claude/settings.json

# Re-run Step 2 pilot to validate denies work
# Then resume Step 3
```

**Prevention:** Prompts 03 and 04 now verify settings.json exists and stop without it.

## When NOT to Use This Kit

- **Incremental migrations** (JavaScript → TypeScript): Target is a superset, adopt file-by-file
- **Greenfield rewrites**: No existing codebase to preserve structure from
- **Small codebases**: < 50 files, manual translation is faster
- **Heavy redesign**: If you're changing architecture radically, parts of this kit become invalid (bakeoff, file-level units)

Use this kit when:
- The move is total (old language gets deleted)
- Every file must cross
- You're preserving structure (same data flow, new syntax)
- Scale justifies automation (100+ files)

## Environment Variables

Scripts expect:

```bash
export MIGRATION_WORKSPACE=./migration
export SOURCE_ROOT=./src
export TARGET_ROOT=./migration/translated
export CLAUDE_SETTINGS=./.claude/settings.json
```

No API keys needed — this kit orchestrates Claude Code workflows via prompts, not API calls.
