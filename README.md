# AI-First Development Methodology v1.2 — Codex Version

Structured AI-First development methodology for [OpenAI Codex CLI](https://github.com/openai/codex). Every feature goes through a spec-driven pipeline with automated validators and quality gates at each stage.

## What It Does

A complete development framework where AI agents handle the full cycle: requirements → architecture → tasks → code → review → documentation. You guide the process, agents do the work.

**Problems it solves:**
- **Context loss between sessions** — distributed knowledge base persists across sessions
- **Quality without human review** — automated validators at every stage
- **Scope creep** — specs approved before coding starts
- **Outdated library knowledge** — Context7 MCP fetches current docs

## Installation

### Prerequisites

- [Codex CLI](https://github.com/openai/codex) installed and configured
- [PowerShell 7+](https://github.com/PowerShell/PowerShell) (`pwsh`) for skill dispatcher
- [GitHub CLI](https://cli.github.com/) (`gh`) for project initialization
- Git Bash on Windows (for shell scripts)

### Step 1: Clone the framework

```bash
git clone https://github.com/stepanenkoviktor0110-boop/ai-dev-methodology-codex.git ~/.agents
```

This places all skills, agents, and templates where Codex expects them (`~/.agents/`).

### Step 2: Configure Codex

Add to `~/.codex/config.toml`:

```toml
model = "gpt-5.4"

# SessionStart hook for checkpoint recovery (optional, for long features)
[[hooks]]
event = "SessionStart"
command = "cat work/*/logs/checkpoint.yml 2>/dev/null || echo 'No active checkpoint'"
```

### Step 3: Configure MCP (optional but recommended)

Add [Context7](https://github.com/upstash/context7) MCP server for up-to-date library documentation. In `~/.codex/config.toml`:

```toml
[mcp_servers.context7]
command = "npx"
args = ["-y", "@upstash/context7-mcp"]
```

### Step 4: Verify installation

```powershell
pwsh -File ~/.agents/shared/scripts/smoke-codex-compat.ps1
# Expected output: PASS
```

## Usage

### New Project (from scratch)

```
/init-project                  # Template + git + GitHub repo
/init-project-knowledge        # Interview → fill all project documentation
```

After this you'll have:
- GitHub repo with `master` + `dev` branches
- `.agents/skills/project-knowledge/` filled with project docs
- `AGENTS.md` pointing to your knowledge base

### New Feature (full pipeline)

```
/new-user-spec                 # Step 1: Interview → user-spec.md (requirements)
                               #   ⛔ user approves spec
/new-tech-spec                 # Step 2: Research → tech-spec.md (architecture)
                               #   ⛔ user approves spec
/decompose-tech-spec           # Step 3: Break into tasks → tasks/*.md
                               #   ⛔ GATE 1: user approves task decomposition
                               #   ⛔ GATE 2: user approves session plan (LOC budget)
                               #   ⛔ HARD STOP — no auto-transition to code
/do-feature                    # Step 4: Execute tasks by waves
                               #   ⛔ GATE 3: user confirms session scope + LOC before start
                               #   ⛔ GATE 4: session end → report + handoff prompt + STOP
/retrospective                 # Step 5: Extract lessons → update skills
/done                          # Step 6: Update project docs → archive feature
```

Each step has validators and **blocking gates** — no step proceeds without explicit user approval:

| Step | Command | Validators | Gates | Output |
|------|---------|-----------|-------|--------|
| Requirements | `/new-user-spec` | quality + adequacy (2) | user approves spec | `user-spec.md` |
| Architecture | `/new-tech-spec` | skeptic + completeness + security + test + template (5) | user approves spec | `tech-spec.md` |
| Tasks | `/decompose-tech-spec` | template + reality (2) | user approves tasks, then session plan | `tasks/*.md` + `session-plan.md` |
| Code | `/do-feature` | code + security + test reviewers (3) | session scope confirmed, STOP at session end | commits |
| Audit | (auto, last waves) | holistic code + security + test audit (3) | — | audit reports |
| QA | (auto, final wave) | pre-deploy + deploy + post-deploy | — | verified feature |

### Single Task (manual control)

```
/do-task                       # Execute one task with quality gates
```

Use when you want to debug, iterate, or manually control execution.

### Ad-hoc Coding (no spec)

```
/write-code                    # TDD cycle: plan → tests → code → review
```

Quick coding with automatic code review. No specs needed.

### Other Commands

| Command | Purpose |
|---------|---------|
| `/init-project-knowledge` | Fill project documentation via interview |
| `/retrospective` | Extract lessons learned, update skills |
| `/done` | Finalize feature, update docs, archive |

## How It Works

### Project Structure

```
your-project/
├── .agents/                        # AI agent context (auto-created by /init-project)
│   └── skills/
│       └── project-knowledge/      # Your project's knowledge base
│           ├── SKILL.md
│           └── references/
│               ├── project.md      # Purpose, audience, scope
│               ├── architecture.md # Tech stack, structure, data model
│               ├── patterns.md     # Code conventions, testing, business rules
│               └── deployment.md   # Platform, CI/CD, monitoring
├── work/                           # Feature work items
│   ├── my-feature/
│   │   ├── user-spec.md           # Requirements (human-readable)
│   │   ├── tech-spec.md           # Architecture (for agents)
│   │   ├── decisions.md           # Decisions made during implementation
│   │   ├── tasks/                 # Atomic task files
│   │   └── logs/                  # Session plans, checkpoints, review reports
│   └── completed/                 # Archived finished features
├── AGENTS.md                      # Minimal — points to project-knowledge
└── README.md
```

### Global Framework (`~/.agents/`)

```
~/.agents/                          # This repository
├── skills/                        # 20+ skills (methodology, execution, quality)
├── agents/                        # 20 agents (validators, reviewers, creators)
├── shared/
│   ├── work-templates/            # Templates for specs, tasks, sessions
│   ├── templates/                 # New project scaffolding
│   ├── interview-templates/       # Interview plans (YAML)
│   ├── scripts/                   # Dispatcher, smoke test, init scripts
│   └── runtime/                   # MCP aliases
└── AGENTS.md                      # Global preferences
```

### Key Principles

- **Spec-Driven** — write specs before code. Hierarchy: User Spec → Tech Spec → Tasks → Code
- **Blocking Gates** — 6 mandatory HARD STOPs in the pipeline. No step proceeds without explicit user approval. Agent never auto-transitions between decomposition → execution or between sessions
- **Multi-level Validation** — automated validators at every stage (2 → 5 → 2 → 3)
- **Session Planning** — waves grouped by ~1200 LOC budget per session (fits context window). LOC budget enforced as a communication gate before each session/task
- **Session Handoff** — at session end: structured report (done/not done/checks/risks) + generated prompt for next session. Execution stops until user explicitly starts next session
- **Checkpoint Recovery** — `checkpoint.yml` persists state; SessionStart hook resumes after compaction
- **Just-In-Time Context** — agents read only what's needed for current task
- **Retrospective** — lessons learned embedded back into skills after each feature

### Model Tiers

| Tier | Model | Use |
|------|-------|-----|
| `tier_opus` | `gpt-5.4` | Complex architecture, security-critical work |
| `tier_sonnet` | `gpt-5.4-mini` | Reviewers, validators, medium tasks |
| `tier_haiku` | `gpt-5.4-mini` (low reasoning) | Simple checks, formatting |

### Skills & Agents

**Skills** contain methodology (WHAT to do). **Agents** add isolation + output contracts (HOW to deliver).

| Category | Skills | Agents |
|----------|--------|--------|
| Planning | user-spec-planning, tech-spec-planning, task-decomposition, project-planning | interview-completeness-checker, task-creator |
| Execution | code-writing, feature-execution, pre-deploy-qa, post-deploy-qa | code-researcher |
| Quality | code-reviewing, security-auditor, test-master | code-reviewer, security-auditor, test-reviewer, prompt-reviewer |
| Validation | — | userspec-quality-validator, userspec-adequacy-validator, tech-spec-validator, skeptic, completeness-validator, task-validator, reality-checker |
| Meta | methodology, retrospective, documentation-writing, skill-master | documentation-reviewer, deploy-reviewer, infrastructure-reviewer, skill-checker |

For full details on any skill, read its `SKILL.md` directly:
```
cat ~/.agents/skills/{skill-name}/SKILL.md
```

## Codex Runtime Compatibility

This framework uses a Codex-compatible shim for skill alias resolution:

- **Skill dispatcher**: `shared/scripts/dispatch-skill.ps1` — maps `/new-user-spec` → `user-spec-planning`, etc.
- **MCP aliases**: `shared/runtime/mcp-aliases.json` — Context7 tool name mapping
- **Smoke test**: `shared/scripts/smoke-codex-compat.ps1` — checks for legacy Claude Code patterns

## Based on

Evolved fork of [molyanov-ai-dev](https://github.com/pavel-molyanov/molyanov-ai-dev) by Pavel Molyanov (MIT License).

## Changelog

### v1.2 — Blocking Gates (2026-03-24)

Found during real-world testing: agents skipped approval steps, ran sessions without stopping, ignored LOC budgets.

- **6 blocking gates** enforced across pipeline (task approval, session plan approval, scope confirmation, session end STOP)
- **Session End Protocol**: structured report + handoff prompt + HARD STOP
- **Session Scope Confirmation**: LOC budget table shown before every session/task start
- **Pre-flight checks** in `/do-feature` and `/do-task`: verify session-plan approved, task in current session
- **No auto-transition**: `/decompose-tech-spec` ends with STOP, user must explicitly invoke execution
- `session-plan.md` now has `status: draft/approved` frontmatter

<details>
<summary>v1.1 changes</summary>

- Fixed deploy task skill reference (`infrastructure` → `deploy-pipeline`)
- Fixed dimension count inconsistency (10 → 11) — Resource Management was missing
- Fixed validator count mismatch (6 → 5) in task-decomposition
- Added team protocol override note in code-writing skill
- Replaced documentation-writing reviewer mapping
- Removed TypeScript/JavaScript ecosystem bias (now language-agnostic)
- Added wave-conflicts to methodology validator
- Fixed Context Files paths (`$AGENTS_HOME/` → `.agents/` relative paths)
- Added LOC budget rationale, `estimated_loc` documentation
- Full Codex CLI adaptation: paths, models, frontmatter, hooks, smoke test
</details>
