---
skill_type: utility
tools: Read, Glob
name: freeze
description: "Scope lock for current task. Declares editable zone — everything outside is frozen (read-only). Call before starting implementation to prevent scope creep."
depends_on: []
tags: [meta, safety]
version: "1.0.0"
source: "garrytan/gstack freeze pattern — adapted (2026-05-08)"
triggers:
  - "/freeze"
  - "freeze this"
  - "scope lock"
  - "only touch these files"
  - "don't modify anything else"
  - "lock the scope"
---

# /freeze — Scope Lock

> Adapted from [garrytan/gstack](https://github.com/garrytan/gstack) freeze pattern. Core principle: no modifications outside declared scope (Execution Discipline).

## Purpose

Declares the editable zone before implementation starts. Everything outside is frozen — read-only for the rest of the task. The most common source of bugs isn't wrong code, it's code that shouldn't have been touched in the first place.

## Dominant Variable

**Is the editable zone explicitly declared?** Without it, scope creep happens naturally during implementation — it feels like "just fixing this while I'm here." The freeze declaration makes every deviation visible and deliberate.

## Trigger

- `/freeze`
- "freeze this"
- "scope lock"
- "only touch these files"
- "don't modify anything else"
- "lock the scope"

## Discard If

- Exploration / research only (no modifications planned)
- Already a clear single-file change (1 file, under 10 lines)
- Still in brainstorming / design phase (scope not yet settled)

---

## Architecture

```
User input (files / modules / glob patterns)
    ↓
[PARSE]   → classify: editable / frozen / read-only
    ↓
[VERIFY]  → Glob to confirm editable files exist
    ↓
[DECLARE] → emit FROZEN SCOPE block
    ↓
[OUTPUT]  → print and stop — no implementation
```

---

## Stage 1: PARSE

Extract from user input:
- **editable**: explicitly named files, modules, or glob patterns — modifications allowed
- **frozen**: everything else (default if not mentioned)
- **read-only**: mentioned but modification status unclear → conservatively read-only

**Glob patterns supported:**
- `/freeze src/auth/**` → all files under src/auth/ are editable
- `/freeze *.test.ts` → all test files are editable
- Glob is expanded via `Glob` tool to list actual files in the DECLARE block

If scope is too vague ("everything", "as needed") → ask **once** to clarify.
Still vague after that → freeze the broader scope (Invariant 2).

---

## Stage 2: VERIFY

Run `Glob` on the editable paths to confirm they exist.
- Missing files → warn in the DECLARE block: `⚠️ [path] not found — verify before editing`
- This prevents freezing a scope around files that were renamed or moved

---

## Stage 3: DECLARE

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔒 FROZEN SCOPE — [one-line task description]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ EDITABLE (modifications allowed)
  [file / module list — expanded from globs if used]

❌ FROZEN (no modifications)
  [remainder — or "everything outside the list above"]

⚠️  READ-ONLY (reference only)
  [things that can be read but not written]

Rules:
- FROZEN files: Edit/Write prohibited. Read only.
- If modification need is discovered → stop immediately, report to user
- Unfreeze: user says "unfreeze [file]" or "add [file] to editable"
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Stage 4: OUTPUT

Print the FROZEN SCOPE block, then stop.
No code generation, no agent spawning, no implementation start.

All implementation work in the same session follows this scope declaration.

---

## Mid-Task Scope Changes

During implementation, the user may need to expand the editable zone:

- **"unfreeze [file]"** or **"add [file] to editable"** → re-emit an updated FROZEN SCOPE block with the new file added to EDITABLE, then continue
- **"freeze 해제"** or **"unfreeze all"** → cancel the freeze entirely — all files become editable
- **AI discovers a frozen file needs changes** → stop immediately, report: "I need to modify [frozen file] because [reason]. Want me to unfreeze it?" Wait for explicit approval before proceeding.

---

## Scope Boundary

| Does | Does NOT |
|------|----------|
| [READ] Parse user input → classify editable/frozen/read-only | Write code or modify files |
| [GLOB] Verify editable paths exist | Spawn agents or start implementation |
| Output FROZEN SCOPE block | Decide unilaterally what's in scope |
| Ask one clarifying question if scope is ambiguous | Continue after declaration without user input |

---

## Error Recovery

Stop → Classify → Apply Recovery → Report & Resume.

| Failure type | Detection | Recovery |
|-------------|-----------|----------|
| `input_error` | Unclear which files/scope to freeze | One question to confirm target — no guessing |
| `logic_inconsistency` | Freeze request conflicts with simultaneous modification request | Declare "this file is frozen — modification request rejected." Never allow both |
| `missing_data` | Specified file doesn't exist (Glob returns empty) | Warn "file not found" in DECLARE block. Do not freeze a different file by guessing the path |

---

## Invariants (never violate)

1. **Frozen = absolutely no modifications**: Files declared frozen cannot be Edit/Write'd in that session. "It's just a tiny change" is a rationalization. Violation → scope creep, unexpected side effects.

2. **When ambiguous, freeze broader**: If the boundary is unclear, freeze the larger scope. Narrowing the freeze breaks the defense. Violation → freeze declaration becomes meaningless.

3. **Stop immediately after declaration**: No next action after printing FROZEN SCOPE. The user will issue the implementation request. Violation → scope awareness fades when declaration and implementation are mixed.

4. **Unfreeze requires explicit user approval**: The AI never unfreezes a file on its own. Even if modification is clearly needed, stop and ask. Violation → the freeze was advisory, not enforced.

---

## Rationalization Table

| Rationalization | Counter |
|----------------|---------|
| "It's frozen but I only need to change 1 line" | Invariant 1. Stop and report to user. |
| "Scope is vague, I'll just freeze narrowly" | Invariant 2. Ambiguous → broader. |
| "I'll declare freeze and start implementing right away" | Invariant 3. Declare, then stop. |
| "It's just 1 file, why do I need a freeze declaration?" | Discard If: 1 file under 10 lines is excluded. Otherwise declare. |
| "This frozen file obviously needs updating too" | Invariant 4. Stop, report, get explicit unfreeze approval. |
| "I'm not going to modify anything, so it's fine" | Thinking you won't modify anything is exactly when scope creep starts. |

---

## Examples

### Single file
```
User: "/freeze src/auth/login.py"

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔒 FROZEN SCOPE — login.py error handling fix
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ EDITABLE
  - src/auth/login.py

❌ FROZEN
  - Everything outside the file above

⚠️  READ-ONLY
  - src/auth/models.py (type reference only)

Rules:
- No Edit/Write on any file except login.py
- If modification need is discovered → stop, report to user
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Multi-file with glob
```
User: "/freeze src/api/routes/** and src/api/middleware/auth.ts"

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔒 FROZEN SCOPE — API route refactor + auth middleware
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ EDITABLE
  - src/api/routes/users.ts
  - src/api/routes/products.ts
  - src/api/routes/orders.ts
  - src/api/routes/index.ts
  - src/api/middleware/auth.ts

❌ FROZEN
  - Everything outside the files above
  - src/api/middleware/cors.ts (not in scope)
  - src/api/middleware/logging.ts (not in scope)

⚠️  READ-ONLY
  - src/db/models/ (schema reference only)
  - src/types/ (type definitions reference)

Rules:
- No Edit/Write on any file outside the EDITABLE list
- If modification need is discovered → stop, report to user
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## Pair with /brief

`/brief` locks WHAT to build and what NOT to build (scope in/out). `/freeze` locks WHICH FILES to touch. Use brief before design, freeze before implementation.

```
/brief "add user search"     → scope IN: search endpoint, scope OUT: existing filters
/freeze src/api/search.ts    → only this file is editable
  implement
/pre-push                    → ship it
```

## Difference from gstack original

gstack freeze is global session file-list based. This version is task-scoped declaration based (no session state persistence).
Reason: Claude Code cannot persist state across sessions → the declaration block itself serves as in-context state.
