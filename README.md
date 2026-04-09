# claude-code-skills v3.2

Audit what's broken. Scaffold what's missing. Wire the AI. Assemble the team. Ship with confidence.

> **Scope:** The full lifecycle pipeline for Claude Code projects — from health check to daily push gate.
> Five skills that build on each other. Each one is useful standalone; the full sequence covers setup to daily workflow.

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

## Installation

```bash
# macOS / Linux
SKILLS_DIR=~/.claude/skills

mkdir -p $SKILLS_DIR/{project-check,project-init,harness-init,team-init,pre-push/scripts}

cp project-check/SKILL.md   $SKILLS_DIR/project-check/SKILL.md
cp project-init/SKILL.md    $SKILLS_DIR/project-init/SKILL.md
cp harness-init/SKILL.md    $SKILLS_DIR/harness-init/SKILL.md
cp team-init/SKILL.md       $SKILLS_DIR/team-init/SKILL.md
cp pre-push/SKILL.md        $SKILLS_DIR/pre-push/SKILL.md
cp pre-push/scripts/scan_secrets.pl $SKILLS_DIR/pre-push/scripts/scan_secrets.pl
```

```bat
:: Windows
set SKILLS=%USERPROFILE%\.claude\skills
for %d in (project-check project-init harness-init team-init pre-push) do mkdir "%SKILLS%\%d" 2>nul
for %d in (project-check project-init harness-init team-init pre-push) do copy %d\SKILL.md "%SKILLS%\%d\SKILL.md"
mkdir "%SKILLS%\pre-push\scripts" 2>nul
copy pre-push\scripts\scan_secrets.pl "%SKILLS%\pre-push\scripts\scan_secrets.pl"
```

Invoke in any Claude Code session:

```
/project-check    # audit an existing project (read-only)
/project-init     # scaffold a new project
/harness-init     # set up Claude Code agent infrastructure
/team-init        # assemble your coding agent team
/pre-push         # run before git push (also triggers automatically on push requests)
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

The five skills map to the full project lifecycle:

| Phase | Skill | Frequency |
|-------|-------|-----------|
| Diagnose | `/project-check` | On-demand |
| Bootstrap | `/project-init` | Once |
| Wire AI | `/harness-init` | Once |
| Build team | `/team-init` | Once |
| **Ship daily** | **`/pre-push`** | **Every push** |

> **Standalone use:** Each skill works independently. `/pre-push` works on any project — it auto-detects the language and only runs relevant checks.

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

---

## References

- [ReS0421/coding-team-orchestrator](https://github.com/ReS0421/coding-team-orchestrator) — Several orchestration patterns in `/team-init` were adapted from this project: "Do Not Trust the Report" (spec reviewer reads code directly, not the implementer's claim), Final Integration Review (cross-task consistency check via `git diff BASE_SHA..HEAD`), and CRITICAL/IMPORTANT/MINOR severity tiers for the correction loop.
- [obra/superpowers](https://github.com/obra/superpowers) — writing-plans + subagent-driven-development patterns that influenced the planning pipeline.

---

## License

MIT
