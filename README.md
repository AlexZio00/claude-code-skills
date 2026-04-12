# claude-code-skills v3.6

Audit what's broken. Scaffold what's missing. Wire the AI. Assemble the team. Lock scope before you build. Ship with confidence. Diagnose how you work.

> **Scope:** The full lifecycle pipeline for Claude Code projects — from health check to daily push gate, plus scope locking before implementation and an AI collaboration audit that turns your own work patterns into a diagnostic.
> Seven skills that build on each other. Each one is useful standalone; the full sequence covers setup to daily workflow.

---

## Quick Start

**New project (10 min):**
```
/project-init    →  CLAUDE.md + ROADMAP + .gitignore + .env.example
/harness-init    →  rules/ + hooks + memory/ + agent routing
/team-init       →  orchestrator + reviewers + implementers
then: /brief before each feature · /pre-push before each push
```

**Existing project (5 min):**
```
/project-check   →  Score N/10 + gap list ordered by severity
  gaps found     →  /project-init (Update mode) + /harness-init + /team-init
  score ≥ 8      →  fix ⚠ items only
then: /brief + /pre-push as daily habits
```

**Already set up, just want the daily loop:**
```
/brief           →  before starting any feature
/pre-push        →  before every git push
/collab-audit    →  monthly to see what patterns have shifted
```

---

## Skills

### `/brief` — Scope Locking Before Implementation

Converts a vague feature idea into a locked brief before any code is written. Forces explicit scope OUT before implementation starts.

**What it does:**
- Asks at most 3 clarifying questions, then generates the brief
- Produces: Goal / Scope IN / **Scope OUT** (mandatory, min 2 items) / Constraints / Exit Criteria / Risk Flags
- Scope OUT items must be plausible extensions someone would suggest — not far-fetched exclusions
- Exit Criteria require "observable action → measurable result" format. Vague criteria are auto-rejected and rewritten
- Saves to `BRIEF.md` only after explicit approval

**Why Scope OUT is mandatory:**
People specify what to build but rarely specify what NOT to build. The skill's core value is forcing that conversation upfront. `BRIEF.md` only saves when Scope OUT has at least 2 entries — this is unconditional.

**Conservative scope floor:**
When input is ambiguous, the skill defaults to the minimum viable scope — but always includes at least one complete user flow and one user-visible outcome. Narrows rather than expands under uncertainty.

**Discard if:** bug fix, single-file change, or a spec is already written.

---

### `/project-check` — Existing Project Health Scan

Read-only audit of an existing project across four dimensions: Infrastructure, Security, Quality, and Harness. Surfaces all gaps ordered by severity with a Score out of 10.

**What it checks:**
- **Security** (always first): hardcoded secrets, API keys, `.env` not in `.gitignore`
- **Infrastructure**: `CLAUDE.md`, `Hard Rules`, `Secrets Policy`, `ROADMAP`, `.gitignore`, `.env.example`, `docs/decisions/`
- **Quality**: test/source file ratio, debug remnants, open work markers (TODO/FIXME)
- **Harness**: `ai-constitution.md`, `agents.md`, hooks, installed agents, orchestrator type, `tasks/lessons.md`, SubagentStop hook

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
│                                # includes SessionStart, PreCompact, Stop, SubagentStop
└── projects/[project]/
    └── memory/
        ├── MEMORY.md
        ├── session-handoff-LATEST.md
        └── tasks/lessons.md     # AI behavior correction log (Boris Cherny pattern)
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

### `/pre-push` — Pre-Push Quality Gate

Mandatory pipeline that runs automatically before every `git push`. Blocks on secrets, test failures, and lint errors. Surfaces code review findings before they land in the remote.

**What it does (8 steps):**

1. **Secrets scan** — runs `scan_secrets.pl` against staged diff only. Covers 12 patterns: AWS/GCP/Azure keys, private keys, connection string passwords, hardcoded credential assignments (quoted + unquoted YAML), platform tokens (GitHub 6 types, Slack, Stripe live), Dockerfile ENV secrets, Google/Gemini API keys, npm auth tokens, LLM provider keys (Anthropic/OpenAI/HuggingFace/Replicate/Groq), Azure Storage/SAS. Scans only `+` lines — never blocks a commit that *removes* a secret.

2. **Routing & remediation** — secrets found → BLOCK with per-pattern remediation instructions. Only `*.md` / `docs/**` changed → fast exit, skip remaining steps.

3. **Supply chain check** — lists all added dependencies from `package.json`, `requirements.txt`, `go.mod`, `Cargo.toml`, etc. Flags misspelled or unfamiliar package names as potential typosquats. Never blocks — warns only.

4. **Build & test** — language-aware, runs only what changed:
   - Python (`.py` changed + `pyproject.toml`/`setup.py`/`requirements.txt` present): `pytest -q`
   - Go (`.go` changed + `go.mod` present): `go test ./...`
   - JS/TS (`.ts`/`.tsx`/`.js`/`.jsx` changed + `package.json`): `npm run build` then `npm test`

5. **Lint gate** — direct lint first, AI lint second:
   - Python: `ruff check` (preferred) → `flake8` fallback
   - Go: `go vet ./...`
   - JS/TS: `eslint` (if config file present)
   - Then **quick-validator** (haiku) for type errors and logical issues static tools miss — skipped if diff < 50 lines

6. **Parallel AI review** — spawns applicable agents in one turn:
   - `code-reviewer` (sonnet) — always
   - `security-reviewer` (opus) — triggered by auth files, API routes, infra, dangerous patterns (`eval`, `exec`, `child_process`, `dangerouslySetInnerHTML`), or new packages
   - `database-reviewer` (sonnet) — triggered by migrations, SQL, `prisma/**`
   - `refactor-cleaner` (sonnet) — triggered when 10+ files changed

7. **Gate check** — Critical/High findings block push. Medium: fix if < 5 min or add `// TODO(security):`. Low: report only.

8. **Report & push** — structured summary with elapsed time per step. Executes `git push` only when all gates pass.

**Extra safeguards:**
- Pushes to `main`/`master` require explicit confirmation
- Test files (`__tests__/**`, `*.test.*`, `*.spec.*`) — secret findings downgraded to Medium
- Emergency override: user says "skip review" → pipeline bypassed with warning printed

**Installation note:** `scripts/scan_secrets.pl` must be co-located with `SKILL.md`. The scanner is called via `find ~/.claude -name "scan_secrets.pl"` so it works regardless of where you install the skill.

---

### `/collab-audit` — AI Collaboration Audit

Analyzes your conversation history and work patterns across 13 sections to generate a behavioral diagnosis report. No surveys. Observation only.

**What it does:**
- Reads actual conversation history — message patterns, artifact production, decision style, delegation habits
- Profiles psychological tendencies (perfectionism, avoidance, context-switching, dependency patterns)
- Identifies blind spots and collaboration anti-patterns with specific evidence
- Delivers behavioral correction feedback integrated with the analysis

**Why it's bundled:**
Psychological analysis and behavioral feedback are intentionally combined. Separated, users read one and skip the other.

**Compare mode:**
```
/collab-audit compare    # diff latest two audits — shows what shifted
```

**Constraints:**
- Requires 2+ sessions or 100+ messages (or 1 high-density session: 50+ messages with substantial artifacts or deep technical dialogue)
- No self-report: does not ask you how you work — observes what you actually did
- Output saved to `~/.claude/collab-audits/YYYY-MM-DD.md`

**Why it matters:**
AI amplifies your existing work patterns — good and bad. Most people don't know which patterns are costing them until they see them documented with evidence.

---

## Installation

```bash
# macOS / Linux
SKILLS_DIR=~/.claude/skills

mkdir -p $SKILLS_DIR/{brief,project-check,project-init,harness-init,team-init,pre-push/scripts,collab-audit}

cp brief/SKILL.md           $SKILLS_DIR/brief/SKILL.md
cp project-check/SKILL.md   $SKILLS_DIR/project-check/SKILL.md
cp project-init/SKILL.md    $SKILLS_DIR/project-init/SKILL.md
cp harness-init/SKILL.md    $SKILLS_DIR/harness-init/SKILL.md
cp team-init/SKILL.md       $SKILLS_DIR/team-init/SKILL.md
cp pre-push/SKILL.md        $SKILLS_DIR/pre-push/SKILL.md
cp pre-push/scripts/scan_secrets.pl $SKILLS_DIR/pre-push/scripts/scan_secrets.pl
cp collab-audit/SKILL.md    $SKILLS_DIR/collab-audit/SKILL.md
```

```bat
:: Windows
set SKILLS=%USERPROFILE%\.claude\skills
for %d in (brief project-check project-init harness-init team-init pre-push collab-audit) do mkdir "%SKILLS%\%d" 2>nul
for %d in (brief project-check project-init harness-init team-init pre-push collab-audit) do copy %d\SKILL.md "%SKILLS%\%d\SKILL.md"
mkdir "%SKILLS%\pre-push\scripts" 2>nul
copy pre-push\scripts\scan_secrets.pl "%SKILLS%\pre-push\scripts\scan_secrets.pl"
```

Invoke in any Claude Code session:

```
/brief            # lock scope before implementation — Goal / Scope IN+OUT / Exit Criteria
/project-check    # audit an existing project (read-only)
/project-init     # scaffold a new project
/harness-init     # set up Claude Code agent infrastructure
/team-init        # assemble your coding agent team
/pre-push         # run before git push (also triggers automatically on push requests)
/collab-audit     # diagnose your AI collaboration patterns (2+ sessions or 100+ messages)
```

---

## Recommended Flow

**Existing project — start here:**
```
/project-check → Score N/10 + gap list
  ├─ gaps found → /project-init (Update mode) + /harness-init + /team-init
  └─ score ≥ 8  → only fix the ⚠ items
then: use /pre-push on every git push going forward
```

**New project — start here:**
```
/project-init    → CLAUDE.md + ROADMAP + .gitignore + .env.example
/harness-init    → rules/ + hooks + memory/ + agent routing
/team-init       → orchestrator + code-reviewer + verification + ...
/project-check   → verify everything landed correctly
/pre-push        → active on every subsequent push
```

The seven skills map to the full project lifecycle:

| Phase | Skill | Frequency | Benefit |
|-------|-------|-----------|---------|
| Diagnose | `/project-check` | On-demand | Know exactly what's broken before you touch anything — ordered by severity |
| Bootstrap | `/project-init` | Once | Hard Rules locked before the first line of code; retrofitting them costs more later |
| Wire AI | `/harness-init` | Once | Every future session starts with full context — no re-explaining conventions |
| Build team | `/team-init` | Once | Implementation drift is caught automatically; you see escalations, not every subagent output |
| **Lock scope** | **`/brief`** | **Before each feature** | **Scope OUT defined before code starts — "this too" additions get blocked at the gate, not mid-sprint** |
| **Ship daily** | **`/pre-push`** | **Every push** | **Secrets, tests, and critical review findings blocked before they land in remote** |
| Reflect | `/collab-audit` | Periodic | Work pattern blind spots surfaced with evidence — not self-report |

> **Standalone use:** Each skill works independently. `/pre-push` works on any project — it auto-detects the language and only runs relevant checks.

---

## When to Use Each Skill

| Skill | Use when… |
|-------|-----------|
| `/brief` | You're about to implement a feature and the requirements feel open-ended. Also: when a previous feature grew mid-sprint — run brief before the next one. |
| `/project-check` | You inherited a repo and want to know the setup state. Or: you just ran `/project-init` and want to verify it landed. Also: run periodically when something keeps breaking at the infrastructure level. |
| `/project-init` | Starting a new codebase. Or: you have code but no `CLAUDE.md`, no `ROADMAP`, no `.gitignore`. Also: handing a project to AI for the first time. |
| `/harness-init` | First Claude Code session on a project. Or: you keep re-explaining conventions at every session start. Run after `/project-init`. |
| `/team-init` | You want drift detection without reviewing every subagent output yourself. Or: your orchestrator was built manually and has no correction loop. Run after `/harness-init`. |
| `/pre-push` | Before every `git push`. Set it as a habit trigger — "I type `git push`, I run this first." Also useful for onboarding: run it once to show a new contributor what the baseline looks like. |
| `/collab-audit` | After 2+ weeks of active use. Or: sessions are getting less productive and you can't pinpoint why. Run monthly with `compare` mode to see what shifted. |

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
- **Scan added lines only** — blocking a commit that removes a secret is counterproductive
- **Direct lint before AI lint** — static tools are deterministic and free; save AI tokens for what grep can't catch

---

## Roadmap

- [x] `/project-check` — health scan for existing projects
- [x] `/project-init` — project scaffolding
- [x] `/harness-init` — agent infrastructure setup
- [x] `/team-init` — agent team assembly
- [x] `/pre-push` — pre-push quality gate (secrets + tests + lint + AI review)
- [x] `/collab-audit` — AI collaboration pattern diagnosis (13 sections, observation-only)
- [x] `/brief` — scope locking before implementation (Scope OUT mandatory, Exit Criteria action+result format)

---

## Changelog

### v3.6 — Scope locking (2026-04-12)
- Added `/brief` — scope locking before implementation. Scope OUT mandatory (min 2 items, plausible extensions only). Exit Criteria require observable action + measurable result format — vague criteria auto-rejected. Conservative minimum scope floor: one complete user flow + one user-visible outcome. 7 invariants.

### v3.5 — Harness Engineering patterns (2026-04-12)
- `harness-init`: Added `tasks/lessons.md` generation + SubagentStop lifecycle hook to generated harness. SessionStart hook now loads lessons.md on session start.
- `project-check`: Added `tasks/lessons.md` and SubagentStop hook to Harness scan checklist.
- `collab-audit`: Section 8 context maturity Level 5 — `tasks/lessons.md` operation as meta-layer.
- **`tasks/lessons.md` pattern** (Boris Cherny, *Programming TypeScript*): Record repeated AI mistakes as correction rules on the spot → review at next session start. Separates behavior correction (lessons.md) from technical knowledge (MEMORY.md).
- **SubagentStop hook**: Logs agent completions with ID + transcript path for debugging multi-agent workflows.

### v3.4 — Defense structure upgrade (2026-04-11)
- All 6 skills: added **Rationalization Table** (common rationalization patterns + rebuttals)
- `pre-push`: added **Dominant variable**, **Discard if**, **Invariants with consequences**, **Scope Boundary** — format now consistent with the rest of the suite
- `project-check/init/harness-init/team-init`: Rationalization Table (5 entries each)

### v3.3 — AI Collaboration Audit (2026-04-11)
- Added `/collab-audit` — 13-section behavioral diagnosis from conversation observation. Compare mode, gitignore protection, observation-only (no surveys).

### v3.2 — Orchestrator upgrade (2026-04-10)
- `team-init`: Advisor Strategy pattern added to Full orchestrator — consults opus advisor before user escalation, spot-check on sonnet after haiku pass

### v3.1 — Harness + project-check polish (2026-04-09)
- `project-check`: Anthropic key pattern, C/C++ file support, project-level agents scan
- `harness-init`: Smart Defaults, violation testing per Tier-0 rule (not 3 total), Memory Discipline unconditional label

### v3.0 — v3 rewrite (2026-04-08)
- Introduced `project-check` — read-only health scan, scale-aware, Security-first report
- Applied v3 patterns across all skills: **Dominant variable**, **Discard if**, Invariants with failure-mode consequences
- `pre-push` v3.0.0: `scan_secrets.pl` scanner (12 patterns), parallel AI review agents, language-aware test/lint detection

---

## References

- [ReS0421/coding-team-orchestrator](https://github.com/ReS0421/coding-team-orchestrator) — Several orchestration patterns in `/team-init` were adapted from this project: "Do Not Trust the Report" (spec reviewer reads code directly, not the implementer's claim), Final Integration Review (cross-task consistency check via `git diff BASE_SHA..HEAD`), and CRITICAL/IMPORTANT/MINOR severity tiers for the correction loop.
- [obra/superpowers](https://github.com/obra/superpowers) — writing-plans + subagent-driven-development patterns that influenced the planning pipeline.
- [Boris Cherny](https://github.com/bcherny) (*Programming TypeScript*, Meta Staff Engineer) — `tasks/lessons.md` behavior correction loop: record repeated mistakes as correction rules, review at session start. Separates behavior correction from technical memory.

---

## License

MIT
