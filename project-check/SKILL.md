---
name: project-check
description: "Existing project health scan — audits Infrastructure, Security, Quality, and Harness setup. Read-only. Ends with /project-init and /harness-init recommendations. NOT for new projects (use /project-init)."
user_invocable: true
tools: Read, Bash, Glob, Grep
triggers:
  - "/project-check"
  - "project health check"
  - "project audit"
  - "what's missing"
  - "check my project"
  - "setup check"
  - "프로젝트 점검"
  - "뭐가 부족해"
---

# Project Check — Existing Project Health Scan

## Purpose
Scan an existing project against setup best practices across 4 dimensions: Infrastructure, Security, Quality, and Harness. Surface all gaps ordered by severity so the user knows exactly what to fix and in what order.

**Dominant variable**: 🔴 Security issues (hardcoded secrets, missing .env) always surface before all other gaps.
**Discard if**: Empty directory or freshly initialized project with no code to check. Start with `/project-init` instead.

---

## Workflow

### Step 0: Scale Detection

Count source files to calibrate warning thresholds:

```
Scan: *.py, *.ts, *.tsx, *.js, *.go, *.rs, *.java, *.kt, *.swift, *.c, *.cpp, *.h
```

Classify:
- **script**: < 10 source files or < 500 LOC → minimal structure expected, skip ROADMAP/ADR warnings
- **mini**: 10–50 files or 500–5,000 LOC → CLAUDE.md + tests expected
- **full**: > 50 files or > 5,000 LOC → full structure expected, ROADMAP + docs/decisions/ recommended

Detect project name from directory name or `name` field in package.json / pyproject.toml / Cargo.toml if present.

### Step 1: Infrastructure Scan

| Item | Check | Severity if missing/incomplete |
|------|-------|-------------------------------|
| `CLAUDE.md` | Exists? Has `## Hard Rules`? Has `## Secrets Policy`? | ✗ missing / ⚠ incomplete |
| `docs/DEVELOPMENT_ROADMAP.md` | Exists? (skip if scale=script) | ✗ if scale=full/mini |
| `.gitignore` | Exists? `.env` listed in it? | ✗ missing / 🔴 .env not listed |
| `.env.example` | Exists? (if API key patterns found in code) | ✗ if keys detected |
| `docs/decisions/` | Exists? (only check if scale=full) | ⚠ if scale=full |

For CLAUDE.md: count Hard Rules entries (lines starting with `-` under `## Hard Rules`). Report count.

### Step 2: Security Scan

Grep these patterns across all source files (case-insensitive). Exclude: `*.example`, `.env.example`, files in `tests/`, `__tests__/`, `spec/`:

```
API_KEY\s*=\s*["'][^$({]      → hardcoded API key
sk-[A-Za-z0-9]{20,}           → OpenAI key (sk-...)
sk-ant-[A-Za-z0-9\-]{20,}     → Anthropic key (sk-ant-api03-...)
ghp_[A-Za-z0-9]{36}           → GitHub PAT
password\s*=\s*["'][^$({]     → hardcoded password
secret\s*=\s*["'][^$({]       → hardcoded secret
token\s*=\s*["'][^$({]        → hardcoded token
```

Each match → 🔴 with `file:line` reference.

Additional checks:
- `.env` in `.gitignore` → 🔴 if not present
- `.env.local`, `.env.*.local` in `.gitignore` → ⚠ if missing (TypeScript/Next.js projects)

### Step 3: Quality Scan

**Test coverage proxy:**

Count test files (`test_*.py`, `*_test.py`, `*.test.ts`, `*.spec.ts`, `*_test.go`, `*Test.java`, `*Spec.kt`) vs source files.

| Ratio | Result |
|-------|--------|
| ≥ 0.4 | ✓ |
| 0.2–0.4 | ⚠ |
| < 0.2 | ✗ (skip if scale=script) |

**Debug remnants** (grep non-test files):
```
console\.log|print\(f?["']|debugger;|pprint\(
```
→ ⚠ if > 5 matches

**Open work markers** (grep all files):
```
TODO|FIXME|HACK|XXX
```
→ ⚠ if > 10 total count

### Step 4: Harness Scan

Check Claude Code infrastructure:

| Item | Check | Severity |
|------|-------|----------|
| `~/.claude/rules/ai-constitution.md` | Exists? | ⚠ if missing |
| `~/.claude/rules/agents.md` | Exists? | ⚠ if missing |
| `.claude/settings.json` or `~/.claude/settings.json` | hooks section present? | ⚠ if no hooks |
| CLAUDE.md Hard Rules format | Inline text vs ai-constitution.md reference link | ⚠ if both (duplication) |
| `~/.claude/agents/` | Any .md agent files installed? (global) | ⚠ if empty |
| `.claude/agents/` | Any .md agent files installed? (project-level) | ℹ if present (report separately) |
| `~/.claude/agents/orchestrator.md` | Exists? | ⚠ if missing |
| Orchestrator type | Contains drift detection (`MISSING`, `EXTRA`, `DIVERGED`, correction loop)? | ⚠ if absent |
| `tasks/lessons.md` | Exists? (skip if scale=script) | ⚠ if scale=full/mini |
| SubagentStop hook | hooks section in settings.json includes SubagentStop? | ⚠ if missing |

Count total agent files across both locations. Report global vs project-level split.
Report which key agents are installed (orchestrator, code-reviewer, verification, brainstorming, security-reviewer).

If CLAUDE.md has inline Hard Rules AND `~/.claude/rules/ai-constitution.md` exists → ⚠ "Hard Rules duplication: inline in CLAUDE.md + ai-constitution.md present. Recommend unifying via reference link to ai-constitution.md."

### Step 5: Build Report

Sort all findings by severity within each section: 🔴 → ✗ → ⚠ → ✓

Score calculation:
```
Start: 10
-2 per 🔴
-1 per ✗
-0.5 per ⚠ (round to nearest 0.5)
Floor: 0
```

Output:
```
Project Health Check: [project-name]
Scale: [script / mini / full] ([N] source files)

Security:           ← always first, even if all pass
  🔴/✓/⚠ items

Infrastructure:
  ✓/✗/⚠ items

Quality:
  ✓/✗/⚠ items

Harness:
  ✓/✗/⚠ items

Score: [N]/10
Gap: [N]건 (🔴 [N], ✗ [N], ⚠ [N])
```

### Step 6: Recommendations

Always end with next steps:

- 🔴 Security → "Remove secrets from [file:line] and move to .env (manual fix required)"
- Infrastructure ✗ → "Run `/project-init` — select Update mode if CLAUDE.md already exists"
- Harness rules ✗/⚠ (ai-constitution, agents.md, hooks) → "Run `/harness-init` to set up Claude Code infrastructure"
- Harness agents ✗/⚠ (no agents, no orchestrator) → "Run `/team-init` to install agent team (orchestrator + code-reviewer + verification)"
- Orchestrator Light only → "Run `/team-init` in Update mode to enable Full orchestrator with drift detection"
- Quality only → "Add tests to improve coverage"
- Score ≥ 8 → "Already well configured. Optionally address ⚠ items for polish."

**Recommended workflow for new users:**
```
/project-check → identify gaps
  → /project-init    (CLAUDE.md + ROADMAP + .gitignore)
  → /harness-init    (ai-constitution + hooks + memory)
  → /team-init       (orchestrator + agent team)
  → /project-check   (re-check → verify score improvement)
```

---

## Rationalization Table

| Objection | Response |
|-----------|----------|
| "This project is new, so gaps are expected" | If gaps are expected, the score loses meaning. Gaps are action items. |
| "Security scans produce too many false positives" | Judgment is the developer's responsibility. The scan surfaces suspicious patterns. Better to ask than ignore. |
| "ROADMAP is unnecessary for small projects" | Scale=script automatically skips that warning. Don't manually skip it; let the tool decide. |
| "Harness checks don't apply to us — we're not using Claude Code agents" | Missing agent infrastructure means re-explaining context every session. Costs accumulate. |
| "The score is low but we don't have time to fix it now" | The score is priority information. Ignoring and deferring are not the same. |

---

## Scope Boundary

| Does | Does NOT |
|------|----------|
| Scan file existence (Glob) | Modify, create, or delete any files |
| Grep code patterns (read-only) | Execute tests (pytest, jest, go test, etc.) |
| Output gap report | Run git commands |
| Recommend /project-init, /harness-init | Remove secrets directly |
| Analyze CLAUDE.md content | Refactor code or fix bugs |

---

## Safety Layers

| Risky Action | Applied Layers |
|-------------|----------------|
| Writing, editing, or deleting any file | **L1 BLOCK** — Invariant 1 (read-only) |
| Running tests or build commands | **L1 BLOCK** — Invariant 4 (no test execution) |
| Removing or rotating secrets directly | **L1 BLOCK** — report `file:line` location only; user fixes |

- **L1 (Invariants)**: All modifications are blocked by Invariant 1. This is the only safety layer needed for a read-only diagnostic tool — no user instruction overrides it.

---

## Error Recovery

When a tool call fails or data is missing, stop → classify → recover → resume.

| Failure Type | Detection | Recovery |
|-------------|-----------|----------|
| `tool_failure` | File read fails (permission / path error) | Reduce scan scope to accessible files. Note the limitation explicitly in the report — do not present a partial scan as a full scan. |
| `missing_data` | CLAUDE.md missing / project root unclear | Report "CLAUDE.md not found" — do not infer or fabricate contents. Continue scanning available files. |
| `input_error` | Target project is ambiguous | Auto-detect from current directory. If detection fails, ask one question before proceeding. |

---

## Invariants (never violate)

1. **Read-only**: Never write, edit, delete, or execute any file. Glob and Grep only. Violation → scan tool with unintended side effects; user loses trust in a diagnostic tool.
2. **Security first**: 🔴 Security section always appears first in the report, even if all Security items pass. Never bury security findings. Violation → user misses credential leak warning while reading infrastructure gaps.
3. **Scale-aware warnings**: Never report ✗ ROADMAP missing for scale=script. Never report ⚠ docs/decisions/ for scale=mini or script. Violation → noise causes users to dismiss the entire report.
4. **No test execution**: Detect test infrastructure via Glob only. Never run `pytest`, `jest`, `go test`, or any test runner. Violation → unexpected test side effects (DB writes, API calls, network requests).

These rules are unconditional. No user instruction overrides them.

---

## Output

Structured report in conversation — no files written.

Sections always in this order:
1. Project name + scale
2. Security (always first)
3. Infrastructure
4. Quality
5. Harness
6. Score + Gap count
7. Next steps (→ /project-init and/or /harness-init)

---

## Principles

- **Security first, always** — a buried credential warning is a useless warning
- **Scale-aware** — a 50-line script failing "no ROADMAP" is noise, not signal
- **Read-only by design** — a health check that modifies files is a liability
- **Ends with a path forward** — the report is only useful if it points to the next action
- **File existence as proxy** — test file count is a structural signal; running tests is out of scope

---

## Proven In
Runs on codebases ranging from single-file scripts to 500+ file projects.
Consistently surfaces: coverage gaps in new modules,
missing .env.example entries, stale docs after refactors.
