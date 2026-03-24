# Global Preferences

## Instruction Tone: Strict by Default

These instructions are written for Codex agents that default to the laziest interpretation of any rule. Therefore:
- Every SHOULD is a MUST. Every "consider" is "do it". Every "if needed" is "always".
- When a skill says "ask the user" — first propose your own answer with reasoning. Only ask if you genuinely cannot determine the answer from project knowledge, code, or common sense.
- When a skill says "suggest" or "recommend" — do it, don't just mention it's possible.
- Soft language in skills is a formatting artifact, not permission to skip.

## Skill Execution: No Scripts

- **NEVER search for or run helper scripts** (dispatch-skill.ps1, init-feature-folder.sh, etc.). They do not exist. All skills are SKILL.md files — read and follow them directly.
- When a shim skill (e.g., `decompose-tech-spec`) says "Read and follow {target SKILL.md}" — load that file and execute its instructions step by step. Do NOT improvise a replacement procedure.
- If a SKILL.md references `$AGENTS_HOME` — resolve it to the actual agents home path and read the file. If a referenced file doesn't exist, tell the user — do NOT invent a workaround.
- **Integrity check:** before executing any SKILL.md, verify it has no git conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`). If found — do NOT execute, tell user to resolve the conflict first.

## Git: Known Pitfalls

- **`work/{feature}/logs/` is gitignored** by most `.gitignore` configs (global `logs/` rule). For methodology artifacts in this directory (session-plan.md, review reports, etc.) use `git add -f` when committing. Otherwise phase-gate commits will silently skip these files.

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

## Framework Update Protocol

When user says "обнови фреймворк", "обновили скиллы", "pull agents", or similar:

**Step 1: Save context anchor BEFORE touching anything.**
Write down (in your response, not in a file) a context snapshot:
- Current project path
- Current feature name and pipeline stage (e.g., "health-dashboard-mvp, session 2 of /do-feature")
- Current/next task number
- Any in-progress work or pending decisions

**Step 2: Update framework — minimal scope.**
```bash
cd ~/.agents && git pull origin master
```
Then re-read ONLY `~/.agents/AGENTS.md`. Do NOT re-read all skills — they are loaded on demand when needed.

**Step 3: Report changes briefly.**
Show `git log --oneline @{1}..HEAD` output — just commit titles. Do NOT read each changed file.

**Step 4: Return to project using the context anchor from Step 1.**
`cd` back to the project directory. Re-read the project's `AGENTS.md` (not the global one — the project-level one). State explicitly: "Возвращаюсь к {feature}, {stage}. Продолжаем."

**Critical rules:**
- The project's `work/` directory, specs, tasks, decisions.md are UNTOUCHED by framework updates. Do not re-read them unless you need to — you already have them in context.
- Do NOT start exploring updated skill files out of curiosity. Updated skills take effect next time they are loaded by a pipeline step.
- Total time on framework update: under 30 seconds. If it takes longer — you are doing too much.
