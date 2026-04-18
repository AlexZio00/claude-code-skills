---
name: session-checkpoint
description: "Saves session state before context compaction, task switching, or ending a session. Triggers: '/session-checkpoint', 'checkpoint', 'compact', 'save progress', 'end session', 'handoff', '체크포인트', '핸드오프 저장', '컴팩트 전에', '세션 저장', '세션 체크포인트'. Runs 5-phase pipeline: context extraction → entity classification → handoff write → memory save → preservation check → compact guidance."
user_invocable: true
context: !cat memory/session-handoff-LATEST.md
---

# Session Checkpoint

## Dominant Variable
Has **what the next session must know** been clearly identified? If that identification is incomplete, go deeper on Phase 1 — compact comes after.

## Discard If
- No code changes and no open decisions this session → compact not needed
- Already ran checkpoint this session → skip, no duplicate
- Only want to update the handoff file directly → edit `memory/session-handoff-LATEST.md` manually

---

## Core Principles
- Handoff is a **single file** (`memory/session-handoff-LATEST.md`) — no version numbers
- Completed items get **deleted**, not archived
- The next session must be able to start from this file alone
- Preservation check before compact — always

---

## Phase 1: Deep Context Extraction

Extract what compact could destroy. Full Compact preserves these 9 categories — cover all that apply:

- **Open decisions** — discussed but not resolved
- **Current work state** — what is in-progress right now (file, function, where you left off)
- **User priority signals** — what was emphasized, repeated, or caused friction → save as feedback memory
- **Key tech concepts** — new patterns, APIs, architecture insights discovered this session
- **File paths + code snippets** — specific files modified, function names, critical code fragments (compact loses these)
- **Errors + fixes** — error messages encountered and how they were resolved
- **Current mental model** — code flow, bug causation, failed approaches and why they failed
- **Next steps** — concrete actions for next session (with commands if applicable)
- **Dead ends** — approaches tried and abandoned this session (prevent repetition next session)

---

## Phase 1.5: Entity Extraction (Dream Cycle pattern)

> **Triple Gate auto-trigger** (autoDream pattern, ch13):
> Accumulated tokens ≥ 5,000 AND tool calls ≥ 3 AND ≥ 24h since last checkpoint
> → When all three conditions hit simultaneously, auto-execute is recommended. Same criteria apply on manual invocation.

Scan the session conversation for four entity types:

**① Permanent fact candidates** → promote to `memory/MEMORY.md`
- Newly discovered file paths / function names / architecture decisions
- New external tools / APIs / libraries (confirm installed)
- New hard rules or constraints
- Criterion: still true next session

**② Episode entries** → append to `memory/context-log.md` (TTL tag required)
- Completion events, external events, future plans
- TTL rules: `ttl:permanent` (decisions/architecture) | `ttl:90d` (completions/plans) | `ttl:30d` (temporary states)
- Format: `[DATE] [TYPE] [ttl:Nd] [ref:0] description`

**③ Raw observations / patterns** → preserve exact user phrasing
- Insights, judgments, or frustrations expressed directly by the user
- Candidates for `tasks/lessons.md` if they describe a repeated mistake

**④ Stale detection** → force MEMORY.md promotion
- Items in `memory/context-log.md` with `[ref:N]` where N ≥ 3 → check if they belong in MEMORY.md
- Same entity appearing 3+ times → add to MEMORY.md if not already there

---

## Phase 1.6: Task-to-Skill Crystallization (GenericAgent pattern)

> **Purpose**: propose auto-promotion of repeated workflows into a skill. The point where "manual repetition 3 times" converts into "one skill call".
> **Origin**: GenericAgent — task execution crystallization pattern.

Scan this session's tool calls and user messages for **repeated workflow signatures**.

**Signature definition:**
Three elements of the same workflow must be similar:
1. **Intent** — user request type (review / analyze / verify / generate / deploy)
2. **Tool Sequence** — executed tool-call pattern (e.g., Read → Grep → Edit → Bash test)
3. **Output Shape** — final deliverable form (report / code change / file creation)

**Crystallization triggers (either one):**
- **Within-session repetition**: same signature executed **≥ 3 times** this session
- **Cross-session accumulation**: `[ref:N]` in `memory/context-log.md` for the same task type ≥ 5

**Output format when triggered:**
```
[Crystallization candidate detected]
- Signature: {Intent} + {Tool Sequence} + {Output Shape}
- Frequency: {N} this session / {M} accumulated
- Suggestion: promote this workflow into a new skill (use your skill-authoring workflow of choice)
- Proposed skill name: {snake_case_name}
- Proposed triggers: {Korean + English trigger phrases — 3 examples}
```

**Non-trigger conditions (skip crystallization):**
- One-off exploration (single Glob → Read)
- Signature duplicates an existing skill — scan `~/.claude/skills/*/SKILL.md` to confirm
- Signature is too generic → would collide with an already-established skill

**Promotion workflow:**
Propose only. Never author the skill automatically.
If user approves ("yes" / "ㅇㅇ" / "approve" / "make it") → hand off to the user's skill-authoring workflow.
If user declines ("no" / "pass" / "skip") → drop proposal, log to `tasks/lessons.md`:
`[YYYY-MM-DD] crystallization candidate rejected: {signature} — reason required`

---

## Phase 2: Write Handoff (single file update)

File: `memory/session-handoff-LATEST.md`

### Rules
1. **Read the previous handoff first** (auto-loaded above) — remove completed items
2. Update status of in-progress items
3. Add newly surfaced items
4. **Do not archive completed items** — just delete them. Git history preserves them.

### Required Sections

```markdown
## Next Actions (priority order)
1. [most urgent] — include the exact command or step to start
2. [next]

## Current Work State  ← Full Compact 2nd priority required
- In-progress work: [file, function, where you left off]
- Omit section if nothing in-progress

## Open Decisions
- [topic]: [options] — user lean: [if known]

## Remaining Issues
- [unresolved bugs or blockers]

## Context Notes (needed next session)
- [important causation discovered this session]
- [approaches tried and failed — do not repeat]
- [critical file paths and function names — for post-compact recovery]

## Current Focus
- Top priority: [what]
- Friction: [what]
```

### Forbidden
- Listing "what was done this session" — handoff is forward-looking only
- Version numbers (v1, v2, v3…)
- Keeping completed items
- Exceeding 200 lines

---

## Phase 3: Save Memory

Apply Phase 1.5 extraction results to files:

1. **`memory/MEMORY.md`** — add permanent fact candidates to the relevant section (rewrite section, never append raw)
   - Check for duplicates before adding (skip or update if already present)
   - Stale items (contradicted by current state) → fix immediately
2. **`memory/context-log.md`** — append episode entries (date + TTL + ref:0 format required)
3. **`tasks/lessons.md`** — add behavior correction rules triggered this session (only if a real mistake occurred)
4. Remove references to old handoff filenames in MEMORY.md index; unify to LATEST.

---

## Phase 4: Preservation Check

Checklist before compact:
```
□ Open decisions are in the handoff?
□ Session feedback saved to memory?
□ Next session can start from handoff alone?
□ In-progress code changes are described?
□ User's last request is complete or recorded?
```
Any NO → fix before proceeding.

---

## Phase 5: Compact Guidance

> ⚠ **NO_TOOLS mode**: `/compact` disables all tools during compression — no file reads or writes occur. All Phase 1–4 data must be persisted to files BEFORE running `/compact`.

`/compact` is a Claude Code CLI built-in — this skill cannot call it directly.
After passing the checklist, tell the user:

"Checkpoint complete. Run `/compact` to compress context. Handoff: `memory/session-handoff-LATEST.md`"

---

## Scope Boundary

| Does | Does NOT |
|------|----------|
| Record incomplete items in handoff file | Preserve completed items (delete is correct) |
| Update MEMORY.md with new/stale items | Write code or implement features |
| Run Phase 1–5 preservation checklist | Call `/compact` directly (CLI only) |
| Maintain single handoff file | Create versioned handoff files (v1, v2…) |
| Propose Crystallization candidates (Phase 1.6) | Author a new skill automatically (only after user approval + explicit handoff) |

---

## Invariants (never violate)

1. **Single handoff file**: only `memory/session-handoff-LATEST.md` exists. No versioned files (`session-handoff-v2.md` etc.). Violation → next session cannot determine which file is current, context recovery fails.

2. **Forward-looking only**: handoff must not list "what was done this session." Only "what to do next session." Violation → handoff becomes a changelog and the actual starting point is lost.

3. **No compact guidance before preservation check**: if any Phase 4 item is NO, do not tell the user to run `/compact`. Violation → incomplete work is lost in compression.

4. **Crystallization proposes only, never executes**: when Phase 1.6 detects a repeated workflow, it only suggests promotion. Automatic skill creation is forbidden — user must explicitly invoke their skill-authoring workflow. Violation → unnecessary skills generated from raw workflows pollute the library.

---

## Output

- **`memory/session-handoff-LATEST.md`** — incomplete items + open decisions + remaining issues only. Max 200 lines.
- **Memory files** — MEMORY.md updated for new/stale items (if applicable)
- **Conversation** — compact readiness confirmation + `/compact` instruction

---

## Rationalization Table

| Rationalization | Counter |
|-----------------|---------|
| "Completed items are useful for reference, keep them" | No. Keeping them turns the handoff into a changelog. Git history preserves them. Delete is the correct action. |
| "Phase 4 has one NO but it's probably fine..." | Invariant 3. The checklist exists precisely for this moment. NO state → do not guide compact. |
| "I'll save it as session-handoff-v2.md so we keep the old one too" | Invariant 1 violation. Git history keeps old versions. Versioned files cause "which one is current?" confusion that breaks context recovery. |
| "The session was short, Phase 1.5 isn't worth running" | Phase 1.5 takes 2 minutes. Missing one permanent fact means next session re-discovers it. Run it. |
| "Phase 1.6 detected a repetition — let me just author the skill right now" | Invariant 4 violation. User must judge whether the task will actually repeat. Auto-promotion turns one-off exploration into skills, polluting the library. Propose only. |
| "A repeated workflow was detected but the signature is too trivial, skip it" | If it matches Phase 1.6's "non-trigger conditions," fine. But dismissing 3+ repetitions as "trivial" is usually measurement laziness. Log it to `context-log.md [ref:N]` so the next session re-evaluates. |

---

## Pair

This skill is the closing half of the session lifecycle.
`/session-start` → work → `/session-checkpoint`

Install both or neither — they are designed as a pair.
