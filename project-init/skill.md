---
name: project-init
description: Interview-based project setup — generates CLAUDE.md + ROADMAP from scratch. Determines language/stack, defines hard rules, structures roadmap. Conversational flow, one question at a time.
user_invocable: true
---

# Project Init — New Project Design Interview

## Purpose
Capture every critical decision before writing a single line of code.
Patterns extracted from building a large-scale production system (98K LOC, 1275 tests, multi-agent architecture).

---

## Phase 0: Context File Check

If the user provides a file path or pastes a brief, read it first.
Otherwise start Phase 1 immediately.

---

## Phase 1: Interview (one question at a time)

Ask the questions below **one at a time**. Confirm understanding before moving to the next.
Adjust later questions based on earlier answers.

### Q1 — Core Definition
```
Describe the project in one sentence.
[What] [Who uses it] [Why they need it]
```

### Q2 — Language / Runtime
```
What language are you thinking? (If undecided, say so — let's choose together)

Decision guide:
- Python: data, ML, automation, scripting → unbeatable ecosystem
- TypeScript: web UI, API server → full-stack unification
- Java/Kotlin: Spring Boot backend, Android app → enterprise/mobile
- Go: high-performance server, CLI tools, concurrency → single binary deploy
- Rust: systems-level, embedded, extreme performance
- Swift: native iOS/macOS apps
```

### Q3 — Data Layer
```
Where does data come from, and where does it live?

- Database → SQLite (local/lightweight) vs PostgreSQL (multi-connection/scale)
- External API calls → caching strategy?
- Files only → what format?
- None (pure computation/transformation)

Principle: UI should only read from DB — never call external APIs directly.
Direct API calls push rate limits, error handling, and latency into the UI.
```

### Q4 — Interface
```
How do users interact with it?

- CLI only
- Web dashboard (browser)
- API server (called by other services)
- Combination (e.g. CLI + dashboard)
- None (background service / daemon)
```

### Q5 — Deployment
```
Where does it run?

- Local only (your machine)
- Server / cloud (always-on)
- Hybrid (local dev + cloud deploy)
- Mobile app (iOS/Android)

Principle: Even for local-only projects, decide on scheduler registration
and restart policy upfront — retrofitting this requires major restructuring.
```

### Q6 — AI / LLM
```
Any AI features?

- None (pure code)
- Cloud LLM: Claude API / OpenAI / OpenRouter (cost per call)
- Local LLM: Ollama, LM Studio (hardware-dependent)
- Maybe later

Principles:
- Always gate LLM features behind a feature flag (default OFF)
- Daily cost cap + budget guard required
- Design cloud fallback before local hardware is available
```

### Q7 — Hard Rules (Invariants)
```
Are there rules that must never be broken?

Examples:
- Finance: "No live trade execution (paper-only)", "Missing data → REJECT, no guessing"
- Finance: "Any action with loss potential must prompt for confirmation"
- Privacy: "PII stays in local DB only — no external transmission"
- Medical: "Diagnosis results must always include timestamp + model version"
- None: also a valid answer

Principle: Document these before writing code.
Adding them later means existing code may already be in violation.
```

### Q8 — Scope & Timeline
```
How long will this take? Solo or team?

- Under 1 week: script-level → keep structure minimal (CLAUDE.md only)
- 1–4 weeks: mini project → CLAUDE.md + test suite
- 1 month+: full project → complete structure + ROADMAP
- Team: add contribution guide + PR template
```

---

## Phase 2: Stack Decision Summary

Based on interview answers, present a summary:

```
Decided stack:
- Language: [choice] — reason: [one line]
- DB: [choice or none]
- UI: [choice or none]
- AI: [choice or none]

Hard Rules:
1. [from Q7]
2. [additional recommendations based on domain/scope]

Open decisions:
- [anything still undecided]
```

Confirm with user before Phase 3.

---

## Phase 3: File Generation

### 3-1. CLAUDE.md

Generate at project root using this structure:

```markdown
# [Project Name] v1.0

## Hard Rules (never bend)
- [from Q7 + additional recommendations]

## Quick Ref
- Entry: [main entrypoint]
- Tests: [test command]
- [additional references]

## Secrets Policy
- Never read, print, or log `.env` — use environment variables only.
- Never commit `.env` — `.env.example` is the template (no real values).
- New API keys → add placeholder to `.env.example` + load via env var.

## Dev Conventions
- Tests before merge. Never declare done without a passing test.
- New features: opt-in via env var, default OFF.
- Logs: append-only (never overwrite log/jsonl files).
- Commit only when explicitly requested.

## Compact Instructions
Preserve on compaction:
1. Hard Rules
2. Current active branch / uncommitted file list
3. Pending tasks and their status
4. Active errors or bugs being investigated
5. Dev Conventions
6. File paths modified in this session
```

### 3-2. docs/DEVELOPMENT_ROADMAP.md

```markdown
# [Project Name] — Development Roadmap

## Phase 1: Foundation (goal: core functionality working)
- [ ] 1-1. Project structure setup
- [ ] 1-2. DB schema / data layer
- [ ] 1-3. [Core feature #1]
- [ ] 1-4. Basic test suite

## Phase 2: Core Features
- [ ] 2-1. [Main feature]
- [ ] 2-2. [Main feature]

## Phase 3: Polish
- [ ] 3-1. Error handling hardening
- [ ] 3-2. Performance optimization
- [ ] 3-3. Documentation

## Backlog (unscheduled)
- [ ] [Future items]
```

### 3-3. .gitignore

Generate at project root based on language:

**Python:**
```gitignore
# Environment
.env
.env.local
venv/
.venv/
__pycache__/
*.pyc
*.pyo
*.pyd

# Outputs & artifacts
outputs/
*.log
*.jsonl

# IDE
.vscode/
.idea/
*.egg-info/
dist/
build/
.pytest_cache/
.mypy_cache/
```

**TypeScript / JavaScript:**
```gitignore
# Environment
.env
.env.local
.env.*.local

# Dependencies
node_modules/

# Build
.next/
dist/
build/
out/

# Logs
*.log
npm-debug.log*

# IDE
.vscode/
.idea/
```

**Go:**
```gitignore
.env
*.exe
*.exe~
*.test
*.out
vendor/
```

**Rust:**
```gitignore
.env
/target/
Cargo.lock   # remove this line if publishing a library
```

**Java / Kotlin:**
```gitignore
.env
.gradle/
build/
out/
*.class
*.jar
.idea/
*.iml
local.properties
```

**Swift:**
```gitignore
.env
.build/
*.xcworkspace/xcuserdata/
DerivedData/
*.ipa
*.dSYM.zip
```

---

### 3-4. .env.example

Generate based on what was decided in Q6 (AI/LLM) and Q3 (data layer):

```bash
# === API Keys ===
# OPENAI_API_KEY=
# ANTHROPIC_API_KEY=
# OPENROUTER_API_KEY=

# === Database ===
# DATABASE_URL=sqlite:///app.db
# POSTGRES_URL=

# === Feature Flags (default OFF) ===
# LLM_ENABLED=0
# [FEATURE_NAME]_ENABLED=0

# === App Config ===
# LOG_LEVEL=INFO
# PORT=8000
```

Only include sections relevant to the project's decided stack.
Leave all values empty — this file is a template, never a config.

---

### 3-5. Folder Structure

Auto-select based on language. Combine for multi-language projects.

**Python (data / automation / backend):**
```
[project]/
├── CLAUDE.md
├── .env.example
├── requirements.txt          # pip install -r requirements.txt
├── [main_entry].py
├── [core_module]/            # core logic
├── tests/                    # pytest
│   └── conftest.py
├── docs/
│   ├── INDEX.md
│   └── DEVELOPMENT_ROADMAP.md
├── scripts/                  # utility scripts
├── config/                   # YAML/JSON config
└── outputs/                  # artifacts (.gitignore)
```

**TypeScript — Next.js / Full-stack web:**
```
[project]/
├── CLAUDE.md
├── .env.example
├── package.json
├── tsconfig.json
├── next.config.ts            # if using Next.js
├── src/
│   ├── app/                  # App Router (Next.js 14+)
│   ├── components/
│   ├── lib/                  # utils, DB client
│   └── types/
├── tests/                    # Vitest / Jest
├── docs/
│   └── DEVELOPMENT_ROADMAP.md
└── scripts/
```

**TypeScript — API server (Express / Fastify / Hono):**
```
[project]/
├── CLAUDE.md
├── .env.example
├── package.json
├── tsconfig.json
├── src/
│   ├── index.ts              # entrypoint
│   ├── routes/
│   ├── services/             # business logic
│   ├── middleware/
│   └── types/
├── tests/
└── docs/
    └── DEVELOPMENT_ROADMAP.md
```

**Java / Kotlin — Spring Boot (backend API):**
```
[project]/
├── CLAUDE.md
├── .env.example
├── build.gradle.kts           # or pom.xml (Maven)
├── settings.gradle.kts
├── src/
│   ├── main/
│   │   ├── kotlin/            # or java/
│   │   │   └── com/[pkg]/
│   │   │       ├── Application.kt
│   │   │       ├── controller/
│   │   │       ├── service/
│   │   │       ├── repository/
│   │   │       └── domain/
│   │   └── resources/
│   │       └── application.yml
│   └── test/
│       └── kotlin/
│           └── com/[pkg]/
├── docs/
│   └── DEVELOPMENT_ROADMAP.md
└── scripts/
```

**Kotlin — Android:**
```
[project]/
├── CLAUDE.md
├── build.gradle.kts
├── settings.gradle.kts
├── app/
│   ├── build.gradle.kts
│   └── src/
│       ├── main/
│       │   ├── kotlin/com/[pkg]/
│       │   │   ├── MainActivity.kt
│       │   │   ├── ui/
│       │   │   ├── viewmodel/
│       │   │   └── data/
│       │   └── res/
│       └── test/
├── docs/
│   └── DEVELOPMENT_ROADMAP.md
└── scripts/
```

**Go (CLI / high-performance server):**
```
[project]/
├── CLAUDE.md
├── .env.example
├── go.mod
├── go.sum
├── cmd/
│   └── [app]/
│       └── main.go           # entrypoint
├── internal/                 # unexported packages
│   └── [feature]/
├── pkg/                      # exported packages
├── tests/
└── docs/
    └── DEVELOPMENT_ROADMAP.md
```

**Rust (systems / CLI):**
```
[project]/
├── CLAUDE.md
├── .env.example
├── Cargo.toml
├── src/
│   ├── main.rs               # or lib.rs for libraries
│   └── [module]/
│       └── mod.rs
├── tests/                    # integration tests
├── benches/                  # benchmarks (optional)
└── docs/
    └── DEVELOPMENT_ROADMAP.md
```

**Swift (iOS / macOS):**
```
[project]/
├── CLAUDE.md
├── [Project].xcodeproj/      # or Package.swift (SPM)
├── Sources/
│   └── [Target]/
├── Tests/
│   └── [Target]Tests/
└── docs/
    └── DEVELOPMENT_ROADMAP.md
```

---

## Phase 4: Refinement Loop

After generating files:

```
Draft complete. Review and let me know what to change.

Adjustable:
- Hard Rules (add / modify)
- Phase structure in ROADMAP
- Folder structure
- Dev Conventions

Approve → files confirmed
[change request] → apply and regenerate
```

---

## Checklist (verify before generating)

```
□ Language / runtime decided
□ Data layer decided
□ At least 1 hard rule defined
□ Hard Rules present in CLAUDE.md
□ Secrets policy included
□ .gitignore generated
□ .env.example generated (if secrets/API keys involved)
□ ROADMAP structured by phases (not flat task list)
□ Test strategy mentioned
```

Any unchecked item → return to the relevant question.

---

## Principles Embedded in This Skill

- **CLAUDE.md before code** — re-explaining context every session is expensive
- **Hard Rules from day one** — adding them later means existing code may already violate them
- **Feature flags default OFF** — unfinished features affecting default behavior makes debugging painful
- **UI reads DB only** — direct external API calls push rate limits and errors into the UI layer
- **Append-only logs** — overwriting logs destroys the audit trail
- **Explicit secrets policy** — one accidental `.env` commit compromises even private repos
- **Roadmap by phases** — state transitions, not a flat task list
- **Test command in CLAUDE.md** — hunting for it every new session adds up
