# claude-code-skills v3.1

Audit what's broken. Scaffold what's missing. Wire the AI. Assemble the team.

> **Scope:** The full setup pipeline for Claude Code projects — from health check to working agent team.
> Four skills that build on each other. Each one is useful standalone; the full sequence gives you the most coherent setup.

---

## Skills

### `/project-check` — Existing Project Health Scan

Read-only audit of an existing project across four dimensions: Infrastructure, Security, Quality, and Harness. Surfaces all gaps ordered by severity with a Score out of 10.

**What it checks:**
- **Security** (always first): hardcoded secrets, API keys, `.env` not in `.gitignore`
- **Infrastructure**: `CLAUDE.md`, `Hard Rules`, `Secrets Policy`, `ROADMAP`, `.gitignore`, `.env.example`, `docs/decisions/`
- **Quality**: test/source file ratio, debug remnants, open work markers (TODO/FIXME)
- **Harness**: `ai-constitution.md`, `agents.md`, hooks, installed agents, orchestrator type

**Why it matters:**
Most projects have gaps they don't know about — missing Hard Rules, hardcoded credentials buried in a config, no test infrastructure, or a harness that was never wired up. This skill surfaces all of it in one scan before you make things worse.

**Scale-aware:** A 5-file script won't fail for missing ROADMAP. Warnings are calibrated to project size.

---

### `/project-init` — New Project Setup Interview

Conversational interview that generates `CLAUDE.md` + `DEVELOPMENT_ROADMAP.md` before you write a single line of code.

**What it does:**
- Asks 8 focused questions (one at a time) to lock in stack, data layer, deployment, AI/LLM strategy, and Hard Rules
- Generates a `CLAUDE.md` with Hard Rules, Secrets Policy, and Dev Conventions tailored to your project
- Outputs a Phase-structured `DEVELOPMENT_ROADMAP.md`
- Generates `.gitignore` and `.env.example` matched to your stack
- Generates `docs/decisions/README.md` for Architecture Decision Records (if scope > 1 month)
- Suggests the right folder structure for your language (Python, TypeScript, Java/Kotlin, Go, Rust, Swift)

**Why it matters:**
Most projects skip the "invariants first" step. By the time you add Hard Rules, the codebase already violates them. This skill forces the conversation upfront.

**Supports:** Python · TypeScript (Next.js / API) · Java · Kotlin (Spring Boot / Android) · Go · Rust · Swift

---

### `/harness-init` — Claude Code Agent Infrastructure Setup

Sets up the full Claude Code harness layer — rules, hooks, memory, agent routing — from a 6-question interview.

**What it does:**
- Determines agent complexity level (minimal → orchestrated multi-agent)
- Configures review gates (code review, security, verification)
- Sets up memory strategy (session-only → structured with handoff files)
- Generates lifecycle hooks (SessionStart, PreCompact, Stop)
- Creates tiered rule hierarchy (immutable hard rules → style preferences)
- Runs violation testing on generated rules before finalizing

**What it generates:**
```
~/.claude/
├── rules/
│   ├── ai-constitution.md       # Tier 0 — immutable rules
│   ├── agents.md                # agent routing & priorities
│   ├── output-style.md          # response formatting
│   └── development-workflow.md  # review gate pipeline
├── settings.json                # hooks (merged, not overwritten)
└── projects/[project]/
    └── memory/
        ├── MEMORY.md
        └── session-handoff-LATEST.md
```

**Why it matters:**
The harness layer determines how productive every Claude Code session will be. Building it manually takes weeks of trial-and-error. This skill captures that experience in a 5-minute conversation.

**Use this AFTER `/project-init`** — project-init scaffolds the codebase, harness-init scaffolds the AI orchestration layer on top.

---

### `/team-init` — Agent Team Assembly

Generates a complete coding team — orchestrator, reviewers, implementers — from a 3-question interview.

**What it does:**
- Determines team size (Solo 3 agents / Standard 6 / Full 9)
- Loads domain-specific review checks (Trading, Web, CLI, Data/ML, General)
- Generates an **orchestrator** with drift detection and correction loop
- Creates only missing agents — existing ones stay untouched
- Updates `agents.md` routing table (merged, not replaced)

**What it generates:**
```
~/.claude/agents/
├── orchestrator.md          # plan tracking + drift detection + gate enforcement
├── code-reviewer.md         # domain-aware review (if not exists)
├── verification.md          # completion checklist (if not exists)
├── brainstorming.md         # design-first gate (Standard+)
├── writing-plans.md         # atomic task planning (Standard+)
├── build-error-resolver.md  # build/type fixes (Standard+)
├── subagent-dev.md          # parallel implementation (Full)
├── systematic-debugging.md  # root-cause analysis (Full)
└── security-reviewer.md     # OWASP + secrets (Full)
```

**Why it matters:**
The orchestrator's correction loop catches implementation drift automatically — when code diverges from the plan, it corrects twice before escalating to you. Without it, you're manually comparing every subagent's output against the spec.

**Use this AFTER `/harness-init`** — harness-init sets the rules, team-init assembles the agents that work within those rules.

---

## Installation

Copy the skill folders into your Claude Code skills directory:

```bash
# macOS / Linux
mkdir -p ~/.claude/skills/{project-check,project-init,harness-init,team-init}
cp project-check/SKILL.md ~/.claude/skills/project-check/SKILL.md
cp project-init/SKILL.md ~/.claude/skills/project-init/SKILL.md
cp harness-init/SKILL.md ~/.claude/skills/harness-init/SKILL.md
cp team-init/SKILL.md ~/.claude/skills/team-init/SKILL.md

# Windows
for %d in (project-check project-init harness-init team-init) do (
  mkdir "%USERPROFILE%\.claude\skills\%d" 2>nul
  copy %d\SKILL.md "%USERPROFILE%\.claude\skills\%d\SKILL.md"
)
```

Then invoke in any Claude Code session:

```
/project-check      # audit an existing project (read-only)
/project-init       # scaffold a new project
/harness-init       # set up Claude Code agent infrastructure
/team-init          # assemble your coding agent team
```

---

## Recommended Flow

**Existing project — start here:**
```
/project-check → Score N/10 + gap list
  ├─ gaps found → /project-init (Update mode) + /harness-init + /team-init
  └─ score ≥ 8  → only fix the ⚠ items
```

**New project — start here:**
```
/project-init    → CLAUDE.md + ROADMAP + .gitignore + .env.example
/harness-init    → rules/ + hooks + memory/ + agent routing
/team-init       → orchestrator + code-reviewer + verification + ...
/project-check   → verify everything landed correctly
```

Run them in order. Each builds on the previous:
- `/project-init` establishes the project foundation
- `/harness-init` layers on the AI rules and infrastructure
- `/team-init` assembles the agent team that operates within those rules
- `/project-check` audits the result — or diagnoses an existing project before you start

> **Standalone use:** Each skill works independently. `/harness-init` works without `/project-init` — it will write Hard Rules directly into `ai-constitution.md`. If you later run `/project-init`, the generated `CLAUDE.md` will check for `ai-constitution.md` and insert a reference link instead of duplicating rules.

---

## v3 Design Principles

Every skill in v3 encodes three things:

- **Dominant variable** — the single factor that most determines whether the skill output is useful. Named explicitly so you know what to optimize for.
- **Discard condition** — when NOT to use the skill. A diagnostic that runs itself when it shouldn't is worse than useless.
- **Invariants with consequences** — rules that cannot be overridden, with explicit failure mode documented for each one.

These came from painful experience on a large production system:

- **CLAUDE.md before code** — re-explaining context every session is expensive
- **Hard Rules from day one** — retrofitting them means existing code may already be in violation
- **Security findings never buried** — a credential warning hidden below infrastructure gaps gets ignored
- **Scale-aware warnings** — a 5-file script failing "no ROADMAP" is noise, not signal
- **Harness before agents** — the orchestration layer determines session productivity
- **Orchestrator catches drift** — auto-correct twice, then escalate. Don't waste human attention on recoverable issues
- **Skip existing agents** — the user's customizations are sacred, never overwrite
- **Feature flags default OFF** — unfinished features affecting default behavior makes debugging painful
- **Merge, never overwrite** — destroying existing settings.json configs is catastrophic
- **Tier 0 rules are immutable** — no agent or skill can override them

---

## Roadmap

- [x] `/project-check` — health scan for existing projects
- [x] `/project-init` — project scaffolding
- [x] `/harness-init` — agent infrastructure setup
- [x] `/team-init` — agent team assembly

---

## References

- [ReS0421/coding-team-orchestrator](https://github.com/ReS0421/coding-team-orchestrator) — Several orchestration patterns in `/team-init` were adapted from this project: "Do Not Trust the Report" (spec reviewer reads code directly, not the implementer's claim), Final Integration Review (cross-task consistency check via `git diff BASE_SHA..HEAD`), and CRITICAL/IMPORTANT/MINOR severity tiers for the correction loop.
- [obra/superpowers](https://github.com/obra/superpowers) — writing-plans + subagent-driven-development patterns that influenced the planning pipeline.

---

## License

MIT
