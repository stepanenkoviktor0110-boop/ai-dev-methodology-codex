---
name: feature-execution
description: |
  Orchestrate feature delivery by waves: spawn workers, run reviewers,
  manage review cycles (max 3 rounds), commit per wave.

  Use when: "выполни фичу", "do feature", "execute feature", "запусти фичу",
  "выполни все задачи", "execute all tasks"
---

# Feature Execution

Orchestrate work as a single coordinator. Use `spawn_agent`, `wait_agent`, `send_input`, `close_agent`.
Do not rely on team APIs or direct agent-to-agent messaging.

Before starting, check [lessons-learned.md](references/lessons-learned.md) for known pitfalls (if file exists).

## Model Profiles

Use model tiers from [model-profiles.md](../tech-spec-planning/references/model-profiles.md):
- Worker default: `tier_opus` (`gpt-5.4`, fallback `gpt-5.3-codex`)
- Reviewer default: `tier_sonnet` (`gpt-5.4-mini`, fallback `gpt-5.3-codex`)
- Cheap routine work: `tier_haiku` (`gpt-5.4-mini` with `reasoning_effort: low`, fallback `gpt-5.1-codex-max`)

## Phase 1: Initialization

1. Read `work/{feature}/tech-spec.md` and `work/{feature}/user-spec.md`.
2. Read `work/{feature}/logs/session-plan.md` if it exists. If missing, treat all waves as one session.
3. Read all task frontmatters in `work/{feature}/tasks/`:
   - `status`, `wave`, `depends_on`, `skills`, `reviewers`, `verify`, optional `worker_name`.
4. Build execution plan from `$AGENTS_HOME/shared/work-templates/execution-plan.md.template`.
5. Save plan to `work/{feature}/logs/execution-plan.md`, show user, wait for approval.
6. Initialize/update `work/{feature}/logs/checkpoint.yml` with `total_waves`, `current_session`, `total_sessions`.

## Phase 2: Execute Wave

1. Select tasks for current wave: `status: planned` and dependencies done.
2. Set each selected task to `in_progress`.
3. For each task, run worker flow:
   - Spawn worker (`agent_type: worker`, model from `tier_opus`).
   - Pass task path and feature context files.
   - Worker performs implementation and local verification, then reports:
     - modified files
     - commits made
     - unresolved risks
4. Review flow per task (if `reviewers` not empty):
   - Coordinator gets `git diff` for task changes.
   - Spawn all reviewer agents in parallel (`tier_sonnet`), pass:
     - task path
     - spec paths
     - changed files list
     - diff text
     - output path: `logs/working/task-{N}/{reviewer}-round{R}.json`
   - Collect reports, apply valid findings, rerun tests.
   - Repeat up to 3 rounds.
5. If `reviewers` is empty, mark task self-verified after worker checks.
6. Ensure worker writes a concise decisions entry to `work/{feature}/decisions.md`.

## Audit Wave

Audit Wave tasks use `reviewers: []` and run as independent auditors:
- Code audit (`code-reviewing`)
- Security audit (`security-auditor`)
- Test audit (`test-master`)

Spawn all auditors in parallel (`tier_opus`).
If findings exist, spawn a fixer worker (`tier_opus`) and re-run only affected auditors as reviewers (`tier_sonnet`), max 3 rounds.

## Phase 3: Wave Transition

1. Verify decisions entries exist and include verification results.
2. Update tasks `in_progress` -> `done`.
3. Commit status/decision updates:
   - `chore: complete wave {N} — update task statuses and decisions`
4. Update checkpoint:
   - `last_completed_wave`, `next_wave`, task statuses.
5. Session boundary rule:
   - If current wave is last in current session: generate `work/{feature}/logs/next-session-prompt.md`, show it, then stop.
   - Else continue to next wave.

## Phase 4: User Review

1. Show built scope, key decisions, QA summary.
2. Ask user for manual checks listed in execution plan.
3. If issues found, run fix/review loop (max 3 rounds).
4. When approved:
   - close active agents,
   - delete `work/{feature}/logs/checkpoint.yml`.

## Escalation

Escalate to user when:
- 3 fix rounds exhausted
- requirement ambiguity blocks implementation
- required external service/tool is unavailable

On escalation:
1. Stop blocked task.
2. Report attempts and unresolved items.
3. Write decisions entry.
4. Commit: `chore: escalate task {N} — unresolved after 3 fix rounds`.

## Self-Verification

- [ ] Execution plan approved
- [ ] All tasks executed with required reviews
- [ ] All waves committed
- [ ] User review complete
