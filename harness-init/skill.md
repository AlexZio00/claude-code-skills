---
name: harness-init
description: Claude Code agent infrastructure setup — interview-based, domain-preset-driven. Generates rules, hooks, memory, and agent routing with substance, not skeletons. Includes violation testing to verify the harness actually works.
user_invocable: true
---

# Harness Init — Claude Code Agent Infrastructure Setup

## Purpose
Set up the full Claude Code harness layer — rules, hooks, memory, agent routing.
Not project scaffolding (use `/project-init` for that). This is the AI orchestration layer.

Key difference from generic templates: domain presets provide **pre-filled rules with real content**,
not empty skeletons. Every harness includes reject-by-default and violation testing.

---

## Phase 0: Prerequisites

Check if `CLAUDE.md` exists in the project root.
- If yes → read it for context (Hard Rules, stack, conventions)
- If no → recommend running `/project-init` first, but don't block

Check if `~/.claude/` global structure exists.
- If yes → read existing rules to avoid conflicts
- If no → this will be the first setup

---

## Phase 1: Domain Selection (determines everything else)

### Q1 — Domain Preset
```
What kind of system are you building?

1. Trading / Finance — paper-only, reject-on-missing-data, dual verification
2. Web Application — secrets protection, input validation, auth-first
3. CLI Tool / Automation — idempotent operations, dry-run default
4. Data Pipeline / ML — reproducibility, no data leakage, version everything
5. General — start minimal, add rules as needed
6. Custom — describe your domain

Your choice determines which hard rules are pre-loaded.
You can add, modify, or remove any of them afterward.
```

After Q1, load the matching preset (see Presets section below).
Show the user what's pre-loaded and ask: "Anything to add, change, or remove?"

### Q2 — Agent Complexity (adapts based on Q1)

```
How complex is your AI agent setup?

- Minimal: rules + memory only. No agent routing.
  → Generates: rules/, memory/, hooks. Done.

- Standard: review agents (code review, testing, verification).
  → Generates: + agent routing, review gates

- Orchestrated: multi-agent with routing, sub-agents, parallel execution.
  → Generates: + agent definitions, tier priorities, keyword triggers, scope boundaries
```

**If Q2 = Minimal → skip Q3. Go to Phase 2.**
**If Q2 = Standard → ask Q3 simplified.**
**If Q2 = Orchestrated → ask Q3 full.**

### Q3 — Review Gates (only if Q2 >= Standard)

**Standard version:**
```
Which review steps before code ships?

- Basic: code review only
- Standard: code review + verification checklist
- Strict: code review + security + verification + build validation

Start with Basic if unsure. Add more after your first production incident.
```

**Orchestrated version:**
```
Let's design your review pipeline. For each gate, tell me:
- When does it trigger? (every commit? before push? before merge?)
- What should it catch? (bugs? secrets? architecture violations?)
- What blocks shipping vs. what's advisory?

Available gates:
  code-reviewer — finds issues, severity scoring, never fixes directly
  security-reviewer — secrets exposure, injection, OWASP Top 10
  verification — mandatory checklist before declaring "done"
  build-error-resolver — fixes build/type errors only, no refactoring
  database-reviewer — SQL injection, missing indexes, N+1 queries

Which do you want, and what should each one do for YOUR project?
```

### Q4 — Memory Strategy (all complexity levels)
```
How should context persist between sessions?

- Session-only: start fresh every time (fine for scripts, short projects)
- Structured: MEMORY.md + session-handoff + checkpoint skill
  → Recommended for any project lasting more than a week.

If structured: Do you want auto-checkpoint hooks?
(Reminds you to save state before /compact and on session exit)
```

### Q5 — Custom Rules (after preset review)
```
The preset loaded these Tier 0 rules: [list from preset]

Two questions:
1. Anything missing that should NEVER be violated?
2. Any style/workflow preferences?
   (e.g., "Korean conversation, English code", "concise responses",
    "commit only when I say so")
```

---

## Domain Presets

### Preset: Trading / Finance
```yaml
tier_0_immutable:
  - "reject-by-default: missing field → REJECT. No guessing, no interpolation."
  - "paper-only: no live trade execution. No exchange credentials in code."
  - "no fabrication: missing data stays null/0/UNKNOWN. Never generate fake prices."
  - "dual verification: APPROVE requires both technical AND domain validation."
  - "horizon mandatory: every decision needs day|swing|position timeframe."
  - "expiry + invalidation: every APPROVE needs both conditions."

tier_1_mandatory:
  - "verification after every code change"
  - "test coverage before merge"

tier_2_process:
  - "brainstorming before multi-file implementation"
  - "DB-only dashboard access — never call external APIs from UI"

tier_4_style:
  - "append-only logs — never overwrite"
  - "feature flags default OFF"

hooks:
  SessionStart: "load handoff file + show last signal status"
  PreCompact: "remind to checkpoint"
  Stop: "remind to checkpoint"

memory: structured (MEMORY.md + session-handoff)
```

### Preset: Web Application
```yaml
tier_0_immutable:
  - "no hardcoded secrets: all credentials via environment variables"
  - "no raw SQL: use parameterized queries or ORM only"
  - "input validation on every user-facing endpoint"
  - "auth-first: no endpoint without authentication unless explicitly public"
  - "no PII in logs: sanitize before logging"

tier_1_mandatory:
  - "security review before any auth/payment code ships"
  - "verification after every code change"

tier_2_process:
  - "API design review before implementation"
  - "migration review before schema changes"

tier_4_style:
  - "feature flags default OFF"
  - "error messages: user-friendly externally, detailed internally"

hooks:
  SessionStart: "load handoff file"
  PreCompact: "remind to checkpoint"

memory: structured
```

### Preset: CLI Tool / Automation
```yaml
tier_0_immutable:
  - "dry-run default: destructive operations require explicit --force or --confirm"
  - "no silent data loss: always confirm before overwrite/delete"
  - "idempotent operations: running twice produces same result"
  - "no hardcoded paths: all paths configurable via args or env"

tier_1_mandatory:
  - "verification after every code change"
  - "help text for every command and flag"

tier_2_process:
  - "test with edge cases: empty input, missing files, permission denied"

tier_4_style:
  - "exit codes: 0 success, 1 user error, 2 system error"
  - "stderr for errors, stdout for output"

hooks:
  SessionStart: "load handoff file"
  PreCompact: "remind to checkpoint"

memory: structured
```

### Preset: Data Pipeline / ML
```yaml
tier_0_immutable:
  - "no data leakage: train/test split before any transformation"
  - "reproducibility: random seeds explicit, all transforms versioned"
  - "no fabrication: missing values stay NaN, never impute without documentation"
  - "baseline required: no model result without comparison to naive baseline"

tier_1_mandatory:
  - "verification after every code change"
  - "experiment logging: parameters, metrics, artifacts"

tier_2_process:
  - "cross-validation before reporting metrics"
  - "feature importance before adding complexity"

tier_4_style:
  - "append-only experiment logs"
  - "notebook cells: one purpose per cell, markdown headers"

hooks:
  SessionStart: "load handoff file + show last experiment results"
  PreCompact: "remind to checkpoint"

memory: structured
```

### Preset: General
```yaml
tier_0_immutable:
  - "no fabrication: if data is missing, say so"
  - "no hardcoded secrets: use environment variables"

tier_1_mandatory:
  - "verification after every code change"

tier_2_process:
  - "test before merge"

tier_4_style:
  - "feature flags default OFF"

hooks:
  SessionStart: "load handoff file"
  PreCompact: "remind to checkpoint"

memory: structured
```

---

## Phase 2: Harness Summary

Present the full configuration for approval:

```
Harness Configuration:
- Domain: [preset name]
- Complexity: [minimal / standard / orchestrated]
- Review gates: [list with trigger conditions]
- Memory: [strategy]
- Hooks: [list with actual commands]

Tier 0 Rules (immutable):
1. [each rule]

Tier 1+ Rules:
- [grouped by tier]

Custom additions:
- [from Q5]

Files to generate: [count with list]
```

**Wait for explicit approval before generating.**

---

## Phase 3: File Generation

### 3-1. Rules

**ai-constitution.md** — always generated, content from preset + Q5:

```markdown
# AI Constitution — [Project Name]

## I. Core Identity
[Domain-specific identity statement from preset]

## II. Truth & Clarity Discipline
1. Unverifiable information → must state "unknown"
2. All key claims tagged as:
   - **Fact**: independently verifiable by third party
   - **Claim**: asserted by author/model only, not externally verified
   - **Disclosure**: predictions, projections — never treat as fact
   Single-tag rule: when ambiguous, use the more conservative tag.
3. No generating specific numbers without source
4. Confidence proportional to evidence strength
5. No definitive predictions — use probability ranges

## III. Execution Discipline
1. Answer first, reasoning second
2. No unrequested features unless enforced by active skills
3. If unsure, say so — never guess confidently

## IV. Hard Rules (Tier 0 — never bend)
[Each rule from preset, numbered]

## V. Invalidation Conditions
Each rule above is valid UNLESS:
- [conditions under which rules should be reconsidered]
- User explicitly overrides with documented reasoning
```

**agents.md** — only if complexity >= Standard:

```markdown
# Agent Orchestration

## Available Agents
[Based on Q3 selections — full descriptions, not just names]

| Agent | Does | Does NOT | Hands off to |
|-------|------|----------|-------------|
[For each selected agent]

## Routing Rules
[Keyword triggers, auto-selection patterns]

## Tier Priorities
Tier 0: Hard Rules — immutable, no agent can override
Tier 1: [mandatory workflow]
Tier 2: [process]
Tier 3: [quality gates]
Tier 4: [style]

Higher tier always wins. Same-tier conflicts → more conservative option.
```

**output-style.md** — from Q5 style preferences:

```markdown
# Output Style
[From user's style preferences in Q5]
- [each preference as a rule]
```

**development-workflow.md** — if review gates selected:

```markdown
# Development Workflow

## Review Pipeline
[Ordered gate list with trigger conditions and blocking behavior]

## Decision Tree
[When each gate fires, what it checks, when it blocks]
```

### 3-2. Hooks

Read existing `~/.claude/settings.json`. **Merge — never overwrite.**

Generate actual working commands, not placeholders:

```json
{
  "hooks": {
    "SessionStart": [{
      "hooks": [{
        "type": "command",
        "command": "echo '=== Session Start ==='; echo \"Project: $(basename $(pwd))\"; HANDOFF=$(ls .claude/memory/session-handoff-LATEST.md 2>/dev/null || ls memory/session-handoff-LATEST.md 2>/dev/null); if [ -n \"$HANDOFF\" ]; then echo '--- Handoff ---'; cat \"$HANDOFF\"; fi"
      }]
    }],
    "PreCompact": [{
      "hooks": [{
        "type": "command",
        "command": "echo '[PRE-COMPACT] Save session context before compacting.'"
      }]
    }],
    "Stop": [{
      "hooks": [{
        "type": "command",
        "command": "echo '[SESSION END] Consider saving context for next session.'"
      }]
    }]
  }
}
```

### 3-3. Memory Structure

```
memory/
├── MEMORY.md                      # project knowledge base
└── session-handoff-LATEST.md      # inter-session state (single file, no versions)
```

Both files generated with preset-appropriate content, not empty templates.

### 3-4. Agent Definitions (if complexity = Orchestrated)

Each agent file gets:
- Clear role statement
- Explicit scope boundaries (does / does NOT do)
- Handoff rules (when to delegate)
- Input/output format

---

## Phase 4: Violation Testing

After generating all files, run verification:

### 4-1. Structure Check
```
□ Rules don't conflict with existing global rules
□ Hooks merged (not overwritten) into settings.json
□ Memory structure created with content
□ No duplicate agent definitions
```

### 4-2. Violation Scenarios

Generate 3 test scenarios that SHOULD be rejected by the harness:

For each Tier 0 rule, create a scenario:
```
Rule: "no fabrication: missing data stays null"
Test: "Generate a price estimate for ticker XYZ when no data exists"
Expected: Claude should refuse or return null/unknown
```

Present scenarios to user:
```
I've generated 3 violation scenarios based on your Tier 0 rules.
Want me to describe them so you can mentally verify?
Or save them as docs/harness-tests.md for future reference?
```

### 4-3. Completeness Check
```
□ Every Tier 0 rule has at least one violation scenario
□ Generated files have actual content (not just headers)
□ Hooks contain working shell commands
□ Memory templates have project-specific sections
```

Any failure → fix and re-verify.

---

## Phase 5: Refinement Loop

```
Harness generated and verified.

Adjustable:
- Add/remove rules at any tier
- Change review gate pipeline
- Modify hook triggers
- Switch domain preset (regenerates Tier 0)

Approve → files confirmed
[change request] → apply and regenerate + re-verify
```

---

## Scope Decision Guide

| Item | Global (~/.claude/) | Project (.claude/) |
|------|--------------------|--------------------|
| Style preferences | Global | — |
| Review agents | Global | — |
| Domain rules (Tier 0) | — | Project |
| Domain agents | — | Project |
| Memory | — | Project |
| Hooks | Global | — |
| Constitution base | Global | Project extends |

Global = applies everywhere. Project = only this codebase.
When both exist, project-level rules extend (never weaken) global rules.

---

## Principles

- **Reject-by-default is not optional** — it's built into every preset
- **Presets provide substance, not structure** — rules have actual content
- **Interview adapts to answers** — Minimal skips half the questions
- **Merge, never overwrite** — destroying existing configs is catastrophic
- **Violation testing proves the harness works** — untested rules are decorative
- **Higher tier always wins** — no agent can override Tier 0
- **Project extends global, never weakens** — project rules add restrictions, never remove them
