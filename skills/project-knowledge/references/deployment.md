# Deployment & Operations

## Purpose
How the methodology is distributed, installed, and updated.

---

## Deployment Model

**Platform:** GitHub + local git clone

**Type:** Not a deployed application. The methodology is installed by cloning the repo into `$AGENTS_HOME/skills/`. Updates arrive via `git pull`.

**Why:** The agent reads skill files from `$AGENTS_HOME/skills/` at runtime. Cloning the repo directly into that location makes all skills immediately available with no additional setup.

---

## Distribution

**GitHub repo:** `https://github.com/stepanenkoviktor0110-boop/ai-dev-methodology`

**Install:** clone `stepanenkoviktor0110-boop/ai-dev-methodology-codex` into `$AGENTS_HOME/skills`.

**Update:** run `git pull origin master` from `$AGENTS_HOME/skills`. Auto-checked on first pipeline command per session (RULE #0 in AGENTS.md).

---

## Codex Variant

**GitHub repo:** `https://github.com/stepanenkoviktor0110-boop/ai-dev-methodology-codex`

**Local clone:** `C:/tmp/ai-dev-methodology-codex/`

**Install (Codex):** clone `stepanenkoviktor0110-boop/ai-dev-methodology-codex` into `$AGENTS_HOME/skills`.

---

## Update Workflow

1. Edit skill files directly in `$AGENTS_HOME/skills/`
2. `git add -A && git commit -m "..."` from `$AGENTS_HOME/skills/`
3. `git push origin master` → available to all installs on next `git pull`
4. Codex adaptation: manually diff changed skills, apply `.claude/` → `.agents/` substitution, push to Codex repo

---

## Environments

**Production:** `$AGENTS_HOME/skills/` on developer's machine — this is what the agent uses daily.

---

## Monitoring

No automated monitoring. Verification is manual: invoke a skill after changes, observe behavior matches intent.
