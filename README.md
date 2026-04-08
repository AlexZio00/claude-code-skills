# claude-code-skills

Set up your project, then set up your AI, then assemble your team. Three skills, zero guesswork.

> **Scope:** The foundation-to-team pipeline for Claude Code projects.
> From empty directory to working agent team in three interviews.

## Skills

### `/project-init` — New Project Setup Interview

Conversational interview that generates `CLAUDE.md` + `DEVELOPMENT_ROADMAP.md` before you write a single line of code.

**What it does:**
- Asks 8 focused questions (one at a time) to lock in stack, data layer, deployment, AI/LLM strategy, and hard rules
- Generates a `CLAUDE.md` with Hard Rules, Secrets Policy, and Dev Conventions tailored to your project
- Outputs a Phase-structured `DEVELOPMENT_ROADMAP.md`
- Generates `.gitignore` and `.env.example` matched to your stack
- Suggests the right folder structure for your language (Python, TypeScript, Java/Kotlin, Go, Rust, Swift)

**Why it matters:**
Most projects skip the "invariants first" step. By the time you add hard rules, the codebase already violates them. This skill forces the conversation upfront.

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
- Installs initial skill set

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
mkdir -p ~/.claude/skills/{project-init,harness-init,team-init}
cp project-init/skill.md ~/.claude/skills/project-init/skill.md
cp harness-init/skill.md ~/.claude/skills/harness-init/skill.md
cp team-init/skill.md ~/.claude/skills/team-init/skill.md

# Windows
for %d in (project-init harness-init team-init) do (
  mkdir "%USERPROFILE%\.claude\skills\%d" 2>nul
  copy %d\skill.md "%USERPROFILE%\.claude\skills\%d\skill.md"
)
```

Then invoke in any Claude Code session:

```
/project-init       # scaffold a new project
/harness-init       # set up Claude Code agent infrastructure
/team-init          # assemble your coding agent team
```

---

## Recommended Flow

```
New project idea
  │
  ├─ /project-init     → CLAUDE.md + ROADMAP + .gitignore + .env.example
  │
  ├─ /harness-init     → rules/ + hooks + memory/ + agent routing
  │
  └─ /team-init        → orchestrator + code-reviewer + verification + ...
```

Run them in order. Each builds on the previous:
- `/project-init` establishes the project foundation
- `/harness-init` layers on the AI rules and infrastructure
- `/team-init` assembles the agent team that operates within those rules

Each skill works standalone, but the full pipeline gives you the most coherent setup.

> **Standalone use:** `/harness-init` works without `/project-init` — it will write Hard Rules directly into `ai-constitution.md`. If you later run `/project-init`, the generated `CLAUDE.md` will check for `ai-constitution.md` and insert a reference link instead of duplicating rules. But if you run them in reverse order, manually remove the Hard Rules section from `CLAUDE.md` and replace with a reference link.

---

## Principles Embedded

These came from painful experience on a large production system:

- **CLAUDE.md before code** — re-explaining context every session is expensive
- **Hard Rules from day one** — retrofitting them means existing code may already be in violation
- **Harness before agents** — the orchestration layer determines session productivity
- **Orchestrator catches drift** — auto-correct twice, then escalate. Don't waste human attention on recoverable issues
- **Skip existing agents** — the user's customizations are sacred, never overwrite
- **Feature flags default OFF** — unfinished features affecting default behavior makes debugging painful
- **Merge, never overwrite** — destroying existing settings.json configs is catastrophic
- **Tier 0 rules are immutable** — no agent or skill can override them
- **Start with fewer review gates** — add them after the first bug that would have been caught
- **Append-only logs** — overwriting logs destroys the audit trail
- **Explicit secrets policy** — one `.env` commit compromises even private repos

---

## Roadmap

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
