---
name: claude-code-academic-workflow
description: AI-assisted academic workflow with LaTeX/Beamer, R, Quarto, multi-agent review, quality gates, and reproducibility protocols
triggers:
  - set up academic workflow for slides and papers
  - create lecture slides with quality review
  - review my research paper with adversarial QA
  - verify reproducibility of my analysis
  - run multi-agent review on my manuscript
  - check my Beamer slides for quality
  - compile and review my academic presentation
  - audit my paper for fabricated claims
---

# claude-code-academic-workflow

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## What This Is

A production-ready template for AI-assisted academic work using Claude Code. Provides specialized agents for reviewing slides, papers, and data analysis; quality scoring with 80/90/95 thresholds; adversarial QA loops; and reproducibility verification. Originally extracted from a PhD course, now community-maintained with 1200+ GitHub stars.

**Core capabilities:**
- **Contractor mode** — describe what you want, Claude plans and orchestrates specialized agents
- **18+ focused agents** — proofreader, slide-auditor, pedagogy-reviewer, r-reviewer, domain-referee, methods-referee, editor
- **Quality gates** — scores 0-100, auto-blocks commits below 80
- **Adversarial QA** — critic + fixer loop until approval or 5 rounds
- **Verification layers** — claim verification, reproducibility audit, AI-voice detection, multi-referee variance sampling

**Supported artifacts:**
- LaTeX/Beamer slides (XeLaTeX)
- Research manuscripts (LaTeX)
- Quarto documents (HTML/PDF)
- R analysis scripts
- Data replication packages

## Installation

### 1. Fork and Clone

```bash
# Fork on GitHub first, then:
git clone https://github.com/YOUR_USERNAME/claude-code-my-workflow.git my-project
cd my-project
```

### 2. Validate Prerequisites

```bash
./scripts/validate-setup.sh
```

**Minimum requirements:**
- Claude Code (CLI or VS Code extension)
- git
- Python 3 (pre-installed macOS/Linux; validation scripts only)

**For LaTeX/Beamer demos:**
- XeLaTeX (TeX Live 2020+)

**For Quarto demos:**
- Quarto 1.3+

**For R analysis:**
- R 4.0+
- Required packages: `tidyverse`, `fixest`, `modelsummary`

**Recommended:**
- GitHub CLI (`gh`)
- VS Code with Claude Code extension

### 3. Initialize with Starter Prompt

Start Claude Code and paste:

```text
I am starting to work on [PROJECT NAME] in this repo. [Describe project in 2-3 sentences.] 
I've set up the Claude Code academic workflow from pedrohcgs/claude-code-my-workflow. 
Please read the configuration files (.claude/CLAUDE.md, .claude/rules/*.md, .claude/skills/*.md) 
and adapt them for my project. Set my name to [YOUR NAME], institution to [YOUR INSTITUTION], 
and configure the project metadata. Enter plan mode and start.
```

Claude will:
1. Read all configuration files
2. Update project metadata in `.claude/project.yaml`
3. Verify setup with `/validate-setup`
4. Enter contractor mode for your first task

## Key Commands (Skills)

### Slide Creation & Review

**Create lecture slides:**
```text
/create-lecture "Introduction to Panel Data"
```

**Compile LaTeX to PDF:**
```text
/compile-latex HelloWorld
```

**Run quality review (slide-specific agents):**
```text
/slide-excellence Slides/MyLecture.tex
```

**Adversarial QA loop:**
```text
/qa-beamer Slides/MyLecture.tex --rounds 5
```

### Paper Review & Verification

**Multi-agent peer review:**
```text
/review-paper manuscript.tex --peer
```

**Adversarial review:**
```text
/review-paper manuscript.tex --adversarial
```

**Multi-referee variance sampling (v1.9.0):**
```text
/review-paper manuscript.tex --variance 10
```
Runs 10 referees with sampled dispositions, reports decision distribution.

**Verify claims against fabrication:**
```text
/verify-claims manuscript.tex
```
Chain-of-Verification with forked verifier. HIGH-WARN findings block `/commit`.

**Audit reproducibility:**
```text
/audit-reproducibility manuscript.tex
```
Cross-checks every numeric claim against script output. Generates `passport.yaml` with PASS/FAIL/STALE status per claim.

**Detect AI-voice patterns:**
```text
/humanize manuscript.tex
```
Read-only detection of boilerplate, hedging, sycophancy. Does NOT auto-rewrite.

### Quarto Workflows

**Deploy Quarto document:**
```text
/deploy HelloWorld
```

**QA Quarto document:**
```text
/qa-quarto Quarto/MyDoc.qmd
```

### R Analysis Review

**Review R script quality:**
```text
/review-r analysis/main.R
```

**Literature review for R packages:**
```text
/lit-review "fixest two-way fixed effects"
```

### Quality & Reproducibility

**Commit with quality gates:**
```text
/commit "Add panel data lecture" --require-score 80
```
Blocks if quality score < 80.

**Create pull request:**
```text
/pr "Manuscript ready for review" --require-score 90
```
Blocks if quality score < 90.

**Compress long session:**
```text
/compress-session
```
Distills conversation to structured note before auto-compaction. Preserves decisions, next actions, marks noise.

**Promote learnings to shared memory:**
```text
/promote-memory
```
Five-critic council reviews personal-memory.md, promotes generic patterns to MEMORY.md.

### Model & Cost Management

**Set effort level:**
```text
/effort high
```
Options: `low`, `medium`, `high`, `xhigh`, `max`. Opus 4.8 defaults to `high`.

**Check usage:**
```text
/cost
/usage
```

**Goal-driven persistence (v1.9.0):**
```text
/goal "all tests pass and manuscript score > 90"
```
Keeps working across turns until fast model confirms condition.

## Configuration Files

### Project Metadata

`.claude/project.yaml`:
```yaml
project:
  name: "My Research Project"
  author: "Your Name"
  institution: "Your University"
  course: "ECON 5000"  # or null for papers
  
quality:
  commit_threshold: 80
  pr_threshold: 90
  excellence_threshold: 95
  
paths:
  slides: "Slides"
  quarto: "Quarto"
  data: "Data"
  analysis: "R"
```

### Agent Configuration

`.claude/agents/` directory contains 18+ agent specs:

**Example — R Reviewer (`.claude/agents/r-reviewer.md`):**
```markdown
---
agent: r-reviewer
focus: R code quality and best practices
model: sonnet-4.6
effort: medium
---

# R Code Reviewer

## Responsibilities
- tidyverse idiom compliance
- fixest usage patterns
- modelsummary output quality
- data.table efficiency
- statistical correctness

## Review Checklist
- [ ] No base R subsetting in tidyverse pipelines
- [ ] Fixed effects specified correctly
- [ ] Standard errors clustered appropriately
- [ ] Tables formatted for publication
- [ ] Code reproducible from raw data
```

### Skills

Skills are in `.claude/skills/`. Frontmatter specifies triggers, effort, context, hooks:

**Example — `/create-lecture`:**
```markdown
---
skill: create-lecture
description: Create Beamer lecture slides with quality review
triggers:
  - create lecture on
  - make slides about
  - build presentation for
effort: high
context: fork
hooks:
  - PreEdit
  - PostEdit
  - PreCompact
---

# Create Lecture

(implementation follows)
```

### Rules

`.claude/rules/` directory:

- `core-principles.md` — contractor mode, quality thresholds, verification discipline
- `latex-rules.md` — XeLaTeX compilation, Beamer best practices, TikZ prevention
- `quarto-rules.md` — rendering, deployment, cross-referencing
- `r-rules.md` — tidyverse style, fixest patterns, reproducibility
- `meta-governance.md` — MEMORY.md vs personal-memory.md distinction

### Hooks

`.claude/hooks/` directory enables event-driven workflows:

**Example — PreCompact hook (`.claude/hooks/pre-compact.yaml`):**
```yaml
name: pre-compact
event: PreCompact
handler:
  type: command
  command: ./scripts/save-context-snapshot.sh
description: Save context before auto-compression
```

## Real Code Examples

### Creating Lecture Slides

```text
User: Create a lecture on difference-in-differences with 2x2 design

Claude: [Enters plan mode]

PLAN:
1. Create Slides/DID-2x2.tex with Beamer template
2. Sections: Motivation, Framework, Identification, Estimation, Inference
3. Include numerical example with treatment/control, pre/post
4. Add R code snippet showing fixest::feols()
5. Run /slide-excellence for quality review
6. Run /qa-beamer for adversarial QA
7. Compile to PDF
8. Score and report

[Implements → reviews → fixes → scores 87 → presents summary]

User: just do it

Claude: [Runs /commit with quality gates passed]
```

### Reproducibility Audit

```text
User: /audit-reproducibility paper.tex

Claude: [Scans paper for numeric claims]

Found 47 claims:
- Table 2, Row 1, Col 2: "0.342 (0.089)" → R/analysis.R:156
- Table 3, Row 4, Col 1: "-1.23" → R/robustness.R:89
- Text line 234: "23% increase" → R/summary-stats.R:45

[Executes scripts, compares output]

FINDINGS:
✓ 44/47 claims verified
⚠ 2 STALE (script modified after paper)
✗ 1 FAIL (Table 3 Row 4: script produces -1.19, paper says -1.23)

passport.yaml updated. Quality score: 72 (below commit threshold).
```

### Multi-Referee Variance

```text
User: /review-paper paper.tex --variance 10

Claude: [Samples 10 referee dispositions, runs domain-referee + methods-referee + editor]

DECISION DISTRIBUTION (N=10):
- Accept: 2
- Minor Revision: 5
- Major Revision: 2
- Reject: 1

VARIANCE ANALYSIS:
- Methodological rigor concerns consistent across 8/10
- Contribution strength varies with reviewer disposition (ρ=0.83)
- Decision highly sensitive to prior belief about external validity

REPRESENTATIVE REVIEWS:
[Shows accept, minor, major, reject reviews with sampled contexts]

Recommendation: Address methodological concerns before submission.
```

### R Code Review

```text
User: /review-r analysis/main.R

Claude: [r-reviewer agent activated]

FINDINGS:
✓ tidyverse idioms correct
✓ fixest models specified properly
✗ Line 45: base R subsetting in dplyr pipeline
  Replace: df[df$year > 2010, ]
  With:    df %>% filter(year > 2010)
  
✗ Line 89: standard errors not clustered
  Add: , vcov = ~state

⚠ Line 134: modelsummary() missing stars_note = FALSE for publication

SCORE: 78/100 (below commit threshold)

[Presents fix plan]
```

### Claim Verification

```text
User: /verify-claims manuscript.tex

Claude: [Forked verifier with no draft access]

CHAIN-OF-VERIFICATION:

Claim 1: "Smith et al. (2023) find a 15% reduction"
- Citation exists: ✓
- Numerical match: ✗ (paper reports 12%, not 15%)
- Severity: HIGH-WARN

Claim 2: "Table 2 shows no significant effect"
- Cross-check with Table 2: ✓
- p-value > 0.05: ✓
- Severity: PASS

Claim 3: "consistent with Jones (2022)"
- Citation exists: ✓
- Conceptual alignment: ✓ (verified via abstract)
- Severity: PASS

Claim 4: "as discussed in Section 3"
- Internal reference exists: ✗ (no Section 3 discussion of this)
- Severity: MED-WARN

RESULT: 1 HIGH-WARN finding blocks /commit. Fix Claim 1 before proceeding.
```

## Common Patterns

### Session 1: Project Setup

```text
1. Clone repo
2. Run ./scripts/validate-setup.sh
3. Start Claude, paste starter prompt
4. Describe first task (lecture or paper)
5. Approve plan
6. Let contractor mode orchestrate
7. Say "just do it" to auto-commit when score > 80
```

### Slide Creation Workflow

```text
/create-lecture "Topic Name"
  → Claude creates .tex, compiles, runs /slide-excellence, /qa-beamer
  → Reports score
  → You review findings or say "just do it"
  → Auto-commits if score ≥ 80
```

### Paper Submission Workflow

```text
1. /review-paper manuscript.tex --peer
   → domain-referee + methods-referee + editor in parallel
2. /verify-claims manuscript.tex
   → Catches fabrications
3. /audit-reproducibility manuscript.tex
   → Verifies all numbers against scripts
4. /humanize manuscript.tex
   → Flags AI-voice patterns (read-only)
5. Address findings
6. /review-paper manuscript.tex --variance 10
   → Sample decision distribution
7. /pr "Ready for submission" --require-score 90
```

### Replication Package

```text
1. /audit-reproducibility manuscript.tex
   → Generates passport.yaml with claim→script mappings
2. Review STALE/FAIL findings
3. Fix scripts or manuscript
4. Re-run until all PASS
5. /commit "Replication package verified"
```

## Troubleshooting

### XeLaTeX Compilation Fails

**Symptom:** `/compile-latex` errors with font not found

**Fix:**
```bash
# Rebuild font cache
sudo fc-cache -fv

# Or install missing fonts (example for Linux Libertine)
tlmgr install libertine
```

### Quality Score Below Threshold

**Symptom:** "Score 76 below commit threshold 80"

**Fix:**
```text
# Review findings
Show me the detailed findings from the last review

# Fix specific issues or override
/commit "Draft WIP" --override-score
```

### Reproducibility Audit STALE Warnings

**Symptom:** "Script modified after paper written"

**Fix:**
```bash
# Re-run analysis
Rscript R/analysis.R

# Re-audit
/audit-reproducibility manuscript.tex
```

### Prompt Cache Miss

**Symptom:** High cost for repeated operations

**Fix:**
```text
# Check cache TTL (5-min API default, 1-hour on subscriptions)
/usage

# Batch related work to leverage cache
/create-lecture "Lecture 1"
/create-lecture "Lecture 2"  # reuses cached rules
```

### Agent Parallel Review Hangs

**Symptom:** `/review-paper --peer` stalls

**Fix:**
```bash
# Monitor agent dashboard (v1.9.0+)
claude agents

# Or run sequentially
/review-paper manuscript.tex --peer --sequential
```

### Memory Overflow on Long Sessions

**Symptom:** Context full, losing decisions

**Fix:**
```text
# Manual compression before auto-compact
/compress-session

# Or promote learnings early
/promote-memory
```

### HIGH-WARN Blocking Commit

**Symptom:** `/verify-claims` found fabricated citation, `/commit` blocked

**Fix:**
```text
# Review specific finding
Show the HIGH-WARN finding details

# Fix the claim in manuscript
# Re-verify
/verify-claims manuscript.tex

# Commit once clear
/commit "Claims verified"
```

## Environment Variables

No secrets required for core workflow. Optional for extensions:

```bash
# GitHub CLI (for /pr skill)
export GH_TOKEN="your_github_token"

# Anthropic API key (if using API directly, not Claude Code app)
export ANTHROPIC_API_KEY="your_api_key"

# R package library (optional custom path)
export R_LIBS_USER="~/R/library"
```

## Version Compatibility

- **Minimum Claude Code:** 1.9.0 (for `/goal`, `claude agents`, `/compress-session`)
- **Recommended models:** Sonnet 4.6 (workhorse), Opus 4.8 (complex reasoning)
- **Effort defaults:** Opus 4.8 uses `high` by default (equivalent to prior `xhigh`)
- **Permission modes:** Auto mode requires Team/Enterprise/API + Opus 4.6+/Sonnet 4.6

## Learn More

- **Full guide:** [psantanna.com/claude-code-my-workflow](https://psantanna.com/claude-code-my-workflow/)
- **GitHub:** [pedrohcgs/claude-code-my-workflow](https://github.com/pedrohcgs/claude-code-my-workflow)
- **CHANGELOG:** See `CHANGELOG.md` in repo for version history
- **Community extensions:** MixtapeTools, autoresearch, ClaudeCodeTools (see guide § The Ecosystem)
