# claude-code-skills

Set up your project, then set up your AI. Two skills, zero guesswork.

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

## Installation

Copy the skill folders into your Claude Code skills directory:

```bash
# macOS / Linux
mkdir -p ~/.claude/skills/project-init ~/.claude/skills/harness-init
cp project-init/skill.md ~/.claude/skills/project-init/skill.md
cp harness-init/skill.md ~/.claude/skills/harness-init/skill.md

# Windows
mkdir "%USERPROFILE%\.claude\skills\project-init"
mkdir "%USERPROFILE%\.claude\skills\harness-init"
copy project-init\skill.md "%USERPROFILE%\.claude\skills\project-init\skill.md"
copy harness-init\skill.md "%USERPROFILE%\.claude\skills\harness-init\skill.md"
```

Then invoke in any Claude Code session:

```
/project-init       # scaffold a new project
/harness-init       # set up Claude Code agent infrastructure
```

---

## Recommended Flow

```
New project idea
  │
  ├─ /project-init     → CLAUDE.md + ROADMAP + .gitignore + .env.example
  │
  └─ /harness-init     → rules/ + hooks + memory/ + agent routing
```

Run `/project-init` first to establish the project foundation, then `/harness-init` to layer on the AI orchestration.

---

## Principles Embedded

These came from painful experience on a large production system:

- **CLAUDE.md before code** — re-explaining context every session is expensive
- **Hard Rules from day one** — retrofitting them means existing code may already be in violation
- **Harness before agents** — the orchestration layer determines session productivity
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
- [ ] `/skill-forge` — skill builder (interview → skill.md generation)

---

## License

MIT
