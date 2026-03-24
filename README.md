# AI-First Development Methodology v1.1

Structured AI-First development methodology for Claude Code CLI. Every feature goes through a spec-driven pipeline with automated validators and quality gates at each stage.

## Based on

This is an evolved fork of [molyanov-ai-dev](https://github.com/pavel-molyanov/molyanov-ai-dev) by Pavel Molyanov — the original AI-First Development Methodology for Claude Code.

## What's new in v1.1

Changes from code review of the entire methodology:

- **Fixed** deploy task skill reference (`infrastructure` → `deploy-pipeline`) in tech-spec template
- **Fixed** dimension count inconsistency (10 → 11) in methodology — Resource Management was missing from count
- **Fixed** validator count mismatch (6 → 5) in task-decomposition
- **Added** `name: do-task` to do-task skill frontmatter
- **Added** team protocol override note in code-writing skill (prevents conflict with feature-execution reviewer flow)
- **Added** `Bash` to code-reviewer agent tools (enables cross-file verification via `tsc --noEmit`, linters)
- **Replaced** documentation-writing reviewer: `code-reviewer` → `documentation-reviewer` in skills-and-reviewers mapping
- **Removed** TypeScript/JavaScript ecosystem bias from code-reviewer agent (now language-agnostic)
- **Removed** prompt-master duplication from Meta Skills table (kept in Execution Skills only)
- **Standardized** lessons-learned.md link wording in retrospective skill
- **Added** wave-conflicts to methodology validator description
- **Fixed** Context Files paths in task template (`~/.claude/` → `.claude/` relative paths)
- **Added** LOC budget rationale in methodology (empirically sized for session context window)
- **Added** `estimated_loc` documentation in tech-spec-planning
- **Added** E2E conditional comment in user-spec template
- **Added** Git Bash requirement comment in init-feature-folder.sh

## Pipeline

```
/new-user-spec → /new-tech-spec → /decompose-tech-spec → /do-feature → /retrospective → /done
```

| Stage | Validators | Output |
|-------|-----------|--------|
| User Spec | quality + adequacy | `work/{feature}/user-spec.md` |
| Tech Spec | skeptic + completeness + security + test + template | `work/{feature}/tech-spec.md` |
| Tasks | template + reality | `work/{feature}/tasks/*.md` |
| Code | code-reviewer + security-auditor + test-reviewer | commits |
| Audit Wave | code + security + test (holistic) | audit reports |
| Final Wave | pre-deploy QA + deploy + post-deploy | verified feature |

## Structure

```
skills/          # 20+ skills (planning, execution, quality, meta)
agents/          # 20 agents (validators, reviewers, creators, QA)
shared/
  work-templates/   # Templates for specs, tasks, sessions
  templates/        # New project scaffolding
  interview-templates/  # Interview plans (YAML)
  scripts/          # Shell scripts
CLAUDE.md        # Global preferences
```

## Key Concepts

- **Spec-Driven Development** — specs approved before code starts
- **Multi-level Validation** — automated validators at every stage (2 → 5 → 2 → 3 → 3)
- **Session Planning** — waves grouped by ~1200 LOC budget per session
- **Checkpoint Recovery** — resume after context compaction
- **Retrospective** — lessons learned embedded back into skills
- **Just-In-Time Context** — agents read only what's needed

## Quick Start

### New project
```
/init-project → /init-project-knowledge
```

### New feature
```
/new-user-spec → /new-tech-spec → /decompose-tech-spec → /do-feature → /done
```

### Ad-hoc coding
```
/write-code
```

## License

Based on [molyanov-ai-dev](https://github.com/pavel-molyanov/molyanov-ai-dev) (MIT License).
