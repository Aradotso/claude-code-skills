---
name: claude-code-game-studios
description: Turn Claude Code into a full game dev studio with 49 specialized AI agents, 73 workflow skills, and complete coordination system
triggers:
  - set up game development studio agents
  - configure multi-agent game dev workflow
  - use Claude Code Game Studios
  - install game dev agent hierarchy
  - create game with specialized AI agents
  - set up studio coordination system
  - use game development workflow skills
  - configure game dev QA and production pipeline
---

# Claude Code Game Studios

> Skill by [ara.so](https://ara.so) — Claude Code Skills collection.

## What It Does

Claude Code Game Studios transforms a single Claude Code session into a complete game development studio with:

- **49 specialized agents** organized in a 3-tier studio hierarchy (Directors → Department Leads → Specialists)
- **73 slash commands** covering brainstorming, design, architecture, implementation, QA, and release
- **12 automated hooks** for validation, session tracking, and quality gates
- **11 path-scoped rules** enforcing coding standards per domain (gameplay, engine, AI, UI, network)
- **41 document templates** for GDDs, ADRs, sprint plans, accessibility specs, and more

Instead of one general-purpose AI assistant, you get a coordinated team with clear responsibilities, escalation paths, and quality gates — while you stay in full control of every decision.

## Installation

### Prerequisites

```bash
# Required
npm install -g @anthropic-ai/claude-code

# Optional (for hook validation)
brew install jq  # or apt-get install jq
python3 --version  # should be 3.x
```

### Setup New Project

```bash
# Clone as template
git clone https://github.com/Donchitos/Claude-Code-Game-Studios.git my-game-project
cd my-game-project

# Start Claude Code
claude

# Run initial setup
/start
```

### Add to Existing Project

```bash
cd your-existing-game

# Clone template files
git clone https://github.com/Donchitos/Claude-Code-Game-Studios.git .ccgs-temp
cp -r .ccgs-temp/.claude .
cp .ccgs-temp/CLAUDE.md .
rm -rf .ccgs-temp

# Detect existing project structure
claude
/adopt
```

## Studio Hierarchy

Agents are organized in three tiers matching real studio structure:

### Tier 1 — Directors (Opus Model)

```yaml
creative-director:    # Guards game vision, resolves design conflicts
technical-director:   # Owns architecture decisions, performance targets
producer:            # Coordinates cross-department changes, manages scope
```

### Tier 2 — Department Leads (Sonnet Model)

```yaml
game-designer:        # Owns systems design, balance
lead-programmer:      # Code architecture, tech stack decisions
art-director:         # Visual style, asset pipeline
audio-director:       # Audio design, implementation strategy
narrative-director:   # Story structure, dialogue systems
qa-lead:             # Test strategy, quality gates
release-manager:     # Build pipeline, deployment
localization-lead:   # i18n strategy, text management
```

### Tier 3 — Specialists (Sonnet/Haiku Model)

```yaml
# Programming
gameplay-programmer, engine-programmer, ai-programmer,
network-programmer, tools-programmer, ui-programmer

# Design
systems-designer, level-designer, economy-designer,
live-ops-designer, ux-designer

# Art & Audio
technical-artist, sound-designer

# Production & QA
qa-tester, accessibility-specialist, performance-analyst,
devops-engineer, analytics-engineer, security-engineer,
community-manager

# Content
writer, world-builder, prototyper
```

### Engine Specialists

```yaml
# Godot 4
godot-specialist:
  - gdscript-programmer
  - shader-programmer
  - gdextension-programmer

# Unity
unity-specialist:
  - dots-ecs-programmer
  - shader-vfx-programmer
  - addressables-programmer
  - uitoolkit-programmer

# Unreal Engine 5
unreal-specialist:
  - gas-programmer
  - blueprint-programmer
  - replication-programmer
  - umg-commonui-programmer
```

## Core Workflow Commands

### Phase 1: Onboarding & Setup

```bash
/start                    # Guided onboarding (detects project stage)
/help                     # Show all available commands
/project-stage-detect     # Analyze existing project
/setup-engine godot 4.6   # Configure engine-specific settings
/adopt                    # Import existing project structure
```

### Phase 2: Game Design

```bash
/brainstorm              # Generate game concepts
/map-systems            # Map core game systems and interactions
/design-system combat    # Create detailed system GDD
/quick-design           # Lightweight design for small features
/review-all-gdds        # Audit all design documents
/propagate-design-change # Update docs after design pivot
```

**Example: Design a combat system**

```bash
# In Claude Code session
/design-system combat

# Agent (systems-designer) asks:
# 1. What's the core player fantasy?
# 2. Turn-based or real-time?
# 3. Complexity target (arcade/sim/hybrid)?
# 4. Key reference games?

# After answers, creates:
# design/systems/combat.md (GDD)
# design/systems/combat-flowchart.mermaid
# src/systems/combat/.design-manifest.json
```

### Phase 3: Architecture

```bash
/create-architecture          # Design technical architecture
/architecture-decision        # Record ADR for tech choice
/architecture-review         # Review existing architecture
/create-control-manifest     # Map input/control schemes
```

**Example: Architecture Decision Record**

```bash
/architecture-decision

# Creates docs/adr/NNNN-network-architecture.md
# Prompts for:
# - Context: Why this decision is needed
# - Options considered (with pros/cons)
# - Decision made
# - Consequences
```

### Phase 4: Implementation

```bash
/create-epics               # Break project into epics
/create-stories combat      # Generate user stories for epic
/dev-story GAME-123        # Implement specific story
/story-done GAME-123       # Mark story complete (runs QA)
/sprint-plan               # Plan upcoming sprint
/sprint-status            # Show sprint progress
```

**Example: Implement a story**

```bash
/dev-story GAME-45

# Reads production/stories/GAME-45.md
# Spawns appropriate agents (e.g., gameplay-programmer + systems-designer)
# Shows implementation plan for approval
# Writes code in src/
# Updates story status
# Runs story-level tests
```

### Phase 5: Quality & Review

```bash
/code-review src/combat/    # Review code quality
/design-review combat       # Review design doc completeness
/balance-check             # Analyze game balance
/perf-profile             # Performance analysis
/tech-debt                # Identify technical debt
/security-audit           # Security review
/consistency-check        # Cross-system consistency
```

### Phase 6: Testing

```bash
/qa-plan                  # Create test strategy
/smoke-check             # Quick sanity tests
/soak-test               # Long-running stability test
/regression-suite        # Run full regression
/test-setup              # Configure test environment
/test-helpers            # Generate test utilities
/test-evidence-review    # Validate test coverage
```

**Example: QA workflow**

```bash
# After implementing GAME-45
/story-done GAME-45

# Automatically:
# 1. Runs /smoke-check on affected systems
# 2. Checks test coverage (warns if <80%)
# 3. Updates production/stories/GAME-45.md status
# 4. Moves to "Ready for Review"
```

### Phase 7: Release

```bash
/release-checklist        # Pre-release validation
/launch-checklist        # Launch day tasks
/changelog               # Generate changelog
/patch-notes             # Write player-facing notes
/hotfix                  # Emergency fix workflow
/day-one-patch           # Plan day-one update
```

## Team Coordination Commands

For features requiring multiple departments:

```bash
/team-combat             # Combat system (design + gameplay + VFX + audio)
/team-narrative          # Story content (narrative + world-builder + writer)
/team-ui                # UI feature (ux-designer + ui-programmer + accessibility)
/team-release           # Release prep (all leads)
/team-polish            # Pre-launch polish (art + audio + QA)
/team-level             # Level creation (level-designer + technical-artist)
/team-live-ops          # Live service (live-ops-designer + analytics)
/team-qa                # Quality push (qa-lead + all testers)
```

**Example: Multi-agent feature**

```bash
/team-combat

# Spawns coordination team:
# - systems-designer (mechanics)
# - gameplay-programmer (implementation)
# - technical-artist (VFX)
# - sound-designer (audio)
# - qa-tester (combat-specific tests)

# Creates shared context document
# Coordinates work phases
# Ensures integration testing
```

## Project Structure

```
my-game/
├── CLAUDE.md                    # Master config (agent registry)
├── .claude/
│   ├── settings.json           # Hooks, permissions, safety rules
│   ├── agents/                 # 49 agent definitions
│   │   ├── creative-director.md
│   │   ├── gameplay-programmer.md
│   │   └── ...
│   ├── skills/                 # 73 slash commands
│   │   ├── start/
│   │   │   ├── SKILL.md
│   │   │   └── metadata.json
│   │   └── ...
│   ├── hooks/                  # 12 automation hooks
│   │   ├── validate-commit.sh
│   │   ├── session-start.sh
│   │   └── ...
│   ├── rules/                  # 11 coding standards
│   │   ├── gameplay-code.md
│   │   ├── engine-code.md
│   │   └── ...
│   └── docs/
│       ├── workflow-catalog.yaml
│       └── templates/          # 41 doc templates
├── src/                        # Game source code
├── assets/                     # Art, audio, data
├── design/                     # GDDs, narrative docs
├── docs/                       # ADRs, technical docs
├── tests/                      # Test suites
├── production/                 # Sprint plans, stories
└── prototypes/                 # Isolated experiments
```

## Configuration

### Engine Selection

Edit `CLAUDE.md` to activate your engine's specialists:

```markdown
## Active Specialists

### Godot 4
- godot-specialist (tier-2-lead)
  - gdscript-programmer
  - shader-programmer
  - gdextension-programmer

<!-- Disable unused engines:
### Unity
- unity-specialist (tier-2-lead)
  ...
-->
```

### Hook Configuration

Edit `.claude/settings.json`:

```json
{
  "hooks": {
    "session.open": [".claude/hooks/session-start.sh"],
    "session.close": [".claude/hooks/session-stop.sh"],
    "preToolUse": {
      "Bash": [
        ".claude/hooks/validate-commit.sh",
        ".claude/hooks/validate-push.sh"
      ]
    },
    "postToolUse": {
      "WriteFile": [".claude/hooks/validate-assets.sh"],
      "EditFile": [".claude/hooks/validate-skill-change.sh"]
    }
  }
}
```

### Permission Rules

```json
{
  "permissions": {
    "allowed_commands": [
      "git status",
      "git log",
      "pytest tests/",
      "npm test"
    ],
    "blocked_paths": [
      ".env",
      "secrets/",
      "production/credentials/"
    ]
  }
}
```

## Common Patterns

### Start New Game from Scratch

```bash
/start
# Choose: "No idea, just exploring"
/brainstorm
# Discuss concepts, pick one
/map-systems
# Define core systems
/design-system [system-name]
# Detail each system
/create-architecture
# Plan tech stack
/setup-engine godot 4.6
# Configure engine
/create-epics
# Break into epics
/create-stories [epic-name]
# Generate user stories
/sprint-plan
# Plan first sprint
```

### Import Existing Project

```bash
/adopt
# Scans src/, assets/, design/
# Detects gaps (missing GDDs, ADRs)
# Suggests next steps

/reverse-document src/combat/
# Generates design doc from code

/architecture-review
# Analyzes current structure
# Recommends improvements
```

### Daily Development Loop

```bash
/sprint-status
# See current sprint progress

/dev-story GAME-67
# Implement next story

/story-done GAME-67
# QA + mark complete

/code-review src/inventory/
# Periodic quality check
```

### Pre-Release Workflow

```bash
/gate-check alpha
# Validate alpha criteria

/regression-suite
# Full test pass

/perf-profile
# Performance validation

/release-checklist alpha
# Final checks

/changelog
# Generate changelog

/patch-notes
# Write player-facing notes
```

## Real Code Examples

### Godot 4 Combat System (GDScript)

```gdscript
# Generated by /dev-story GAME-12 (gameplay-programmer)
# Design: design/systems/combat.md
# ADR: docs/adr/0003-combat-state-machine.md

class_name CombatController
extends Node

## Combat state machine following design/systems/combat.md
## Manages turn order, damage resolution, status effects

signal turn_started(combatant: Combatant)
signal turn_ended(combatant: Combatant)
signal combat_ended(victor: Team)

enum CombatState { SETUP, PLAYER_TURN, ENEMY_TURN, RESOLUTION, ENDED }

@export var turn_time_limit: float = 30.0  # From combat.md: 30s turn timer
@export var damage_formula: DamageFormula  # Injected, see ADR-0003

var _state: CombatState = CombatState.SETUP
var _turn_queue: Array[Combatant] = []
var _active_combatant: Combatant

func _ready() -> void:
    assert(damage_formula != null, "DamageFormula required")
    _initialize_turn_queue()

func execute_action(action: CombatAction) -> void:
    # Validates action legality (design constraint: one action per turn)
    assert(action.actor == _active_combatant, "Not active combatant's turn")
    
    var result := damage_formula.calculate(action)
    _apply_damage(result)
    _check_status_effects()
    _end_turn()
```

### Unity DOTS Combat (C#)

```csharp
// Generated by /dev-story GAME-23 (dots-ecs-programmer)
// Design: design/systems/combat.md
// ADR: docs/adr/0005-dots-combat-architecture.md

using Unity.Entities;
using Unity.Mathematics;
using Unity.Collections;

[UpdateInGroup(typeof(SimulationSystemGroup))]
public partial struct CombatResolutionSystem : ISystem
{
    public void OnUpdate(ref SystemState state)
    {
        // Process all pending combat actions (batched per design/systems/combat.md)
        var ecb = new EntityCommandBuffer(Allocator.TempJob);
        
        foreach (var (action, entity) in 
            SystemAPI.Query<RefRO<CombatAction>>()
                .WithEntityAccess())
        {
            // Calculate damage using formula from combat.md
            var damage = CalculateDamage(
                action.ValueRO.AttackerPower,
                action.ValueRO.DefenderArmor,
                action.ValueRO.CriticalChance
            );
            
            // Apply damage to target (buffered to avoid structural changes)
            ecb.SetComponent(action.ValueRO.Target, new Health 
            { 
                Current = math.max(0, GetHealth(action.ValueRO.Target) - damage)
            });
            
            // Remove processed action
            ecb.DestroyEntity(entity);
        }
        
        ecb.Playback(state.EntityManager);
        ecb.Dispose();
    }
    
    private float CalculateDamage(float power, float armor, float critChance)
    {
        // Formula from design/systems/combat.md section 3.2
        var baseDamage = power * (100f / (100f + armor));
        var isCrit = UnityEngine.Random.value < critChance;
        return baseDamage * (isCrit ? 1.5f : 1.0f);  // 150% crit multiplier per design
    }
}
```

### Unreal Engine 5 GAS Ability (C++)

```cpp
// Generated by /dev-story GAME-34 (gas-programmer)
// Design: design/systems/combat.md
// ADR: docs/adr/0007-gas-ability-system.md

#pragma once

#include "CoreMinimal.h"
#include "Abilities/GameplayAbility.h"
#include "MeleeAttackAbility.generated.h"

/**
 * Melee attack ability implementing design/systems/combat.md
 * Uses GAS for damage calculation, status effects, cooldowns
 */
UCLASS()
class MYGAME_API UMeleeAttackAbility : public UGameplayAbility
{
    GENERATED_BODY()

public:
    UMeleeAttackAbility();

    virtual void ActivateAbility(
        const FGameplayAbilitySpecHandle Handle,
        const FGameplayAbilityActorInfo* ActorInfo,
        const FGameplayAbilityActivationInfo ActivationInfo,
        const FGameplayEventData* TriggerEventData
    ) override;

protected:
    /** Base damage (modified by GE_DamageCalculation) */
    UPROPERTY(EditDefaultsOnly, Category = "Combat")
    float BaseDamage = 10.0f;

    /** Attack range in cm (from combat.md: 200cm melee range) */
    UPROPERTY(EditDefaultsOnly, Category = "Combat")
    float AttackRange = 200.0f;

    /** Damage calculation gameplay effect */
    UPROPERTY(EditDefaultsOnly, Category = "Combat")
    TSubclassOf<UGameplayEffect> DamageEffectClass;

private:
    void ApplyDamageToTarget(AActor* Target);
    bool IsInAttackRange(AActor* Target) const;
};
```

## Troubleshooting

### Hooks Not Running

```bash
# Check hook permissions
ls -l .claude/hooks/
# Should be -rwxr-xr-x (executable)

# Make executable if needed
chmod +x .claude/hooks/*.sh

# Test individual hook
.claude/hooks/validate-commit.sh
```

### Agent Not Found

```bash
# Verify agent exists
ls .claude/agents/ | grep gameplay-programmer

# Check CLAUDE.md registry
grep "gameplay-programmer" CLAUDE.md

# If missing, agent was removed — restore from template or create custom
```

### Skill Fails to Load

```bash
# Validate skill metadata
cat .claude/skills/dev-story/metadata.json | jq .

# Check for syntax errors
python3 -m json.tool .claude/skills/dev-story/metadata.json

# Test skill in isolation
/skill-test dev-story

# Fix issues, then re-test
/skill-improve dev-story
```

### Validation Errors on Commit

```bash
# Hook found issues — read output
git commit -m "Add combat system"
# ❌ Validation failed: Hardcoded magic number in src/combat/damage.gd:42

# Fix the issue
# Replace: var damage = power * 1.5
# With: const CRIT_MULTIPLIER = 1.5; var damage = power * CRIT_MULTIPLIER

# Commit again
git commit -m "Add combat system"
# ✅ Validation passed
```

### Context Limit Warnings

```bash
# Project grew too large for context window
# Compact session history (preserves active.md)
claude compact

# Or work in focused mode (single epic)
/sprint-plan
# Select subset of stories to minimize loaded context
```

### Engine Specialist Not Available

```yaml
# Check CLAUDE.md for active specialists
# If godot-specialist is commented out:

## Active Specialists
<!-- 
### Godot 4
- godot-specialist
-->

# Uncomment to activate:
### Godot 4
- godot-specialist (tier-2-lead)
  - gdscript-programmer
  - shader-programmer
```

## Support & Community

- **Documentation**: [Full workflow catalog](.claude/docs/workflow-catalog.yaml)
- **Issues**: [GitHub Issues](https://github.com/Donchitos/Claude-Code-Game-Studios/issues)
- **Discussions**: [GitHub Discussions](https://github.com/Donchitos/Claude-Code-Game-Studios/discussions)
- **Support the project**: [Buy Me a Coffee](https://www.buymeacoffee.com/donchitos3) or [GitHub Sponsors](https://github.com/sponsors/Donchitos)

## License

MIT License — see [LICENSE](LICENSE) file for details.
