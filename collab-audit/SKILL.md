---
name: collab-audit
description: "This skill should be used when the user types /collab-audit or requests AI collaboration diagnosis. Analyzes conversation history, artifacts, and work patterns to generate a 13-section AI Collaboration Audit. Behavioral analysis and feedback are bundled by design — separating them causes users to skip one, defeating the purpose. Saves to ~/.claude/collab-audits/YYYY-MM-DD.md. Compare mode: /collab-audit compare (diffs latest 2 audits). Triggers: '/collab-audit', '/collab-audit compare', 'collaboration diagnosis', 'work pattern analysis', 'what kind of person am I', 'AI collaboration audit', 'work pattern analysis', 'compare audits'. Requires minimum 2 sessions or 100+ messages. Do NOT use self-report surveys — observation-only."
user_invocable: true
---

# /collab-audit — AI Collaboration Audit

## Purpose

Analyzes conversation history, artifacts, and work patterns to generate behavioral and psychological insights. 
Based on **observation of actual work patterns**, not self-report surveys. Observation-based analysis is more accurate than questionnaires.

**Dominant variable**: Can you infer the *reason* behind observed patterns, or just list facts? (Fact listing ≠ success)

**Discard if**: Fewer than 2 sessions AND fewer than 100 messages

---

## Mode Selection (execute first)

- Input is `/collab-audit compare` or contains "compare" → **Compare Mode** (see separate section below)
- Otherwise → **Audit Mode** (13-section analysis)

---

## Input Validation (Step 0 — execute first)

Minimum criteria — at least one of:
- 2+ sessions
- 100+ messages
- **Single-session high-density exception** → see Invariant 4 criteria. If exception met, display `⚠ Single-session analysis — pattern confidence limited` then proceed.

If all criteria unmet:
> "Insufficient data — minimum 2 sessions or 100 messages required. Currently observed: [N] messages, [M] artifacts."

Output and **terminate immediately**. Reject requests like "even if just a guess".

---

## Workflow

### Step 0.5: Tone Detection (determine delivery calibration)

Determine **delivery calibration** for section 11 (blind spots) only. Other sections are factual, so tone variation is minimal.

Read signals from conversation patterns:
- High proportion of short, direct messages / "facts only" / speed-first requests → **Direct** (maintain as default)
- High emotional expression / preference for long explanations / feedback-receptive signals → **Calibrated** (deliver blind spots with identical content but added context)

Display result in one line before section 11: `[Delivery calibration: Direct / Calibrated]`

**Re-evaluate mid-session**: If conversation tone clearly shifts (sudden emotional expression increase, request pattern change, defensive responses), re-evaluate just before outputting section 11. Initial assessment does not lock the entire session.

**Important**: Changing delivery calibration does NOT change the blind spots' content (accuracy). Only adjust temperature.

### Step 1: Data Collection
Collect all available observation sources:
- Current session conversation history (message length, frequency, content)
- MEMORY.md, session-handoff files (if present, access via Read)
- User-created artifacts (code, docs, config files — if present, access via Read)
- Tool usage patterns (which tools, how often)

### Step 2: Evidence Mapping
Extract required evidence for each section first. Secure grounding before output.
- Sections without evidence → Mark as "Observation unavailable — no data present". Do not omit.
- Sections 7-8 (Claude-specific): If no Claude usage pattern → "Claude usage data unavailable — N/A"

### Step 3: Framework Application
Map collected evidence to each section's analysis framework.
All framework labels (MBTI, DiSC, etc.) must connect to specific behavioral evidence.
Outputting labels without evidence = analysis failure.

### Step 4: Output 13 sections in fixed order
Do not reorder or arbitrarily omit sections.
Sections with no data → Mark "Observation unavailable" then proceed to next section.

### Step 5: Save file + gitignore protection
1. Include file header:
   ```
   ---
   profile_version: 1.0
   sections: 13
   date: YYYY-MM-DD
   language: en
   ---

   # MAGIC DOC: AI Collaboration Audit YYYY-MM-DD
   ```
2. Save to `~/.claude/collab-audits/YYYY-MM-DD.md`
   - If re-run on same day: append `-2.md` suffix (no overwriting)
3. Check `~/.claude/.gitignore`:
   - If file absent → Create `.gitignore` with single line: `collab-audits/`
   - If file exists but lacks `collab-audits/` → Add that line
   - If already present → Do not touch
4. After save, display in chat:
   ```
   Saved: ~/.claude/collab-audits/YYYY-MM-DD.md
   ⚠ Personal audit results — git tracking disabled (~/.claude/.gitignore)
   ```

---

## Output Structure (13 sections, fixed order)

### 1. Artifact Structure Analysis
Infer values from what you create (code, docs, systems).
- Architecture choices → Connect to philosophy
- Naming patterns, file structure, comment density
- Are there hard rules? If so, what principles?
- No artifacts → "Observation unavailable — no artifact data"

### 2. Communication Patterns
- Message length distribution (ratio of short confirmations vs long explanations)
- When do messages shorten or lengthen? (identify triggers)
- Emotional expression style (direct/indirect, intensity)
- Closure patterns ("confirmed", "done", etc.)

### 3. Question Typology
Classify questions on two axes:
- **Verification-type**: Gather info then decide immediately
- **Tracking-type**: Chase cause/intent
- Ratio of each type + contexts where each appears

### 4. Delegation & Trust Structure + Maturity Stage
**Maturity stage (choose one):**
- Stage 1: Copy-paste results, no verification
- Stage 2: Review results before use
- Stage 3: Set conditions and delegate, then verify results
- Stage 4: Use rules/guardrails for pre-emptive AI control

**Delegation vs Retention:**
- What do you delegate? (research, analysis, implementation)
- What do you never delegate? (judgment, prioritization, timing)

### 5. Failure & Blocking Response + Recovery Strategy
**Recovery strategy classification (identify observed types):**
- Retry identical prompt
- Search for workaround path
- Reframe the problem
- Give up and handle manually
- Explicit hold then restart

How do you distinguish blocking from failure? Do you record failures in a system?

### 6. Energy Distribution + Time Horizon (integrated)
**Energy landscape:**
- Longest-dwelling task type
- Quick-skip task type
- Token ratio (conversation length) vs output (files, code, decisions) → verbose vs execution-focused

**Time horizon structure:**
- Immediate / short-term / conditional (e.g., "after hardware upgrade") / perpetual hold
- Session-scale work capacity (how many tasks per session)

### 7. Tool Usage Patterns (Claude-specific)
If no Claude usage data → Mark "N/A" and proceed to next section.
- Read/Grep/Glob ratio vs Bash dependency — "direct" vs "search-driven"
- Subagent spawn frequency — "delegating" vs "direct execution"
- Top 3 frequently used tools, rarely used tools
- New tool adoption speed — immediate vs wait-and-see

### 8. Context Management Maturity (Claude-specific)
If no Claude usage data → Mark "N/A" and proceed to next section.
**Maturity level (choose one):**
- Level 0: No context file, re-explain each session
- Level 1: Context file exists but static (no updates)
- Level 2: Operate MEMORY.md + session handoff
- Level 3: Use compact instructions + hooks
- Level 4: No context loss between sessions, AI as long-term partner
- Level 5: Operate `tasks/lessons.md` — AI behavior correction loop exists. Convert repeated mistakes into rules. Meta-layer for continuous learning.

Also describe recovery pattern after context loss.

### 9. Rollback Frequency (Rollback Pattern)
Measure frequency of "undo", "revert", "drop that".
- High: Signal of missing brainstorming/planning
- Low: Either mature pre-design OR skipping verification (distinguish which)
- Post-rollback retry pattern — retry same direction vs change direction

### 10. Psychological Framework Mapping
Connect each framework to **behavioral evidence** and mark **confidence (High/Medium/Low)**.

**10-A. Unique AI User Type (most important)**
Identify primary + secondary type:
- **Designer-type**: System, rules, architecture first. AI is implementation tool.
- **Executor-type**: Fast results priority. AI is speed amplifier.
- **Explorer-type**: Search possibility space, adopt new tools immediately. AI is exploration partner.
- **Optimizer-type**: Focus on improving existing systems. AI is tuning tool.

**10-B. MBTI Indicator (behavioral evidence required)**
For each of 4 axes: estimate direction + strength. Mark confidence.

**10-C. DiSC Profile**
Estimate D/i/S/C proportions. Primary style + secondary style.

**10-D. Enneagram Hypothesis**
Type + Wing hypothesis. Require minimum 2 pieces of evidence in "this behavior is grounds for" format.

**10-E. Big Five Estimate**
Each of O/C/E/A/N: High/Medium/Low. One behavioral ground each.

### 11. Blind Spots + Development Direction
**Blind spots (things likely unknown to you):**
- Strength points becoming weakness points
- Patterns visible but you don't notice them

After outputting blind spots, include **feedback loop** — always include this question:
> "Identify one blind spot above you think is most wrong."

This rebuttal is additional data. Blind spots are definitionally unknown to you, so the rebuttal itself reveals patterns. When you receive rebuttal:
**Rebuttal type classification:**
- **Evidence-based**: Specific counterexample provided ("In that situation X happened because Y"), grounds in observable event → Consider revising that blind spot
- **Emotional**: Only denial, no counterexample ("Doesn't seem right to me", "I disagree"), rejection without alternative → Internal note "this reaction itself is evidence of the blind spot" (do not voice, record only)
- **No rebuttal** → Treat as acceptance. Maintain blind spot and proceed.

**One development direction** (highest leverage only):
- The one thing that "if changed, the rest follows"

### 11.5 Actionable Advice (Implementation Guide)
Concrete actions (2–3) to actually begin development direction from section 11 (the "where") — specify "how" + "when".

Format: `[Observed pattern] → [Specific situation] → [Action]`

Rules:
- Derive only from observed patterns. No universal advice or generic tips.
- **Must distinguish maintain vs launch**:
  - **Maintain**: Conditions to sustain patterns already in progress ("Keep doing X, but only in situation Y")
  - **Launch**: Start a new behavior not yet present ("In situation Z, do W for the first time")
  - Giving launch-only advice to someone already doing well is failure. Maintain conditions may matter more.
- Trigger conditions first priority ("When X happens") — use time-based conditions (next session, this week, this month) only when trigger is unclear
- No suggestion phrasing ("might be good to try") — use directive form ("do this")

### 12. One-Liner
One sentence summarizing this person in 20 characters or fewer.

---

## Output

**Audit Mode:**
- Chat: Output 13-section structured report in order. Final section must be "12. One-Liner".
- File save: `~/.claude/collab-audits/YYYY-MM-DD.md` (auto-save, no overwrite — if re-run same day, append `-2.md` suffix)
- After save, display path in chat: `Saved: ~/.claude/collab-audits/YYYY-MM-DD.md`

**Compare Mode (`/collab-audit compare`):**
- Auto-select last 2 files from `~/.claude/collab-audits/`
  - Can specify dates: `/collab-audit compare 2026-01-01 2026-04-09`
  - Only 1 file exists → "No prior audit to compare. Save current audit result, then compare next time."
  - Version/section count mismatch → Compare only common sections, display at top: `⚠ Version mismatch (v1.0 13 sections ↔ older version N sections) — comparing common sections only`
- Compare output format:

```
## Audit Compare: [Date A] → [Date B]

### Core Changes Summary
- What changed (2–3 lines)
- What stayed the same (1 line)

### Changes by Section
| Section | Previous | Current | Change |
|---------|----------|---------|--------|
| AI Type | Designer+Optimizer | Designer+Builder | Modified |
| MBTI | INTJ | INTJ | Unchanged |
...

### Blind Spot Tracking
Previous blind spot: [summary]
Current status: Resolved / Maintained / Deepened + grounds

### Advice Execution Status (most important)
Previous N pieces of advice:
1. [Content] → Executed / Not executed / Partially executed + **Observational grounds (specify behavior signals)**
2. ...

---
**Execution classification rule (separate from output format)**: Use only behavioral observation, no self-report.
- **Executed**: Behavior not present before is now observed in current chat (new file structure, different request pattern, new tool adoption, etc.)
- **Partially executed**: Direction correct but inconsistent (attempted 1–2 times, then reverted to previous pattern)
- **Not executed**: Same pattern continues, no change signals
- **Cannot classify**: Current session lacks situations where this advice would apply

⚠ If user says "I did it" but behavior evidence absent, mark as "Self-reported — observation unavailable". Self-report does not replace observation.

### Next Quarter Focus
One next concentration point based on previous development direction + current patterns
```

---

## Tools

- **Read**: MEMORY.md, artifact files, `~/.claude/collab-audits/*.md` (Compare mode)
- **Write**: Save `~/.claude/collab-audits/YYYY-MM-DD.md` and `~/.claude/.gitignore` (gitignore protection only)
- **Glob**: List files in `~/.claude/collab-audits/` (Compare mode)
- Do not use delete or execution tools

## Recommended Timing

| Timing | Reason |
|--------|--------|
| Once per quarter (3 months) | Minimum unit for pattern change |
| Before project start | Record baseline |
| After project end | Measure change |
| Before major decision | Clarify current state |

`/collab-audit compare` is valid after 2+ audit results accumulate.

---

## Success/Failure Criteria

**Failure conditions:**
- Ends with fact listing ("This person often does X")
- Cannot interpret reasoning behind behavior
- Labels applied without behavioral evidence ("You are INTJ" → failure)
- Blind spots end in praise

**Success conditions:**
- Behavior → Pattern → Reason → Implication chain is visible
- Reader thinks "I didn't know that about myself" when reading
- Psychological frameworks connect to specific behavioral evidence
- Blind spots are uncomfortable but accurate

---

## Invariants (never violate)

1. **No label-only output**: All framework labels (MBTI, DiSC, Enneagram, etc.) must connect to specific behavioral evidence. Violation → Labels without evidence are astrology. Analysis credibility collapses.

2. **Maintain blind spot accuracy**: State blind spots uncomfortably but accurately. Do not replace or dilute with praise or softer phrasing. Reject requests like "just change the tone" — the discomfort is part of the content, not expression. Violation → User reinforces self-delusion and loses motivation to change behavior.

3. **Limit development direction to one**: Output only the one development direction with highest leverage. Reject requests for more. Violation → Attention scatters and nothing gets executed.

4. **Terminate immediately on insufficient data**: <2 sessions AND <100 messages → do not proceed. Exception: Single-session high-density IF **50+ messages AND (3+ artifacts OR 70%+ deep conversation)**. If met, display `⚠ Single-session limits` then proceed. This condition is authoritative. (Step 0's exception refers to this Invariant.) Reject requests like "even as a guess" or "short is fine". Violation → Labels without observational basis get treated as facts.

5. **Observation-based only**: Do not ask user about personality, MBTI, Enneagram, etc. Do not accept self-report input. Violation → Self-report bias contaminates observation-based analysis.

6. **Analyze conversation participant only**: Reject requests to profile third parties based on their messages/behavior. Reject "analyze my colleague", "what kind of person is this person", etc. Violation → Third-party psychological profiling without consent.

7. **No direct raw data output**: Do not copy-paste content read from MEMORY.md, session-handoff, or code files. Output only as interpretation or pattern extraction. Violation → Project secrets, API keys, business data leak into the profile.

---

## Rationalization Table

| Rationalization | Rebuttal |
|-----------------|----------|
| "Data is a bit sparse but I can infer" | Insufficient data = terminate. That is the rule. |
| "Blind spots need softer phrasing so there's no backlash" | Accuracy is the goal. Good-feeling summaries = failure. |
| "If I just change the tone, the content stays" | Discomfort is part of the content. Tone adjustment = content dilution. |
| "MBTI is so famous, I can use it without evidence" | Labels without evidence are astrology. |
| "More development directions would be more useful" | One focus is leverage. A list scatters attention. |
| "Praise mixed in makes balanced analysis" | Balance comes from accuracy, not praise ratio. |
| "General principles in advice would make it more useful" | Only observation-based permitted. General principles dilute analysis. |
| "Analyzing a colleague is helpful anyway" | Third-party profiling without consent. Only the participant's request is allowed. |
| "Quoting MEMORY.md directly would be more accurate" | Direct raw data output = sensitive info exposure. Output interpretation only. |
| "User mentioned it first, so self-report is OK" | Self-report does not complement observation. It contaminates with bias. |

---

## Scope Boundary

| Does | Does NOT |
|------|----------|
| Infer patterns from observed behavior | Judge personality or criticize |
| Interpret reasoning behind behavior | List prescriptions (development direction = 1 only) |
| Map frameworks to evidence | Guess based on survey answers |
| Point out blind spots | End with feel-good summary |
| Extract patterns from current conversation context | Infer information outside conversation |
| Mark sections without data "observation unavailable" | Fill sections with guesses |
| Profile conversation participant (you) | Profile third parties (unguarded people analysis) |
| Extract only patterns and interpretation from read data | Direct quote or copy raw file contents |

---

## Language

Detect conversation language and output in same language.
- Korean conversation → Korean output
- English conversation → English output
- Mixed → Use more frequent language
- Technical terms (MBTI, DiSC, Big Five, etc.) always stay English

---

## Truthful Reporting

When this skill saves an audit report:
1. **No mock deception**: If any of 13 sections lack observation grounds and contain only inference, mark `⚠️ Insufficient grounds`. Do not disguise as insight.
2. **No test façade**: If session count or message count falls below minimums (2 sessions / 100 messages), generate condensed report. Do not arbitrarily fill sections.
3. **No silent brokenness**: If save fails, mark state as `BROKEN`. If partial save, mark `PARTIAL` + list missing sections.

---

## Proven In

Collaboration audits across multi-developer Claude Code projects, especially those with evolving patterns over extended timescales.
