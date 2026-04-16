---
name: adr
description: "Records an architecture or design decision as a structured ADR file. Trigger: '/adr', 'record this decision', 'write an adr', 'document this choice', 'architecture decision', 'why did we choose'. Use after a decision is made — not during exploration. Companion to /brief (brief locks scope; adr records the key technical choices made during or after implementation)."
user_invocable: true
---

# ADR — Architecture Decision Record

## Purpose
Capture a design or architecture decision with its context, the choice made, alternatives rejected, and consequences. Prevents re-litigating settled decisions and makes future AI sessions aware of constraints not visible in code.

**Dominant variable**: Is the "why" captured — not just the "what"? An ADR without context is a changelog entry. Future sessions will see the decision but not know when to override it.

**Discard if**: The decision is trivial (naming, formatting), already documented in code comments, or still being explored. ADR is for settled decisions, not hypotheticals.

---

## Workflow

### Step 1: Capture the Decision

If not already in the conversation, ask:
- **What was decided?** (one sentence)
- **What were the alternatives?** (list if known — skip if none were seriously considered)
- **Why this over the alternatives?**
- **What are the known downsides or constraints?**

If the user says "you know what we discussed": extract from conversation context. Do not ask for information already visible in the session.

Ask **at most one clarifying question** — if more than one gap exists, apply best-guess to the rest and mark `[inferred]`.

### Step 2: Generate ADR

Produce in this exact format:

```markdown
# MAGIC DOC: [Short Title — noun phrase, e.g. "Use SQLite for local market data cache"]

**Date:** YYYY-MM-DD  
**Status:** Accepted  
**Deciders:** [names or roles — or "solo" for individual projects]

## Context
[1–3 sentences: what problem or constraint made this decision necessary? What was the forcing function?]

## Decision
[1–2 sentences: what was chosen. Start with an active verb: "Use X", "Adopt Y", "Replace Z with W".]

## Alternatives Considered

| Option | Reason Rejected |
|--------|----------------|
| [alt 1] | [why not] |
| [alt 2] | [why not] |

(Omit table entirely if no alternatives were seriously considered — write "No alternatives considered." Do not fabricate options.)

## Consequences

**Good:**
- [concrete benefit]

**Bad / Constraints:**
- [known downside or constraint this decision introduces]
- [when this decision should be revisited]

## Override Conditions
[When is it acceptable to reverse this decision? E.g. "If throughput exceeds X" or "When library Y reaches stable release." If none identified: "No planned override — revisit if core constraints change."]
```

### Step 3: Approval Gate

Present the draft. Ask: `Save this ADR? (yes / edit / discard)`

- **yes / ㅇㅋ / ㅇㅇ / approved / lgtm / go** → proceed to Step 4
- **edit** → apply changes, re-present, ask again (no limit on rounds)
- **discard** → exit, do not save

### Step 4: Save

Determine filename: `YYYY-MM-DD-<kebab-case-title>.md`  
Save to: `docs/decisions/` (create directory if it doesn't exist)

Print:
```
ADR saved: docs/decisions/YYYY-MM-DD-<title>.md
```

---

## Scope Boundary

| Does | Does NOT |
|------|----------|
| Record a settled decision with context | Make or recommend design decisions |
| Capture alternatives and reasons for rejection | Evaluate which option is better |
| Generate `docs/decisions/YYYY-MM-DD-*.md` | Modify existing code or existing ADRs |
| Ask at most one clarifying question | Design system architecture |

---

## Invariants (never violate)

1. **Context is mandatory**: an ADR without the "why" is a changelog entry. If the user can't supply context, ask one targeted question: "What problem forced this decision?" Do not proceed to Step 2 without at least one sentence of context.

2. **No fabricated alternatives**: if no alternatives were seriously considered, write "No alternatives considered." — do not invent options to fill the table. Violation → misleads future readers about what was actually evaluated.

3. **Override Conditions required**: every ADR must state when it can be reversed. "Never" is not acceptable — constraints change. Minimum: "Revisit if core constraints change." Violation → decisions calcify into permanent rules that can never be questioned.

4. **Save only after approval**: never write `docs/decisions/*.md` until the user explicitly approves. Violation → an unreviewed ADR becomes canonical spec without the user intending it.

---

## Output

- **`docs/decisions/YYYY-MM-DD-<title>.md`** — saved after approval only
- **Conversation** — draft ADR + approval prompt

---

## Rationalization Table

| Rationalization | Counter |
|-----------------|---------|
| "Context is obvious from the title" | Invariant 1: context captures the forcing function — why now, under what constraint. Title captures what was chosen. Both are required, they answer different questions. |
| "I'll add a couple of alternatives to make it look thorough" | Invariant 2: fabricated options mislead future readers about what was evaluated. "No alternatives considered." is honest and correct. |
| "The decision is still being explored, I'll write it now to capture thinking" | Discard condition: ADR is for settled decisions. Use a comment, scratch note, or `/brief` for ongoing exploration. |
| "Override Conditions can just say 'if requirements change' — too vague to bother writing" | Invariant 3: write it anyway. Even "if requirements change significantly" is better than nothing — it signals the condition class and invites future revisiting. |
| "User approved verbally mid-conversation, that counts" | Step 3 approval signals are explicit: yes / ㅇㅋ / ㅇㅇ / approved / lgtm / go. Implicit positive reactions do not count. |

---

## Principles

- **Record the fork in the road, not the destination.** Future readers need to understand what alternatives existed and why they were rejected — not just what was chosen.
- **Short beats comprehensive.** A 2-paragraph ADR that gets written beats a 10-paragraph ADR that doesn't. If it fits on one screen, it will be read.
- **Status matters.** An ADR with `Status: Superseded` is still valuable — it explains why the old approach isn't used anymore.
- **ADRs are not postmortems.** They don't require something to have gone wrong. Record routine design choices too — the ones that feel obvious today won't feel obvious in six months.

---

## Pair

`/brief` locks scope before implementation.  
`/adr` records the key technical choices made during or after.

Brief → implement → `/adr` for the non-obvious decisions made along the way.
