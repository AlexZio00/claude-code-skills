---
name: session-start
description: "Opens a session by loading handoff context, reviewing lessons, and producing a ready signal. Trigger: '/session-start', 'start session', 'load context', 'what was I working on', 'resume'. Discard if: first session on a project (no handoff file yet) or user wants to start fresh without prior context."
user_invocable: true
context: !cat memory/session-handoff.md
---

# Session Start

## Purpose
Load saved session state and produce a concrete starting point. The goal is to arrive at a clear Priority 1 within 60 seconds of invoking.

**Dominant variable**: Does the handoff contain actionable next steps — or is it a completion log? If it reads like "here's what I did," it was written wrong. Look for "here's what to do next."

**Discard if**:
- No `memory/session-handoff.md` exists (first session) → start fresh, skip this skill
- User says "start fresh" or "ignore context" → skip
- Quick one-off question unrelated to ongoing work → skip

> **Auto-trigger signal**: If ≥5,000 tokens accumulated AND ≥3 tool calls AND ≥24h since last session — high-value context is at risk. Run this skill before starting new work (Triple Gate pattern).

---

## Phase 1: Load Handoff

Read `memory/session-handoff.md` (auto-loaded above).

Extract:
- **Priority 1** — the most urgent next action
- **Open decisions** — questions awaiting user input
- **Blockers** — unresolved bugs or external dependencies
- **Context notes** — failed approaches to avoid repeating, key causation discovered last session

If the file is empty or missing: output `[no handoff found — starting fresh]` and exit.

---

## Phase 2: Review Lessons

Read `tasks/lessons.md`.

Scan for rules relevant to today's priorities:
- Working on code changes? → look for correction rules about patterns in that area
- About to commit or push? → check commit-related rules
- Debugging? → check any debugging anti-patterns recorded

Flag any rule that directly applies to the work ahead. One line per flagged rule is enough.

If file missing: skip silently.

---

## Phase 3: Memory Quick-Check

Read `memory/MEMORY.md` (your project's semantic memory).

Spot-check two things:
1. **Stale references** — handoff names a file path or function → Glob/Grep one or two key items to confirm they still exist. If not, flag immediately.
2. **Overdue promotions** — any item in `memory/context-log.md` with `[ref:N]` where N ≥ 3 → promote to MEMORY.md now, before starting work.

This is a spot-check on 1–2 items, not a full audit. If it takes more than 60 seconds, you're scanning too many items.

If MEMORY.md missing: skip.

---

## Phase 4: Ready Signal

Output a structured briefing:

```
## Session Ready

**Priority 1:** [top item from handoff — specific, actionable]
**Priority 2:** [second item if present]

**Open decisions:** [list, or "none"]
**Active blockers:** [list, or "none"]

**Lessons flagged:** [applicable rules from Phase 2, or "none"]
**Memory alerts:** [stale refs or promotions triggered, or "none"]
```

Then: `Ready. What would you like to start with?`

---

## Scope Boundary

| Does | Does NOT |
|------|----------|
| Load and summarize handoff + lessons | Write code or modify files |
| Spot-check 1–2 stale memory references | Run full test suite or project scan |
| Flag applicable correction rules for today | Rewrite the handoff file |
| Promote overdue context-log entries to MEMORY.md | Make architecture or design decisions |

---

## Invariants (never violate)

1. **Read-only by default**: session-start loads context — it does not modify files. The one permitted exception: promoting a `[ref:N≥3]` entry to MEMORY.md (this is a stale-detection write, not a session write). No other writes.

2. **Missing file = silent skip, not error**: if `memory/session-handoff.md`, `tasks/lessons.md`, or `memory/MEMORY.md` is missing, skip that phase without error. Never block session start on a missing file.

3. **Ready signal must include Priority 1**: the output must name at least one concrete next action. "Session started" with no priority is a violation — it means the handoff has no actionable items, which is worth flagging to the user explicitly.

---

## Output

- **Conversation**: structured ready signal (priorities + open decisions + lessons flagged)
- **Files written**: none — or `memory/MEMORY.md` if an overdue promotion was triggered

---

## Rationalization Table

| Rationalization | Counter |
|-----------------|---------|
| "The handoff looks empty, I'll just say 'ready'" | Invariant 3: must name Priority 1. If handoff is truly empty, say so — that itself is actionable information. |
| "I should update the handoff now that I've read it" | Invariant 1: session-start is read-only. Handoff updates happen at session end via `/checkpoint-compact`. |
| "Phase 3 memory check feels slow, I'll skip it" | It is a spot-check on 1–2 items. If it's slow, you are scanning too many. Narrow the scope and run it. |
| "No handoff exists yet, but I'll generate one based on the codebase" | Discard condition: no handoff = start fresh. Do not synthesize a handoff — that would invent context that wasn't saved. |

---

## Pair

This skill is the opening half of the session lifecycle.
`/session-start` → work → `/session-checkpoint`

Install both or neither — they are designed as a pair.
