---
name: "token-audit"
description: "Measures your actual Claude Code token overhead and generates a personalized infographic. Triggers: '/token-audit', 'token waste audit', 'token overhead', 'how much of my context is overhead', 'audit my token usage'. Discard if: you just want usage cost totals (use /ccusage instead)."
license: "MIT"
metadata:
  version: "1.0.0"
  author: "coinangel"
---

# Token Waste Audit

**Dominant variable**: Actual measurement from your JSONL session data — estimates without running the code are not audits.

**Discard if**: You want cost totals or billing data only. This skill measures *overhead structure*, not spend.

## Prerequisites

| Requirement | Used in | Install |
|-------------|---------|---------|
| `python3` | Steps 1–4 | Ships with macOS/Linux; [python.org](https://python.org) for Windows |
| `matplotlib` | Step 4 (infographic) | `pip install matplotlib` — **optional**, Step 4 has a text-only fallback |
| `ccusage` | Step 4 (cost stat) | `npm install -g ccusage` — **optional**, falls back to estimate |

> **Zero-dependency path**: Steps 1–3 + 5–6 require only `python3` (standard library). Skip Step 4 if you don't want the infographic — the findings in Steps 5–6 are the same.

> **ccusage (optional)**: If `npx ccusage` is available, actual spend for the last 7 days is pulled automatically and shown in the infographic. If not, the cost stat falls back to a JSONL-based estimate (usually a significant undercount due to large-session filtering).

---

## Step 1: Measure your setup

Run these measurements and collect the raw data:

```bash
# CLAUDE.md word counts
for f in ~/.claude/CLAUDE.md $(find . -maxdepth 1 -name CLAUDE.md 2>/dev/null); do
  echo "$(wc -w < "$f" 2>/dev/null || echo 0) words: $f"
done

# Rules auto-loaded
find ~/.claude/rules -name "*.md" 2>/dev/null | while read f; do
  echo "$(wc -w < "$f") words: $(basename $f)"
done | sort -rn | head -15

# Skills inventory
for d in ~/.claude/skills/*/; do
  name=$(basename "$d")
  words=$(wc -w < "$d/SKILL.md" 2>/dev/null || echo 0)
  echo "$words $name"
done | sort -rn | head -15

# Settings: hooks, MCPs, plugins
python3 -c "
import os, json
try:
    s = json.load(open(os.path.expanduser('~/.claude/settings.json')))
    print('Hooks:', list(s.get('hooks',{}).keys()))
    print('MCPs:', list(s.get('mcpServers',{}).keys()))
    print('Plugins:', s.get('enabledPlugins',[]))
except Exception as e:
    print('settings.json:', e)
"
```

## Step 2: Analyze session JSONL

Find the 5 most recent JSONL files in `~/.claude/projects/` and analyze tool usage patterns:

```python
import json, glob, os, collections

base = os.path.expanduser('~/.claude/projects')
files = sorted(glob.glob(f'{base}/**/*.jsonl', recursive=True),
               key=os.path.getmtime, reverse=True)[:5]

tool_counts = collections.Counter()
read_files  = collections.Counter()
skill_loads = collections.Counter()
agent_types = collections.Counter()
turns = 0

for f in files:
    for line in open(f, encoding='utf-8', errors='ignore'):
        try:
            e = json.loads(line)
            if e.get('type') != 'assistant': continue
            turns += 1
            for block in e.get('message',{}).get('content',[]):
                if not isinstance(block, dict): continue
                if block.get('type') != 'tool_use': continue
                name = block.get('name','')
                inp  = block.get('input',{})
                tool_counts[name] += 1
                if name == 'Read':
                    read_files[os.path.basename(inp.get('file_path',''))] += 1
                elif name == 'Skill':
                    skill_loads[inp.get('skill','?')] += 1
                elif name == 'Agent':
                    agent_types[inp.get('subagent_type','?')] += 1
        except: pass

n = len(files)
print(f'Sessions: {n}, Turns: {turns:,}')
print('\nTop 15 tools:')
for k,v in tool_counts.most_common(15): print(f'  {v:>5} {k}')
print('\nTop 15 read files:')
for k,v in read_files.most_common(15): print(f'  {v:>4} {k}')
print('\nSkills loaded:')
for k,v in skill_loads.most_common(): print(f'  {v:>4} {k}')
print('\nAgent types:')
for k,v in agent_types.most_common(10): print(f'  {v:>4} {k}')
```

## Step 3: Compute token overhead

The script uses **turn-weighted averaging** — not a simple mean across sessions.

**Why turn-weighted?**
A 2-turn session and a 100-turn session are not equal samples. The short session
has almost no productive output but carries the full fixed overhead (user config,
rules, tool definitions). Averaging them equally inflates the apparent overhead %.
Turn-weighting gives long sessions proportionally more influence:

```
weight_i = turns_i / total_turns
result   = Σ(value_i × weight_i)
```

**Fixed overhead** (same every session — not weighted):
- `config_baseline` = (rules_words + config_words) × 1.33
- `tool_definitions`   = 15,000 base (native Claude Code tools) + 2,000 × MCP count
- `plugin_autoload`    = 1,000 per enabled plugin

**Variable overhead** (turn-weighted per session):
- `memory_rereads`  = Σ(reads of MEMORY/handoff/lessons/STATE × 4,000 tokens × weight_i)
- `skill_loading`   = Σ(skill_calls × avg_skill_words × 1.33 × weight_i)
- `agent_overhead`  = Σ(agent_spawns × 3,000 × weight_i)
- `hook_injection`  = fixed part (per-session hooks × 300) + variable (per-turn/per-agent hooks, weighted)
- `cache_miss`      = non-zero only when hit_rate < 0.5; penalises cache_creation × waste_fraction × 0.25

**Denominator** = `overhead_total + productive_tokens` (synthetic budget).
Do NOT use `input + cache_creation` as denominator — cache_creation tokens are an
investment (amortised across future turns), not overhead. Using them inflates the
denominator and makes structural overhead appear negligibly small.

| Category | Type | Formula |
|----------|------|---------|
| config_baseline | fixed | (rules_words + config_words) × 1.33 |
| tool_definitions | fixed | 15,000 + mcps × 2,000 |
| plugin_autoload | fixed | plugins × 1,000 |
| memory_rereads | variable/weighted | Σ(mem_reads_i × 4,000 × w_i) |
| skill_loading | variable/weighted | Σ(skill_calls_i × avg_words × 1.33 × w_i) |
| agent_overhead | variable/weighted | Σ(agent_spawns_i × 3,000 × w_i) |
| hook_injection | mixed | per-session hooks × 300 + Σ(per-turn/agent hooks × w_i) |
| cache_miss | variable/weighted | 0 when hit_rate ≥ 0.5 |

## Step 4: Generate infographic

> **Optional** — if you prefer text-only output or `matplotlib` is not installed, skip to Step 5. The findings are identical; only the visual is missing.

Check whether matplotlib is available before running:
```bash
python3 -c "import matplotlib; print('matplotlib', matplotlib.__version__)" 2>/dev/null || echo "matplotlib not installed — run: pip install matplotlib"
```

If not installed and you want the infographic: `pip install matplotlib`

Create a dark-theme figure (20×11 inches, 150 dpi) with 6 panels:

1. **Stats card** — Turns, Sessions, Overhead %, Productive %
2. **Pie chart** — Token budget breakdown by overhead category
3. **Comparison bar** — Your profile vs. benchmark (config 14%, history re-reads 13%, hook injection 11%, cache misses 10%, skill loading 7%, tool definitions 6%)
4. **Tool call distribution** — Horizontal bar; green = productive, orange = overhead-linked
5. **File re-read heatmap** — Most-read files; red = memory/config files, green = code files
6. **Quick wins** — Top 5 actions with estimated token savings

Style: `facecolor='#0D1117'`, cards `'#161B22'`, accent blue/orange/green/red/purple

Save to: `outputs/token_audit_infographic.png`

Open with:
- macOS: `open outputs/token_audit_infographic.png`
- Windows: `explorer.exe outputs/token_audit_infographic.png`

## Step 5: Report findings

Answer these questions based on the measured data:

1. What's your actual overhead % vs. the article's 73% benchmark?
2. Which single category wastes the most tokens?
3. How many times does the same memory file get re-read in a typical session?
4. Which skill gets loaded most often — is it proportionate to its value?
5. What's the ratio of Agent spawns to actual productive Bash/Edit calls?

## Step 6: Top 3 quick wins

For each, state: current cost (tokens/session) → target cost → what to change → 5-minute implementation.

---

## Benchmark reference

| Setup type | Typical overhead | Productive % |
|-----------|-----------------|--------------|
| Minimal (no skills, few rules) | 15–25% | 75–85% |
| Standard (5–10 skills, rules) | 35–50% | 50–65% |
| Heavy (20+ skills, agent system) | 50–65% | 35–50% |
| Over-engineered | 70%+ | <30% |

> Source: 430-hour analysis of Claude Code token overhead patterns.

---

## Common findings

Most users discover:
- **MEMORY.md and session files re-read 10–100×** per session — use `#on-demand` tags to load sections only when needed
- **Session-checkpoint loaded every session** — verify the skill isn't re-reading itself in full each time
- **Agent spawns 20–30×** — many are general-purpose searches that could be replaced with lighter `Grep`/`Glob` calls
- **Rules files 8K–15K tokens** auto-loaded every session — consider splitting into hot rules (always load) vs. cold rules (load on demand)

---

## Invariants

1. **Run the code, don't estimate** — skip the bash/python steps and the output is not an audit. Dominant variable applies.
2. **outputs/ directory** — create it if it doesn't exist before saving the infographic.
3. **Report all 5 questions** — partial findings skip the categories that are usually the biggest surprises.
