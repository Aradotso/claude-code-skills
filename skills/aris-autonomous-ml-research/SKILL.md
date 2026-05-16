---
name: aris-autonomous-ml-research
description: Use ARIS (Auto-Research-In-Sleep) for autonomous ML research — idea generation, paper review, experiment automation, and cross-model collaboration with Claude Code, Codex, or any LLM agent.
triggers:
  - set up ARIS for autonomous research
  - use ARIS to generate research ideas
  - run autonomous ML experiments with ARIS
  - review a paper using ARIS cross-model workflow
  - configure ARIS research pipeline
  - help me with ARIS research automation
  - write a rebuttal using ARIS
  - start an ARIS research workflow
---

# ARIS Autonomous ML Research

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

ARIS (Auto-Research-In-Sleep) is a lightweight Markdown-based system for autonomous ML research that orchestrates cross-model collaboration. It enables AI agents to discover ideas, review papers, run experiments, and write rebuttals — all autonomously. Works with Claude Code, Codex CLI, Cursor, Trae, Antigravity, or any LLM agent.

## What ARIS Does

- **Idea Generation**: Automatically discovers research ideas from arXiv papers, GitHub repos, or research directions
- **Cross-Model Review**: Uses different LLMs for execution vs. review to break self-play blind spots (e.g., Claude Code executes, GPT-5.4 reviews)
- **Experiment Automation**: Clones codebases, runs experiments, analyzes results
- **Paper Writing**: Generates drafts with auto-review loops to improve quality
- **Rebuttal Generation**: Parses reviews, builds strategy, drafts rebuttals under character limits
- **Research Wiki**: Persistent knowledge base tracking papers, ideas, experiments, and claims

## Installation

### As Claude Code Skills (In-Editor)

```bash
# Clone skills into your Claude Code skills directory
cd ~/claude-code-skills  # or your skills path
git clone https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep.git aris
cd aris/skills

# Skills are now available in Claude Code
```

### As Standalone CLI (ARIS-Code)

```bash
# Download latest release
# Visit: https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep/releases/latest

# Or install from source
git clone https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep.git
cd Auto-claude-code-research-in-sleep/aris-code
cargo build --release

# Run setup
./target/release/aris-code
# Follow interactive setup to configure API keys
```

### Environment Setup

ARIS requires API keys for LLM providers:

```bash
# For Claude (primary executor)
export ANTHROPIC_API_KEY=your_key_here

# For OpenAI (reviewer/alternative)
export OPENAI_API_KEY=your_key_here

# For alternative Chinese models (optional)
export MOONSHOT_API_KEY=your_key_here      # Kimi
export MINIMAX_API_KEY=your_key_here       # MiniMax
export GLM_API_KEY=your_key_here           # GLM
export DEEPSEEK_API_KEY=your_key_here      # DeepSeek
export DOUBAO_API_KEY=your_key_here        # Doubao
```

## Key Commands

### Standalone CLI Commands

```bash
# Interactive setup
/setup

# Run full research pipeline
/research-pipeline "your research direction"

# Run with reference paper and base repo
/research-pipeline "improve method X" -- ref paper: https://arxiv.org/abs/2406.04329, base repo: https://github.com/org/project

# Generate rebuttal from reviews
/rebuttal "paper/ + reviews" -- venue: ICML, character limit: 5000

# Plan mode (step-by-step breakdown)
/plan "implement transformer variant"

# Research Wiki operations
/wiki add paper <arxiv_url>
/wiki add idea "your idea description"
/wiki query "search term"
/wiki export

# Meta-optimization (self-improvement)
/meta-optimize

# Task management
/tasks
/tasks add "task description"
/tasks complete <id>

# Help and info
/help
/models  # List available models
```

### Claude Code Skill Commands

When using ARIS as Claude Code skills, trigger workflows with natural language:

```
"Use ARIS to review this paper: https://arxiv.org/abs/2406.04329"
"Generate research ideas about discrete diffusion models"
"Run the research pipeline for improving attention mechanisms"
"Help me write a rebuttal for these ICML reviews"
```

## Core Workflows

### 1. Full Research Pipeline

End-to-end autonomous research:

```bash
/research-pipeline "factorized gap in discrete diffusion LMs"
```

**What happens:**
1. Discovers related papers from arXiv
2. Generates novel research ideas
3. Reviews ideas with external LLM (cross-model)
4. Runs experiments on top ideas
5. Writes paper draft
6. Auto-reviews and improves draft
7. Outputs final paper + code

**With reference paper + codebase:**

```bash
/research-pipeline "improve attention efficiency" -- ref paper: https://arxiv.org/abs/2305.xxxx, base repo: https://github.com/org/attention-impl
```

ARIS reads the paper → finds weaknesses → uses that specific codebase → generates targeted improvements.

### 2. Targeted Idea Discovery

Generate ideas from specific sources:

```python
# In Claude Code, reference the skill
"""
Use Workflow 1: DiscoverPaper skill to find ideas from:
- arXiv search: "vision transformers"
- GitHub repo: https://github.com/google-research/vision_transformer
- Local paper: ./papers/vit_analysis.pdf
"""
```

**ARIS will:**
- Parse papers/code
- Extract key insights
- Generate 5-10 novel ideas
- Route to external reviewer
- Return scored, critiqued ideas

### 3. Paper Review Loop

Multi-round review with automated improvements:

```bash
# Standalone CLI
/review-paper paper_draft.md --rounds 3

# In Claude Code
"Review this draft and improve it through 3 rounds: ./draft.md"
```

**Review process:**
1. External LLM critiques (e.g., GPT-5.4)
2. Claude Code addresses weaknesses
3. Repeat until score plateau or max rounds
4. Final output with score progression graph

### 4. Rebuttal Generation

Parse reviews and draft rebuttal:

```bash
/rebuttal "paper_dir/" -- venue: ICML, character limit: 5000
```

**Phases:**
1. **Parse reviews**: Extract all reviewer concerns
2. **Build strategy**: Map concerns → responses
3. **Draft rebuttal**: Generate structured response
4. **Format check**: Ensure under character limit

**Quick mode** (stop before drafting):

```bash
/rebuttal "paper_dir/" -- venue: NeurIPS, character limit: 8000, quick mode: true
```

## Configuration

### Model Selection

ARIS supports multiple executor + reviewer combinations:

**In standalone CLI:**

```bash
# Interactive setup
/setup
# Choose:
# 1. Claude (Anthropic)
# 2. OpenAI (GPT-4/5)
# 3. Kimi (Moonshot)
# 4. MiniMax
# 5. GLM (Zhipu)
# 6. DeepSeek
# 7. Doubao
# 8. LM Studio (local)
# ... and more
```

**In skill files (YAML frontmatter):**

```yaml
---
executor: claude-opus-4.7  # Primary LLM
reviewer: gpt-5.5          # Review LLM
---
```

**Available executors:**
- `claude-opus-4.7`, `claude-sonnet-4.5`
- `gpt-5.5`, `gpt-5.4`, `o1`, `o3`, `o4`
- `kimi-k2.5`, `kimi-k3`
- `minimax-m2.7`, `minimax-pro`
- `glm-5`, `glm-5-plus`
- `deepseek-v3`, `deepseek-r1`
- `doubao-lite`, `doubao-pro`

**Reviewer routing:**

```yaml
reviewer: oracle-pro  # GPT-5.4 Pro via Oracle MCP (strongest)
reviewer: gpt-5.4     # GPT-5.4 standard
reviewer: claude-opus # Claude for review
reviewer: auto        # Smart routing based on executor
```

### Research Wiki Configuration

Enable persistent memory across sessions:

```bash
# In CLI
/setup
# Enable "Research Wiki" option

# Or set in config
wiki:
  enabled: true
  path: ~/.aris/wiki/
  auto_commit: true  # Git commit after each change
```

**Wiki structure:**

```
~/.aris/wiki/
├── papers/         # Tracked papers
├── ideas/          # Research ideas
├── experiments/    # Experiment results
├── claims/         # Key claims and evidence
└── graph.json      # Relationship graph
```

### Proxy and Custom Endpoints

**HTTP/HTTPS proxy:**

```bash
export HTTP_PROXY=http://localhost:8080
export HTTPS_PROXY=http://localhost:8080

# Or in /setup
# Select provider → Configure proxy URL
```

**Custom API endpoints:**

```bash
# For Anthropic-compatible proxies (Bedrock, etc.)
/setup
# Provider: Anthropic
# Custom base URL: https://bedrock.amazonaws.com/anthropic/
```

**Local models (LM Studio/Ollama):**

```bash
/setup
# Select "LM Studio / Ollama (Local)"
# Base URL: http://localhost:1234/v1
# Model: local-model-name
```

## Code Examples

### Example 1: Idea Discovery from Paper

```python
# skill: workflow-1-discover-paper.md
"""
I want to discover ideas from this paper:
https://arxiv.org/abs/2305.14342 (Transformer-XL)

Focus on: attention mechanism improvements
"""

# ARIS will:
# 1. Download and parse paper
# 2. Extract key insights about XL attention
# 3. Generate 5-10 novel ideas
# 4. Send to GPT-5.4 for review
# 5. Return scored ideas with critiques
```

**Output structure:**

```markdown
## Discovered Ideas (Reviewed)

### Idea 1: Factorized Relative Position Embeddings (Score: 8.5/10)
**Core insight:** XL uses dense relative position matrix — factorize it.

**Reviewer critique (GPT-5.4):**
- ✅ Novelty: High (not explored in XL paper)
- ✅ Feasibility: Doable (standard tensor decomposition)
- ⚠️  Impact: Need to verify on long sequences
- ⚠️  Risk: May hurt performance if rank too low

**Next steps:** Implement SVD-based factorization, benchmark on PG-19

---

### Idea 2: Learnable Decay for Relative Attention (Score: 7.2/10)
...
```

### Example 2: Experiment Automation

```python
# skill: workflow-2-run-experiment.md
"""
Clone https://github.com/kimiyoung/transformer-xl
Implement Idea 1 (factorized position embeddings)
Run on enwik8 benchmark
Compare with baseline
"""

# ARIS will:
# 1. Clone repo
# 2. Create experiment branch
# 3. Modify model code (e.g., pytorch_modules/rel_multihead_attn.py)
# 4. Set up training config
# 5. Run experiment
# 6. Parse results
# 7. Generate comparison report
```

**Generated experiment code:**

```python
# aris_experiments/factorized_rel_pos/model_patch.py
import torch
import torch.nn as nn

class FactorizedRelativeAttention(nn.Module):
    def __init__(self, d_model, n_heads, rank=64):
        super().__init__()
        self.d_model = d_model
        self.n_heads = n_heads
        self.rank = rank
        
        # Factorized position embeddings: (seq_len, d_model) ≈ U @ V^T
        self.U = nn.Parameter(torch.randn(2048, rank))  # max_seq_len
        self.V = nn.Parameter(torch.randn(d_model, rank))
    
    def forward(self, q, k, v, pos_emb):
        # Compute relative position on-the-fly
        rel_pos = self.U @ self.V.t()  # (2048, d_model)
        # ... rest of attention logic
```

**Experiment results:**

```markdown
## Experiment Results: Factorized Relative Position

| Metric | Baseline (XL) | Ours (Factorized) | Δ |
|--------|---------------|-------------------|---|
| PPL (enwik8) | 1.06 | 1.08 | +0.02 ↓ |
| Speed (tok/s) | 12.3k | 18.7k | +52% ↑ |
| Memory (GB) | 11.2 | 7.8 | -30% ↑ |
| Params (M) | 277 | 261 | -5.8% ↑ |

✅ **Success**: 52% faster, 30% less memory, minor PPL degradation acceptable for long-context tasks.
```

### Example 3: Paper Writing with Auto-Review

```python
# skill: workflow-3-write-paper.md
"""
Write a paper about the factorized position embeddings experiment.

Title: "Efficient Transformers via Factorized Relative Attention"
Target venue: ICML 2026
Sections: Abstract, Introduction, Method, Experiments, Conclusion
"""

# ARIS will:
# 1. Generate initial draft
# 2. Send to GPT-5.4 for review
# 3. Address critiques (e.g., "add ablation study")
# 4. Re-review
# 5. Repeat for N rounds or until score plateau
```

**Review loop:**

```markdown
## Round 1 Review (GPT-5.4, Score: 6.5/10)

**Strengths:**
- Clear motivation (long-context efficiency)
- Solid experimental results

**Weaknesses:**
- Missing ablation on rank hyperparameter
- No comparison with Linear Attention baseline
- Introduction lacks related work on efficient Transformers

**Suggestions:**
1. Add Table 2: Rank ablation (r=16,32,64,128)
2. Cite Performer, Linformer in related work
3. Add wall-clock time comparison

---

## Round 2 Review (GPT-5.4, Score: 8.1/10)

**Improvements:**
✅ Added rank ablation (Table 2)
✅ Expanded related work
✅ Wall-clock benchmarks included

**Remaining issues:**
- Figure 3 caption unclear
- Conclusion should mention future work

---

## Round 3 Review (GPT-5.4, Score: 8.8/10)

**Near-ready:** Minor edits only. Ready for submission.
```

### Example 4: Rebuttal Generation

```bash
# reviews.txt contains reviewer comments
# paper/ directory has the submitted paper

/rebuttal "paper/ reviews.txt" -- venue: ICML, character limit: 5000
```

**Generated rebuttal structure:**

```markdown
# Rebuttal to ICML Reviews

**Character count: 4847 / 5000**

## Response to Reviewer 1 (Score: 6 → ?)

**Q1: "Rank ablation missing — how does r affect performance?"**

We thank the reviewer for this suggestion. We have added Table 2 (Appendix) showing rank ablation (r=16,32,64,128). Key findings: r=64 is optimal (PPL 1.08, speed +52%), r=128 matches baseline quality (PPL 1.06) but slower, r=32 degrades PPL to 1.15. We will include this in the camera-ready.

**Q2: "No comparison with Linear Attention methods."**

Valid point. We added Performer and FNet baselines (Table 3). Our method outperforms Performer by 0.04 PPL while being 20% faster due to factorization locality. Updated draft attached.

---

## Response to Reviewer 2 (Score: 7 → ?)

**Q1: "Scalability to 100k+ sequences unclear."**

We have run additional experiments on PG-19 (100k context). Results: our method maintains +45% speedup with PPL degradation <0.05. Memory scales O(r·L) vs O(L²) for standard XL. Will add to Section 4.3.

---

**Common concern (R1, R3): "Related work incomplete."**

We have expanded Section 2 to cite Linformer, Performer, Longformer, and BigBird. Table 1 now includes complexity comparison.

---

**Summary:** We address all major concerns with new experiments and expanded analysis. We believe these changes strengthen the paper significantly.
```

## Common Patterns

### Pattern 1: Iterative Improvement Loop

```python
# Discover → Review → Refine → Repeat
ideas = discover_papers("diffusion models")
reviewed_ideas = cross_model_review(ideas)
top_idea = select_highest_score(reviewed_ideas)
results = run_experiment(top_idea)
paper = write_paper(results)
final_paper = auto_review_loop(paper, rounds=3)
```

### Pattern 2: Multi-Source Idea Generation

```python
# Combine arXiv + GitHub + local papers
sources = [
    "arxiv:diffusion+models",
    "github:CompVis/stable-diffusion",
    "local:./papers/ddpm_analysis.pdf"
]
ideas = discover_from_sources(sources)
ideas = deduplicate_ideas(ideas)
ideas = cross_review(ideas)
```

### Pattern 3: Targeted Paper Improvement

```python
# Read existing paper → find gaps → generate fixes
paper_url = "https://arxiv.org/abs/2406.04329"
base_repo = "https://github.com/org/paper-code"

# ARIS extracts weaknesses from paper
weaknesses = extract_weaknesses(paper_url)

# Generate ideas that specifically address those weaknesses
ideas = generate_targeted_ideas(weaknesses, base_repo)

# Run experiments with the exact codebase from the paper
results = run_experiments(ideas, base_repo)
```

### Pattern 4: Cross-Model Review Chain

```python
# Use different models for different stages
executor = "claude-opus-4.7"      # Fast, creative execution
reviewer_1 = "gpt-5.4"            # Rigorous critique
reviewer_2 = "oracle-pro"         # Final stress test

draft = generate_draft(executor)
critique_1 = review(draft, reviewer_1)
improved = revise(draft, critique_1, executor)
critique_2 = review(improved, reviewer_2)
final = revise(improved, critique_2, executor)
```

## Troubleshooting

### API Key Issues

```bash
# Error: "ANTHROPIC_API_KEY not found"
export ANTHROPIC_API_KEY=your_key_here

# Verify in CLI
/setup
# Check "Current configuration" section
```

### Model Availability

```bash
# List available models for your API keys
/models

# Error: "Model not available"
# Solution: Check /setup → verify API key → select available model
```

### Rate Limits

```python
# ARIS auto-retries on 429/5xx errors
# Default: 3 retries with exponential backoff

# To adjust (in skill YAML):
retry:
  max_attempts: 5
  backoff_multiplier: 2.0
  initial_delay: 1.0  # seconds
```

### Cross-Model Review Not Working

```bash
# Error: "Reviewer returned empty response"
# Common cause: reviewer API key missing

# Check:
echo $OPENAI_API_KEY  # For gpt-5.4 reviewer
echo $ANTHROPIC_API_KEY  # For claude reviewer

# Fix:
/setup
# Configure reviewer provider separately
```

### Experiment Failures

```python
# Error: "Git clone failed"
# Solution: Check repo URL, network, auth

# Error: "Experiment timeout"
# Solution: Increase timeout in skill YAML:
experiment:
  timeout: 7200  # seconds (default: 3600)
  
# Error: "CUDA out of memory"
# Solution: Add batch size reduction to experiment config:
training:
  batch_size: 16  # reduce from default
  gradient_accumulation: 4
```

### Wiki Corruption

```bash
# Error: "Wiki index corrupted"
# Solution: Rebuild index
cd ~/.aris/wiki
rm graph.json
/wiki rebuild

# Or reset wiki entirely
rm -rf ~/.aris/wiki
/setup  # Re-enable wiki
```

### Character Limit in Rebuttal

```bash
# Error: "Rebuttal exceeds character limit"
# ARIS auto-compresses, but if still over:

# 1. Use quick mode to see strategy first
/rebuttal "reviews" -- character limit: 5000, quick mode: true

# 2. Manually edit strategy.md to remove low-priority responses

# 3. Re-run with edited strategy
/rebuttal "reviews" -- character limit: 5000, resume: strategy.md
```

### Local Model Connection

```bash
# Error: "Cannot connect to LM Studio"
# Solution: 
# 1. Start LM Studio server
# 2. Check port (default: 1234)
curl http://localhost:1234/v1/models  # Should return model list

# 3. In /setup:
# Provider: LM Studio
# Base URL: http://localhost:1234/v1
# Model: <model-name-from-curl>
```

### Permission Issues (Standalone CLI)

```bash
# Error: "Permission denied" when running experiments
# Solution: ARIS prompts before executing tools

# Check permission mode:
/setup
# Tool Permission: Prompt (recommended) | Auto-allow | Deny

# Or set in config:
security:
  tool_permission: prompt  # always ask before running code
```

## Advanced Features

### Research Wiki (Persistent Memory)

Track papers, ideas, experiments across sessions:

```bash
# Add paper
/wiki add paper https://arxiv.org/abs/2305.14342

# Add idea
/wiki add idea "Factorized relative position embeddings for Transformer-XL"

# Link idea to paper
/wiki link idea:1 paper:1 "inspired_by"

# Query relationships
/wiki query "ideas from Transformer-XL paper"

# Export to Markdown
/wiki export ./research_notes/
```

**Relationship types:**
- `inspired_by`: idea → paper
- `improves_on`: idea → idea
- `validated_by`: idea → experiment
- `contradicts`: claim → claim

### Meta-Optimization (Self-Improvement)

ARIS analyzes its own logs and proposes skill improvements:

```bash
/meta-optimize

# ARIS will:
# 1. Parse conversation logs
# 2. Identify failure patterns
# 3. Generate SKILL.md patches
# 4. Apply patches (with confirmation)
```

**Example meta-optimization:**

```markdown
## Detected Pattern: Review scores often plateau at 8.0

**Root cause:** Reviewer prompt lacks specific grading rubric.

**Proposed fix:**
```diff
--- skills/workflow-3-write-paper.md
+++ skills/workflow-3-write-paper.md
@@ -15,6 +15,12 @@
 
 Review this paper draft and provide:
 - Score (1-10)
 - Strengths (3-5 points)
 - Weaknesses (3-5 points)
+
+**Grading rubric:**
+- 9-10: Publication-ready, minor edits only
+- 7-8: Strong work, needs revisions
+- 5-6: Major issues, needs rework
+- 1-4: Fundamental flaws
```

**Apply this patch? (y/n):** y
```

### Plan Mode

Break down complex tasks before execution:

```bash
/plan "implement a vision transformer from scratch and train on CIFAR-10"

# ARIS generates step-by-step plan:
# 1. Set up environment (PyTorch, datasets)
# 2. Implement patch embedding layer
# 3. Implement transformer encoder blocks
# 4. Implement classification head
# 5. Write training loop
# 6. Run experiments with different configs
# 7. Analyze results

# Execute plan:
/execute-plan
```

## Integration with Other Tools

### Cursor Integration

```bash
# In Cursor settings.json:
{
  "cursor.aiRules": [
    "Use ARIS skills from ~/claude-code-skills/aris/skills/",
    "For research tasks, trigger ARIS workflows",
    "For paper review, use workflow-3-write-paper.md"
  ]
}
```

### Trae Integration

See [TRAE_ARIS_RUNBOOK_EN.md](https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep/blob/main/docs/TRAE_ARIS_RUNBOOK_EN.md) for detailed setup.

### Codex CLI Integration

```bash
# ARIS has native Codex CLI skills
cd skills/skills-codex/

# Use with OpenAI Codex
codex chat --system-file workflow-1-discover-paper.md
```

### Oracle MCP (GPT-5.4 Pro Reviewer)

```bash
# Install Oracle MCP
npm install -g @steipete/oracle

# Configure in /setup
# Reviewer: oracle-pro

# Or in skill YAML:
reviewer: oracle-pro
```

## Best Practices

1. **Start small**: Use targeted workflows (Workflow 1-4) before full pipeline
2. **Validate reviews**: Cross-model review is powerful but not infallible — verify critiques
3. **Use Research Wiki**: Enable persistent memory for long-term projects
4. **Monitor costs**: GPT-5.4 Pro (oracle) is expensive; use for final review only
5. **Version control**: ARIS creates Git branches for experiments — commit frequently
6. **Iterate**: Auto-review loops improve quality — aim for 3+ rounds
7. **Read logs**: ARIS logs all LLM interactions in `~/.aris/logs/` — useful for debugging
8. **Customize skills**: Fork and modify SKILL.md files for your domain

## Resources

- **Project**: https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep
- **Technical Report**: https://huggingface.co/papers/2605.03042
- **Agent Guide**: [AGENT_GUIDE.md](https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep/blob/main/AGENT_GUIDE.md)
- **Alternative Models**: [Model Configuration Guide](https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep/blob/main/docs/MiniMax-GLM-Configuration.md)
- **Community**: Discord, GitHub Discussions

---

**License**: MIT

**Contributing**: PRs welcome — see [CONTRIBUTING.md](https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep/blob/main/CONTRIBUTING.md)
