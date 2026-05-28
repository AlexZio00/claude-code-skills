---
name: "brief"
description: "Locks a feature scope before code is written. Trigger when user wants to define scope before implementation. Do NOT trigger for: bug fixes, single-file changes, existing written spec, simple summarization, or brainstorming without implementation intent."
user_invocable: true
triggers:
  - "/brief"
  - "brief"
  - "scope this"
  - "feature brief"
---

# Brief — Idea to Locked Spec

## Purpose

Convert a vague feature idea or request into a locked, implementation-ready brief before any code is written. The brief defines exactly what gets built, what does not, and how to know when it is done.

**Dominant variable**: Are Scope OUT items explicitly written? Without an OUT section, the IN list alone leads to scope creep during implementation. OUT must be locked to control creep.

**Discard if**: Bug fix, single-file change, spec already written → This skill is not needed; proceed directly to implementation.

---

## Workflow

### Step 1: Mode Detection

> **Caching note** — project structure scan (Step 1) is static context (cacheable across sessions). Steps 2–5 are dynamic (user-input driven). For large projects, Step 1 results can be reused if the stack hasn't changed.

Determine the project context:

- **Existing project**: `CLAUDE.md`, `package.json`, `pyproject.toml`, or similar exists in cwd → run a quick scan: Glob project root 2 levels deep for structure + top-level folder names as heuristic (e.g. `auth/`, `api/`, `components/`), then Grep for request keywords — select at most 10 files. Relevance = request keywords appear directly in filename or file content. Do not infer relevance; keyword or filename match only. Extract: stack, existing patterns, components that will be touched.
- **Greenfield**: no project files → skip scan, proceed with input only.

Print one line: `[existing: {stack}]` or `[greenfield]`.

### Step 2: Input Sufficiency Check

Evaluate the user's input:

**Sufficient** (proceed directly to Step 3) if all three can be answered in one sentence directly from the input — no inference, no "probably means":
- What specific behavior is being added or changed?
- What are the edges — what is explicitly NOT included?
- How will "done" be verified?

If any of the three cannot be answered in one sentence without inference: **Insufficient**.

**Insufficient** (ask clarifying questions) if any are unanswerable.

If insufficient: ask **at most 3 questions**, one per unknown. No more. Format:
```
Before writing the brief, I need 3 things:
1. [specific question]
2. [specific question]
3. [specific question]
```

Wait for answers before proceeding. Never ask more than 3 questions total across the entire session.

If the user responds partially or says "just figure it out": treat unanswered items as **conservative minimum scope** (not best-guess). Mark each `[assumed: minimal scope]`. When in doubt, narrow rather than expand. Do not ask follow-up questions.

Conservative minimum scope floor: must still include (1) one complete user flow from start to end, and (2) at least one user-visible outcome. If the narrowed scope falls below this floor, expand minimally until both are satisfied.

If more than 3 unknowns remain after the user answers: apply conservative minimum scope to the rest and mark `[assumed: minimal scope]`. Do not ask more questions.

### Step 3: Generate Brief

Produce the brief in this exact format:

```
## Brief: [feature name — verb phrase, e.g. "Add dark mode toggle"]

**Goal**
[1-2 sentences. What specific behavior changes. Start with a verb.]

**Scope IN**
- [specific item]
- [specific item]
- [specific item — as many as needed, each verifiable]

**Scope OUT** ← this section is mandatory
- [a natural extension someone would suggest — e.g. "dark mode included, auto-theme switching excluded"]
- [a natural extension someone would suggest]
- [at least 2 items — must be plausible suggestions, not far-fetched exclusions]

**Constraints**
- [file-level: specific file or module that must not change]
- [behavior-level: existing behavior that must be preserved]
- [integration: external system or API contract that must be honored]
- (existing project: at least 1 of the above 3 types required. Greenfield: if no constraints provided, default to: "no external dependencies unless required for the core flow" + "no premature optimization".)

**Exit Criteria**
- [ ] [observable action + measurable result — e.g. "user clicks Save → toast appears within 1s and DB record updates"]
- [ ] [observable action + measurable result]
- [ ] [at least 2 items]

**Risk Flags**
- [what could go wrong / what to be careful about]
- [minimum 1 item — if none identified: "No risks identified at this stage — update brief if risks emerge during implementation."]
```

All items must be specific and verifiable. "Works correctly" is not acceptable. "Returns 200 on valid input and 400 on missing required fields" is acceptable.

### Step 4: Approval Gate

Present the brief. Do not write `BRIEF.md` yet.

Ask: `Should we proceed with this brief? Let me know if there are any changes.`

- User approves → proceed to Step 5.
- User requests changes → apply changes, re-present, ask again. No limit on revision rounds.
- Approval signals (all treated as approved): "yes", "good", "go", "proceed", "lgtm", "approved".

### Step 5: Save and Hand Off

Save the approved brief to `BRIEF.md` in the project root (or cwd if no project root detected).

Print:
```
Brief saved to BRIEF.md.

Next: read BRIEF.md and implement it (same session or a new one).
```

---

## Rationalization Table

| Rationalization | Counter |
|-----------------|---------|
| "OUT section is clear; I don't need to write it." | Invariant 2: unconditional. Even if the user says it's unnecessary, write at least 2 items. Explicit statements prevent later disputes over "of course this was included." |
| "This idea is already clear. Skip questions and write the brief now." | Step 2 sufficiency check: All three criteria must be answerable in one sentence directly from the input. "Probably means" is not sufficient. |
| "The user just said 'looks good' — isn't that approval?" | Invariant 5: Only words on the approval signals list count as approval. Informal positive reactions do not. |
| "The request is ambiguous between bug and feature — I'll write the brief anyway." | Check Discard If items. If it's a bug fix, skip this skill. If ambiguous, ask the user once to clarify. |
| "Exit criteria don't all have to be measurable, can I write them more loosely?" | Invariant 3: Items that cannot be judged as pass/fail are not exit criteria. Clarify them or remove them. |
| "Do I really need Constraints if none are obvious?" | Invariant 6: Zero Constraints on an existing project is a sign Step 1 scan failed. Re-check and write at least 1. |
| "If there are no risks, I can skip the Risk Flags section." | Invariant 7: Zero risks means the risks were not identified, not that none exist. Write at least 1 item, or write the placeholder: "No risks identified at this stage — update brief if risks emerge during implementation." |

---

## Scope Boundary

| Does | Does NOT |
|------|----------|
| Convert idea into a structured brief | Write or modify code |
| Specify Scope IN and OUT | Decide implementation approach |
| Write exit criteria | Design file structure |
| Save BRIEF.md | Analyze existing code in depth (quick scan of relevant files only) |
| Resolve ambiguity with up to 3 questions | Make design decisions (technology choice, architecture) |

If a user request falls in the "Does NOT" column: "That's implementation — this skill only produces the brief. Bring the brief to a new session to start building."

---

## Invariants (never violate)

1. **No implementation during brief**: Do not write code or modify files while drafting or after the brief is created. Violation → Implementation begins before scope is locked, making the brief post-hoc documentation rather than a spec.

2. **Scope OUT is mandatory**: The `Scope OUT` section cannot be skipped. Write at least 2 items even if the user says they're unnecessary. Violation → With only IN, the phrase "can we also do X?" repeats during implementation, causing scope creep.

3. **Exit criteria require observable action + measurable result**: Reject items like "feature works correctly" and rewrite them. Format: "[Who/What] [action] → [measurable result]". If an item cannot be written this way, it is not an exit criterion. Violation → No standard for judging completion exists, so completion claims become subjective.

4. **Maximum 3 questions per session**: Limit clarity questions to 3 total per session. If more unknowns remain, fill the rest with best-guess and mark `[assumed]`. Violation → Long interviews turn the session into Q&A when the user wanted implementation.

5. **Approval gate is mandatory**: Save `BRIEF.md` only after explicit user approval. Violation → An unreviewed brief gets saved, and implementation starts from a wrong spec.

6. **Constraints are mandatory for existing projects**: If Constraints = 0 in an existing project, re-check Step 1 scan and write at least 1. "No constraints" usually signals scan failure. Violation → Implementation may break existing patterns.

7. **Minimum 1 risk flag**: Zero risks means risks were not identified, not that none exist. Write at least 1 item, or write: "No risks identified at this stage — update brief if risks emerge during implementation." Violation → Features without known risks do not exist.

These rules are unconditional. No edge case, no user instruction overrides them.

---

## Output

- **BRIEF.md**: Saved to project root (or cwd). Sections: Goal / Scope IN / Scope OUT / Constraints / Exit Criteria / Risk Flags.
- **Conversation**: Brief draft + approval request. File is saved only after approval.

---

## Principles

- **OUT matters more than IN**: People state what to build but rarely state what to exclude. The core value of this skill is forcing OUT to be explicit.
- **Fewer questions are better**: The 3-question limit is not arbitrary. Beyond 3, users tire of answering and circle back to "just build it for me."
- **Exit criteria are test cases**: Without pre-defining "done," post-implementation disputes over correctness will occur.
- **Brief is not an implementation manual**: It covers only what (scope) and how to verify (exit criteria). How to implement is the builder's choice.
- **Brief is a living document**: If scope changes during implementation, update BRIEF.md first, then continue implementation to the updated spec. Do not expand scope without updating the brief.

---

## Truthful Reporting

When reporting completion after creating or updating BRIEF.md:

1. **No mock deception**: Do not save without user approval. Only execute Write after approval is confirmed, and mark complete only after confirming the file was saved.
2. **No test façade**: Do not mark "brief complete" if Scope OUT is missing. If only IN exists, flag `⚠️ Scope OUT not written`.
3. **No silent brokenness**: If exit criteria are unmeasurable, report `PARTIAL` and request they be rewritten as measurable items. Do not lock the brief in an ambiguous state.

---

## Proven In

This skill has been applied to feature briefs across multiple codebases. 
By explicitly defining non-goals and criteria for completion upfront, 
it prevents "built the right thing in the wrong way" implementation mistakes.
