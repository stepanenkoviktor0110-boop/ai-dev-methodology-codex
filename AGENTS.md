# Global Preferences

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

## Pipeline Navigation

After completing any pipeline step, ALWAYS tell the user the next step. This is the standard flow:

**New project:**
`/init-project` → `/init-project-knowledge` → start first feature

**New feature (full pipeline):**
`/new-user-spec` → `/new-tech-spec` → `/decompose-tech-spec` → `/do-feature` or `/do-task` → `/retrospective` → `/done`

**After each step, say:** "Следующий шаг: `/command-name` — краткое описание. Запустить?"

If unsure where the user is in the pipeline, check `work/` directory for existing specs/tasks and their statuses to determine the current stage.
