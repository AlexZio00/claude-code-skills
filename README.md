# claude-code-skills

A collection of practical Claude Code skills built from real production experience.

## Skills

### `/project-init` — New Project Setup Interview

Conversational interview that generates `CLAUDE.md` + `DEVELOPMENT_ROADMAP.md` before you write a single line of code.

**What it does:**
- Asks 8 focused questions (one at a time) to lock in stack, data layer, deployment, AI/LLM strategy, and hard rules
- Generates a `CLAUDE.md` with Hard Rules, Secrets Policy, and Dev Conventions tailored to your project
- Outputs a Phase-structured `DEVELOPMENT_ROADMAP.md`
- Suggests the right folder structure for your language (Python, TypeScript, Java/Kotlin, Go, Rust, Swift)

**Why it matters:**
Most projects skip the "invariants first" step. By the time you add hard rules, the codebase already violates them. This skill forces the conversation upfront.

**Supports:** Python · TypeScript (Next.js / API) · Java · Kotlin (Spring Boot / Android) · Go · Rust · Swift

---

## Installation

Copy the skill folder into your Claude Code skills directory:

```bash
# macOS / Linux
mkdir -p ~/.claude/skills/project-init
cp project-init/skill.md ~/.claude/skills/project-init/skill.md

# Windows
mkdir "%USERPROFILE%\.claude\skills\project-init"
copy project-init\skill.md "%USERPROFILE%\.claude\skills\project-init\skill.md"
```

Then invoke it in any Claude Code session:

```
/project-init
```

Or with an existing brief/notes file:

```
/project-init
[paste your project idea or point to a file]
```

---

## Usage

Claude will interview you one question at a time:

1. **Core definition** — what, who, why
2. **Language / runtime** — with decision criteria
3. **Data layer** — DB, external API, files, or none
4. **Interface** — CLI, web, API, background service
5. **Deployment** — local, cloud, hybrid, mobile
6. **AI / LLM** — cloud vs local, cost gating
7. **Hard rules** — invariants that must never be broken
8. **Scope & timeline** — determines how much structure to generate

After the interview, Claude presents a stack summary for your approval, then generates the files.

---

## Principles Embedded

These came from painful experience on a large production system:

- **CLAUDE.md before code** — re-explaining context every session is expensive
- **Hard Rules from day one** — retrofitting them means existing code may already be in violation
- **Feature flags default OFF** — unfinished features affecting default behavior makes debugging painful
- **UI reads DB only** — direct external API calls push rate limits and errors into the UI layer
- **Append-only logs** — overwriting logs destroys the audit trail
- **Explicit secrets policy** — one `.env` commit compromises even private repos

---

## License

MIT
