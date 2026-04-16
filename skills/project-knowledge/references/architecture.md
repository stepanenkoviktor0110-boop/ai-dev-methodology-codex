# Architecture

## Purpose
Technical architecture overview for AI agents. Helps agents understand HOW the system is built.

---

## Tech Stack

**Runtime:** No traditional runtime — the framework consists of Markdown files, YAML templates, and Bash scripts consumed by Codex.

**Format:** Markdown (SKILL.md per skill, agent .md files, templates, documentation)

**Templates:** YAML (interview.yml), Markdown (.md.template files)

**Scripting:** Bash (shared/scripts/ — e.g., init-feature-folder.sh)

**Version control:** Git + GitHub (two repos: Claude Code version and Codex adaptation)

---

## Project Structure

The repository root (`$AGENTS_HOME/skills/`) contains:

- `{skill-name}/SKILL.md` — one directory per skill with its instruction file
- `shared/scripts/` — Bash utilities (e.g., `init-feature-folder.sh`)
- `shared/templates/new-project/` — scaffold for bootstrapping new projects
- `shared/work-templates/` — per-feature artifact templates (`user-spec.md.template`, `tech-spec.md.template`, `session-plan.md.template`, `sketch.md`, `tasks/`)
- `project-knowledge/references/` — this documentation
- `quick-learning/references/` — knowledge buffer files: `reasoning-patterns.md`, `triad-index.md`, `quick-ref-{skill}.md`
- `work/{feature-name}/` — per-feature working directory with `user-spec.md`, `tech-spec.md`, `tasks/`, `decisions.md`, `logs/userspec/interview.yml`

---

## Key Components

**Skills** — each is a directory with `SKILL.md` containing instructions the agent follows when the skill is invoked. Skills are loaded by reading and following $AGENTS_HOME/skills/{skill}/SKILL.md.

**Agents** — `agents/` directory contains agent definition files (e.g. `sketch-interviewer.md`). Agents are spawned directly via `spawn_agent`. Validators are separate: launched via the spawn_agent tool.

**Shared templates** — reusable scaffolds for work artifacts: user-spec, tech-spec, session-plan, tasks. Copied and edited per feature, never modified in place.

**Quick-learning system** — `reasoning-patterns.md` is a triad-based buffer (Goal / Context / Decision). New patterns added via `/quick-learning`, promoted into skills when `Seen >= 4`. `triad-index.md` prevents duplicates.

**Project knowledge** — per-project documentation in `.agents/skills/project-knowledge/references/`. Read by agents at start of each skill execution for project context.

---

## External Integrations

**GitHub (`stepanenkoviktor0110-boop/ai-dev-methodology`)**
- Purpose: Version control and distribution of Claude Code methodology
- Auth method: HTTPS, credentials via git credential manager

**GitHub (`stepanenkoviktor0110-boop/ai-dev-methodology-codex`)**
- Purpose: Codex-adapted version of the methodology
- Key difference: paths use `.agents/` instead of `.claude/`, agent API uses `spawn_agent`/`wait_agent`

---

## Platform Differences: Claude vs Codex

| Aspect | Claude Code (this repo) | Codex |
|--------|------------------------|-------|
| Skills path | `~/.claude/skills/` | `$AGENTS_HOME/skills/` |
| Agent API | Built-in `Agent` tool | `spawn_agent` / `wait_agent` |
| Agents dir | `agents/` directory (interview agents, e.g. sketch-interviewer) | `agents/` directory (spawned via spawn_agent) |
| Config | `~/.claude/settings.json` | `$AGENTS_HOME/config.toml` |
| Models | Claude (Opus/Sonnet/Haiku) | GPT-5.x tiers |

---

## Data Flow

User invokes a skill → agent reads `{skill}/SKILL.md` → skill may spawn subagents (validators, researchers) → work artifacts written to `work/{feature}/` → knowledge distilled to `quick-learning/references/reasoning-patterns.md` → patterns promoted back into skills.

---

## Data Model

No database. All state is in files:
- `work/{feature}/` — ephemeral per-feature working data
- `quick-learning/references/` — persistent knowledge accumulation
- `project-knowledge/references/` — stable project documentation

### Moneymaker Data Storage (`~/.moneymaker/`)

Separate storage tree for the moneymaker pipeline (not inside the methodology repo):

```
~/.moneymaker/
  config.yml                          # Global config: hourly_rate, hosting tiers, agent_costs, billing, catalog
  projects/
    {project-name}/
      context.md                      # Accumulated project context: Требования / Договорённости / Открытые вопросы
      materials/
        {timestamp}.md                # Raw ingested material (transcript, chat, etc.)
      expand-output.md                # Cached output of /moneymaker-expand (overwritten on each run)
      overrides.yml                   # Per-project rate override (optional, created only after explicit confirmation)
      kp-{timestamp}.md               # Final quote (KP) as markdown table
```

Key invariants:
- `config.yml` is never written by any skill except `moneymaker-setup`
- `overrides.yml` is never written without explicit user confirmation
- `context.md` is only written after all conflicts resolved (never partial writes)
- `expand-output.md` is always overwritten, never appended
