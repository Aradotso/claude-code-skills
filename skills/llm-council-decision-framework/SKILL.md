---
name: llm-council-decision-framework
description: Run decisions through 5 AI advisors with peer review using the LLM Council methodology
triggers:
  - "run a council on this"
  - "council this decision"
  - "get multiple perspectives on"
  - "pressure test this with advisors"
  - "run this through the council"
  - "get expert opinions on"
  - "war room this decision"
  - "stress test with advisors"
---

# LLM Council Decision Framework

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## Overview

LLM Council is a decision-making framework that transforms a single question into five independent expert perspectives, followed by peer review and a synthesized final recommendation. Instead of getting one AI opinion, you get diverse viewpoints that challenge assumptions, reveal blind spots, and surface conflicts before you commit.

Based on Andrej Karpathy's LLM Council methodology. Originally built by Ole Lehmann.

## Core Concept

The council operates in three phases:

1. **Independent Analysis**: 5 advisors with distinct cognitive styles analyze your question
2. **Peer Review**: Each advisor reviews the others' recommendations
3. **Synthesis**: A chairman delivers a final verdict highlighting consensus, conflicts, and action items

## When to Use the Council

**Good Use Cases:**
- Strategic decisions with high stakes (product direction, pricing, positioning)
- Evaluating multiple options where trade-offs aren't obvious
- Pressure-testing assumptions before major commitments
- Getting outside perspectives on plans you've already drafted
- Scenarios where you need to see blind spots

**Poor Use Cases:**
- Factual questions with single correct answers
- Creative generation tasks (writing, coding)
- Simple processing tasks (summarization, formatting)
- Decisions you've already committed to emotionally

## Installation

The skill is designed to work in Claude Code and Claude Cowork environments.

### Method 1: Direct Installation

Place the `SKILL.md` file in your Claude skills directory:

```bash
# For Claude Code
mkdir -p ~/.config/claude/skills
curl -o ~/.config/claude/skills/llm-council.md \
  https://raw.githubusercontent.com/aiwithremy/claude-skills-llm-council/main/SKILL.md

# For Cursor
mkdir -p ~/.cursor/skills
curl -o ~/.cursor/skills/llm-council.md \
  https://raw.githubusercontent.com/aiwithremy/claude-skills-llm-council/main/SKILL.md
```

### Method 2: AI-Assisted Installation

In a new chat:

```
Please install this Claude skill: 
https://github.com/aiwithremy/claude-skills-llm-council

Set it up so I can use it immediately.
```

The AI will fetch and configure the skill automatically.

## Usage Patterns

### Basic Invocation

Trigger the council with natural language:

```
council this: Should I launch a $97 workshop or a $497 course?
```

```
run the council on: Which of these 3 positioning angles is strongest?
[paste your options]
```

```
pressure-test this:
I'm thinking of pivoting from SaaS to info products. Here's my reasoning...
```

### Context-Rich Questions

The council performs best with detailed context:

```
war room this decision:

Context: I run a B2B SaaS with 200 customers at $50/mo. 
60% churn annually. I'm considering switching to annual-only pricing.

Options:
1. Move everyone to annual ($500/yr, ~15% discount)
2. Keep monthly but add annual option
3. New customers annual-only, grandfather existing monthly

Concerns:
- Will forcing annual increase initial friction?
- Do I have enough leverage with current value prop?
- How do I handle the 40 customers who just renewed monthly?

What should I do?
```

### Decision Documentation

Use the council to document major decisions:

```
council this for the record:

We're choosing between Rails and Next.js for our MVP.

Rails pros: Faster to ship, better conventions, team knows it
Rails cons: Harder to find devs, frontend feels dated

Next.js pros: Modern stack, easier hiring, better DX
Next.js cons: More complexity, slower initial velocity

We're leaning Rails. Talk us out of it or confirm.
```

## The Five Advisor Archetypes

Understanding the advisor perspectives helps you interpret results:

1. **The Analyst**: Data-driven, risk-focused, wants proof
2. **The Visionary**: Big-picture thinking, opportunity-focused, explores upside
3. **The Skeptic**: Challenges assumptions, finds holes, plays devil's advocate
4. **The Pragmatist**: Implementation-focused, asks "can we actually do this?"
5. **The Contrarian**: Takes the opposite position, surfaces ignored alternatives

## Interpreting Results

### Strong Consensus

When all 5 advisors agree on direction (rare), confidence is high:

```
Example output:
"Unanimous recommendation: Annual-only pricing
All advisors agree your churn problem requires commitment mechanism.
Confidence: Very High
Risk: Low (at your stage)"
```

### Productive Conflict

When advisors split but patterns emerge:

```
Example output:
"3 advisors recommend A, 2 recommend B
Split reflects genuine trade-off between speed (A) and sustainability (B)
Chairman recommendation: Hybrid approach
Start with A, transition to B at $X milestone"
```

### Analysis Paralysis

If the council reveals too many variables, simplify your question:

```
Instead of: "Should I change my pricing, positioning, and product strategy?"
Ask: "Should I change my pricing from $X to $Y?"
```

## Configuration Options

### Custom Advisor Roles

For specialized domains, you can request custom archetypes:

```
run a council with these roles:
- Technical Architect
- Security Expert  
- DevOps Engineer
- Product Manager
- Engineering Manager

Question: Should we migrate from monolith to microservices?
```

### Weighted Perspectives

Emphasize certain viewpoints:

```
council this, but weight the Pragmatist's opinion 2x:

Should we build our own auth or use Auth0?
[We have limited eng resources]
```

### Time-Bounded Decisions

Add urgency constraints:

```
emergency council - need decision by EOD:

Our API provider just 10x'd pricing. Do we:
1. Pay and pass costs to customers
2. Migrate to alternative (2 week timeline)
3. Build in-house (4 month timeline)
```

## Advanced Patterns

### Iterative Refinement

Use multiple council rounds:

```
# Round 1: Direction
council this: Should we focus on enterprise or SMB?
[review output]

# Round 2: Validation  
council this: Based on last council recommendation for enterprise,
should we hire sales first or build features first?

# Round 3: Tactics
council this: For enterprise sales hire, agency recruiter or 
internal search?
```

### Pre-Mortem Analysis

Test a decision you've already made:

```
stress test this:

We decided to launch on Product Hunt next Tuesday.
What could go wrong? What are we not seeing?
```

### Comparison Mode

Evaluate multiple options side-by-side:

```
run parallel councils on:

Option A: [details]
Option B: [details]  
Option C: [details]

Give me separate council votes on each, then rank them.
```

## Common Issues

### "The advisors all agree too easily"

- Your question may be too obvious
- Add more context about constraints and trade-offs
- Try: "What am I missing?" or "What's the contrarian take?"

### "The output is too long"

- Request executive summary: "council this, executive summary only"
- Or: "council this, key conflicts and recommendation only"

### "The advisors don't understand my domain"

- Provide more context upfront
- Use custom roles specific to your field
- Include relevant metrics/benchmarks

### "I got a good answer but want to dig deeper"

- Follow up: "Have the Skeptic and Analyst debate this specific point"
- Or: "Expand on where the Visionary and Pragmatist disagreed"

## Integration with Workflows

### In Documentation

```markdown
## Decision Log: Pricing Change

**Decision**: Move to annual-only pricing
**Date**: 2024-01-15
**Council Vote**: 4-1 in favor (Analyst dissented on timing)
**Key Risk**: Customer pushback during transition
**Mitigation**: Grandfather existing monthly for 6 months
```

### In Pull Requests

```
council this PR decision:

We're choosing between two approaches to rate limiting:
[paste technical options]

Add council output to PR description before merge.
```

### In Product Specs

```
## Feature Spec: Multiplayer Mode

### Decision Framework
council: Should we build real-time or async multiplayer first?
[council output here]

### Rationale
Based on council recommendation, proceeding with async.
Real-time deferred to Q3 pending performance infrastructure.
```

## Best Practices

1. **Be Specific**: Vague questions get vague answers
2. **Show Your Work**: Share what you've already considered
3. **Set Stakes**: Explain why the decision matters
4. **Accept Conflict**: Disagreement among advisors is valuable data
5. **Act Decisively**: The council informs, you decide
6. **Document**: Save council outputs for future reference

## Example Session

```
User: council this decision:

Context: Bootstrapped SaaS, $15K MRR, 2 founders
Decision: Raise VC funding now vs grow bootstrapped another year

Raising now:
- $1-2M seed possible based on initial conversations  
- Could hire 2-3 engineers, accelerate shipping
- Market window feels narrow (competitors growing)

Staying bootstrapped:
- Keep control and optionality
- Already profitable, could hit $50K MRR organically  
- VC changes the game, can't undo

Leaning toward staying bootstrapped but FOMO is real.

---

[Council runs...]

Chairman Verdict:
Strong 4-1 recommendation: Stay bootstrapped for 12 more months

Consensus points:
- Your profitability is rare leverage - don't surrender it prematurely
- $15K→$50K is achievable with current trajectory
- Raising at $50K MRR gets better terms and more options

Key conflict:
- Visionary warns about market timing risk
- But Analyst + Pragmatist note: competitors with funding 
  haven't proven better product-market fit

Recommended action:
Set milestone: If you're not at $40K MRR by Q4, revisit funding
Until then, optimize for profitable growth

Red flags that would change this:
- Competitor raises $10M+ and targets your customers directly
- Key team member wants to leave due to resource constraints  
- Major platform change requires immediate technical investment
```

## Resources

- Original repo: https://github.com/aiwithremy/claude-skills-llm-council
- Methodology: https://github.com/karpathy/llm-council
- Creator: https://x.com/itsolelehmann

## Troubleshooting

**Skill not triggering:**
- Verify skill file is in correct directory
- Restart Claude Code/Cowork
- Try explicit trigger: "run the council on:"

**Unexpected output format:**
- Restate: "Use the full LLM Council format with 5 advisors"
- Check you're using a supported Claude model

**Performance issues:**
- Council queries are token-intensive
- For faster results: "Quick council with 3 advisors"
- Or: "Skip peer review, just give me 5 perspectives"
