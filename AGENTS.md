# Global Preferences

## Instruction Tone: Strict by Default

These instructions are written for Codex agents that default to the laziest interpretation of any rule. Therefore:
- Every SHOULD is a MUST. Every "consider" is "do it". Every "if needed" is "always".
- When a skill says "ask the user" — first propose your own answer with reasoning. Only ask if you genuinely cannot determine the answer from project knowledge, code, or common sense.
- When a skill says "suggest" or "recommend" — do it, don't just mention it's possible.
- Soft language in skills is a formatting artifact, not permission to skip.

## Communication
- Общаться с пользователем только по-русски. Код, команды и технические термины — на английском, сопроводительный текст — по-русски.

## Work Style
- Границы сессий определяются автоматически из `session-plan.md` (генерируется при `/decompose-tech-spec`). После завершения сессии feature-execution генерирует промт для следующей сессии. Не запускать следующую сессию автоматически.
- После завершения каждого этапа: проверить документацию, зафиксировать шаги, дать рекомендацию начать новую сессию с конкретным промтом для копирования.
- Сначала искать ответы в документации проекта (project knowledge, backlog, code-research, skills), не спрашивать пользователя то, что можно найти самостоятельно.

## Intermediate Summaries

After completing each phase/checkpoint WITHIN a skill (not just at the end), ALWAYS show a brief summary:
- What was done in this phase (1-2 sentences)
- Key decisions or artifacts created
- What comes next within the current skill

This applies to every checkpoint marked in skill instructions. Do NOT silently proceed to the next phase — the user must see progress between steps.

## Documentation Discipline

- **decisions.md**: update after every significant decision during any pipeline step — tech choice, scope change, tradeoff, rejected alternative. If a decision was made — it goes into decisions.md immediately, not at the end.
- **`/done` is mandatory**: after `/retrospective`, ALWAYS remind the user to run `/done`. This is the step that updates Project Knowledge. Skipping it = documentation debt.
- **Mid-feature PK updates**: if a pipeline step reveals that project-knowledge is outdated or wrong (e.g., architecture changed, new pattern established), fix it immediately — don't wait for `/done`.

## Pipeline Navigation

After completing any pipeline step, ALWAYS tell the user the next step. This is the standard flow:

**New project:**
`/init-project` → `/init-project-knowledge` → start first feature

**New feature (full pipeline):**
`/new-user-spec` → `/new-tech-spec` → `/decompose-tech-spec` → `/do-feature` or `/do-task` → `/retrospective` → `/done`

**After each step, say:** "Следующий шаг: `/command-name` — краткое описание. Запустить?"

If unsure where the user is in the pipeline, check `work/` directory for existing specs/tasks and their statuses to determine the current stage.
