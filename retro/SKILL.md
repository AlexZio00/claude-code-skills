---
skill_type: workflow
tools: Read, Edit, Grep
name: retro
description: "Milestone retrospective. After a feature or sprint completes, extract what went wrong, what went right, and what patterns emerged. Write actionable lessons to tasks/lessons.md."
depends_on: []
tags: [learning, meta]
version: "1.0.0"
triggers:
  - "/retro"
  - "retrospective"
  - "retro this milestone"
  - "what did we learn"
  - "extract lessons"
  - "post-mortem"
---

# /retro — Milestone Retrospective

## Purpose

Extracts actionable lessons from a completed milestone, feature, or sprint. The goal isn't documentation — it's behavioral change. A retro that produces "be more careful" has failed. A retro that produces "when implementing against an existing API, read the source before writing call sites" succeeds.

## Dominant Variable

**Are the lessons extracted specific enough to change behavior next time?** Vague lessons ("be more careful") don't survive the next session. A good lesson names the exact trigger and the exact alternative action.

## Trigger

- `/retro`
- "retrospective"
- "retro this milestone"
- "what did we learn"
- "extract lessons"
- "post-mortem"

## Discard If

- Session just started (nothing to retro yet)
- No code changes or decisions were made — pure exploration
- Scope is trivial (1 file, under 10 min) — session-level reflection is enough
- A session checkpoint was just run and already extracted lessons for this same scope

---

## When to Use /retro vs Session Reflection

| | Session-level reflection | /retro |
|---|---|---|
| Scope | Single session | Milestone / feature / sprint |
| Trigger | End of every session | After major completion |
| Focus | What went wrong this session | Patterns across the full arc |
| Output | lessons.md entries | lessons.md entries + optional summary block |
| Depth | Shallow (3 items max) | Deep (root cause + cross-session patterns) |

If your project uses `session-checkpoint`, its built-in reflection covers single-session lessons. `/retro` is for the bigger picture — when a feature took 3+ sessions and you want to find recurring patterns across them.

---

## Phase 1: Scope Declaration

Ask (or infer from context):
- **What milestone / feature just completed?** One sentence.
- **Time span**: how many sessions / days did this cover?
- **Key files touched**: which areas of the codebase?

If user says `/retro` without context → ask one question: "Which feature or milestone are we doing retro on?"

---

## Phase 2: What Went Wrong

Scan conversation history (or ask user to recall):

Collect up to **5 friction points**:
1. Things that took longer than expected
2. Bugs that required rework
3. Wrong assumptions that were corrected
4. Tool or agent misroutes (wrong approach, wasted round trips)
5. Communication gaps (user had to repeat or re-explain)

For each: extract **root cause**, not just symptom.

```
Symptom: "verification failed twice"
Root cause: "implemented without reading existing test patterns first"
```

---

## Phase 3: What Went Right

Collect up to **3 wins** — things that worked well and should be repeated:
1. Approaches that saved time or prevented bugs
2. Patterns that should become standard practice
3. Tool/workflow choices that paid off

This phase prevents retros from being purely negative. Reinforcing good patterns is as important as correcting bad ones.

```
Win: "Wrote tests before implementation — caught 2 edge cases early"
Keep: RED-GREEN-REFACTOR pattern for bug fixes in this codebase
```

---

## Phase 4: Pattern Extract

From the friction points, identify **recurring patterns** (2+ friction points sharing the same root):

```
Pattern: "Assumed API behavior without checking docs → wrong implementation → rework"
Affected: [friction point 2, friction point 4]
Principle: verify premise before acting
```

One-off issues → no pattern, skip.

---

## Phase 5: Lesson Write

For each pattern (and significant one-off issues), write to `tasks/lessons.md`:

**Format (v2 — required)**:
```markdown
### [YYYY-MM-DD] {one-line lesson title}
> conf: 0.5 · seen: YYYY-MM-DD · obs: 1

[Concrete behavioral rule: "When X happens, do Y instead of Z."]
Source: /retro — {milestone name}
```

**Rules**:
- Title must be specific enough to search for later
- Body must name a trigger condition and an alternative action
- No vague lessons: "be more thorough" → REJECT
- If `tasks/lessons.md` does not exist, create it now before proceeding:
  ```markdown
  # Lessons
  <!-- conf scale: 0.3 (tentative) → 0.5 (moderate) → 0.7+ (verified) -->
  <!-- Format: ### [YYYY-MM-DD] Title / > conf: X · seen: YYYY-MM-DD · obs: N / body -->
  ```
- Check for duplicates first: `Grep` tasks/lessons.md for similar patterns before writing

**Update existing lesson** (if duplicate found):
- `seen` → today
- `obs +1`
- If obs reaches 3, 6, or 9 → `conf +0.1` (max 0.9)
- Append "Also seen in: {milestone name}" to body

**Lesson metadata explained**:
- `conf` (confidence): starts at 0.5, increases with repeated observations. Higher conf = more validated lesson
- `seen`: last date this lesson was relevant
- `obs` (observations): how many times this pattern has been seen

---

## Phase 6: Summary Block (optional)

If milestone covered 3+ sessions or was a significant feature, output a summary block:

```
## Retro Summary — {milestone name} ({date})
Duration: {N sessions / N days}
Friction points: {N}
Wins: {N}
New lessons: {N}
Updated lessons: {N}
Key pattern: {one sentence}
```

This is for the conversation output only — not written to any file.

---

## Scope Boundary

| Does | Does NOT |
|------|----------|
| [READ] Scan conversation for friction signals | Modify code or fix bugs discovered during retro |
| [READ] Read tasks/lessons.md for duplicate check | Delete existing lessons |
| [GREP] Search for duplicate lessons before writing | Rewrite or restructure lessons.md |
| [EDIT] Append new entries to tasks/lessons.md | Create new files outside lessons.md |
| Output summary block to conversation | Make architectural decisions |

---

## Error Recovery

Stop → Classify → Apply Recovery → Report & Resume.

| Failure type | Detection | Recovery |
|-------------|-----------|----------|
| `input_error` | No clear milestone scope | Ask: "Which feature or milestone are we doing retro on?" |
| `missing_data` | tasks/lessons.md doesn't exist | Create file with header, then proceed |
| `logic_inconsistency` | Lesson too vague to be actionable | Rewrite with specific trigger + alternative action before writing |

---

## Invariants (never violate)

1. **Specific over vague**: Every lesson must have a trigger condition ("when X") and an alternative action ("do Y"). Lessons without these are not written. Violation → lessons.md fills with noise that doesn't change behavior.

2. **Duplicate check first**: Before writing any new lesson, `Grep` tasks/lessons.md for the same pattern. Writing duplicates inflates obs counts and makes lessons harder to maintain. Violation → lessons.md becomes inconsistent.

3. **No code fixes during retro**: If a bug or improvement is discovered during retro, note it as a pending task — don't implement it inline. Violation → retro becomes an unscoped implementation session.

4. **v2 meta format required**: Every new lesson entry must include the `> conf · seen · obs` meta line. Without it, lessons can't be prioritized or aged out. Violation → high-confidence lessons are buried among noise.

5. **Root cause over symptom**: Friction points must identify the root cause, not just what went wrong on the surface. "Tests failed" is a symptom. "Implemented without reading existing patterns" is a root cause. Violation → same mistake repeats because the actual cause was never named.

---

## Rationalization Table

| Rationalization | Counter |
|----------------|---------|
| "The lesson is obvious, no need to be specific" | Invariant 1. If it's obvious now, it won't be in 3 weeks. Name the trigger. |
| "I'll check for duplicates later" | Invariant 2. Check before writing, not after. Duplicate lessons corrupt obs counts. |
| "While we're here, let me fix this bug I noticed" | Invariant 3. File a pending task. Retro scope is reflection only. |
| "Session reflection already covers this" | Check Discard If. If the milestone was multi-session, retro depth is needed. |
| "conf: 0.5 is low, I'll just set it higher" | conf starts at 0.5 for all new lessons. It earns higher conf through repeated observation, not initial optimism. |
| "Nothing went right, skip Phase 3" | Something always went right. Skipping wins makes retros feel punitive and teams stop doing them. |

---

## Examples

### Web app feature (multi-session)
```
User: "/retro — user search feature"

## Phase 1
Milestone: User search — full-text search endpoint + UI
Span: 3 sessions (2026-01-10 to 2026-01-12)
Key files: src/api/search.ts, src/components/SearchBar.tsx, src/db/queries.ts

## Phase 2 — Friction Points
1. search() query built with string concat → SQL injection flagged in review
   (root: no parameterized query pattern check before implementing)
2. Debounce logic re-implemented from scratch → existing util discovered later
   (root: no codebase search before writing new code)
3. Empty results UI showed raw "[]" for 10 min before catching it
   (root: no edge case testing for empty state)

## Phase 3 — Wins
1. Wrote integration tests early — caught the SQL issue before deploy
2. Used /freeze to lock scope — prevented search-UI from expanding into filters

## Phase 4 — Patterns
Pattern A: "Skipped codebase search → reinvented existing code" (FP2)
Pattern B: "Built query without checking security patterns → rework" (FP1)

## Phase 5 — Lessons Written
→ tasks/lessons.md: 2 new entries
  - "When adding database queries, check existing query patterns for parameterization first"
  - "Before writing any utility function, grep the codebase for existing implementations"
```

### Bug fix (single session, deeper analysis)
```
User: "/retro — auth token refresh bug"

## Phase 1
Milestone: Auth token refresh — race condition fix
Span: 1 session (2026-02-05)
Key files: src/auth/token.ts, src/middleware/auth.ts

## Phase 2 — Friction Points
1. Assumed refresh was synchronous → missed concurrent request race
   (root: didn't trace the full request lifecycle before patching)
2. First fix introduced a deadlock → had to revert and retry
   (root: added lock without checking existing lock hierarchy)

## Phase 3 — Wins
1. TDD approach caught the deadlock immediately in the second attempt

## Phase 4 — Patterns
Pattern: "Patched without tracing full flow → introduced new bug" (FP1, FP2)

## Phase 5 — Lessons Written
→ tasks/lessons.md: 1 new entry
  - "When fixing concurrency bugs, trace the full request lifecycle (lock order,
    async boundaries, shared state) before writing any fix"
```

---

## Pair with session-checkpoint

If you use `session-checkpoint`, it handles per-session micro-reflection. `/retro` handles multi-session macro-reflection. They complement each other:

```
session 1 → checkpoint (session-level lessons)
session 2 → checkpoint (session-level lessons)
session 3 → checkpoint (session-level lessons)
         → /retro (cross-session patterns from the full feature arc)
```

## Lesson lifecycle

Lessons written by `/retro` follow this lifecycle:
1. **Created** at `conf: 0.5` — newly observed pattern
2. **Reinforced** when seen again (obs +1, conf rises at obs 3/6/9)
3. **Referenced** at session start — high-conf lessons are flagged before related work
4. **Archived** when stale — `conf < 0.4` and not seen for 90+ days → move to archive
