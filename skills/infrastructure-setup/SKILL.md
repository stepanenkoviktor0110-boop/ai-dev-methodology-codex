---
name: infrastructure-setup
description: |
  Sets up dev infrastructure for new projects: framework init, folder structure,
  Docker, pre-commit hooks (gitleaks), testing infrastructure, .gitignore.

  Use when: "настрой инфраструктуру", "подготовь проект", "настрой тесты",
  "настрой проверки при коммите", "настрой проверки при пуше", "setup infrastructure"
---

# Infrastructure Setup

## Gathering Project Context

Read project-knowledge references:
- `.agents/skills/project-knowledge/references/architecture.md` — tech stack, framework
- `.agents/skills/project-knowledge/references/patterns.md` — code conventions, branching strategy, testing
- `.agents/skills/project-knowledge/references/deployment.md` — deployment strategy

If files lack needed info, search other project-knowledge references — info may exist under different names. If missing entirely, propose a default and confirm with user.

## Phase 0: Confirm Setup Plan with User — BLOCKING GATE

Before installing or creating anything, present a summary of what you will set up. Use **plain language** — describe what the user will get, not technical internals.

Example format:
```
Вот что я настрою для проекта:

1. **Основа приложения:** React + TypeScript — интерфейс будет работать в браузере, без серверной части.
   Стили: Tailwind CSS — готовые классы вместо написания CSS вручную.
2. **Структура папок:** отдельные папки для компонентов, сервисов, тестов.
3. **Тесты:** Vitest — можно будет запускать проверки одной командой `npm test`.
4. **Защита от утечек:** pre-commit хук gitleaks — не даст случайно закоммитить пароли/ключи.
5. **Docker:** не нужен (нет серверной части).

Подтверждаешь? Или хочешь что-то изменить?
```

**Rules for this summary:**
- Describe what each tool DOES for the user, not what it IS. Wrong: "Vitest — fast unit testing framework with ESM support". Right: "Vitest — можно будет запускать проверки одной командой `npm test`."
- If project-knowledge specifies the stack — still confirm it. The user may have changed their mind since planning.
- If project-knowledge is missing or vague about a choice — propose a default with one-sentence reasoning and ask to confirm.
- Wait for explicit approval before proceeding to Phase 1.

## Phase 1: Framework Initialization

**Non-empty directory handling:** Many scaffolding tools (Vite, Next.js, CRA) refuse to init in a non-empty directory. If the project root already has files (e.g., `work/`, `.agents/`, `.gitignore`):
1. Create the scaffold in a temporary subdirectory (e.g., `_scaffold_tmp/`).
2. Move runtime files (src/, package.json, tsconfig.json, vite.config.ts, etc.) from `_scaffold_tmp/` to the project root.
3. Delete `_scaffold_tmp/`.
4. Merge any generated `.gitignore` entries into the existing `.gitignore` instead of overwriting it.

Init framework from confirmed stack. Use Context7 for up-to-date init commands and flags.

**Dev server verification:** Run `npm run dev` (or equivalent) and confirm the process starts. If the environment does not support long-running process stdout (timeouts, restricted terminals), verify the toolchain via `npm run build` instead — a successful build confirms the framework is functional.

**Checkpoint:** dev server starts or build succeeds.

## Phase 2: Folder Structure

Convention — separate concerns by purpose:

- **Web Apps:** `src/{components, services, lib, config}` + `tests/{unit, integration, e2e}`
- **APIs:** `src/{routes, services, models, middleware, config}` + `tests/{unit, integration}`
- **CLI tools:** `src/{commands, services, config}` + `tests/{unit, integration}`

Add `src/prompts/` if project uses LLM prompts. Add `src/messages/` if project uses i18n.

**Checkpoint:** structure created, matches project type.

## Phase 3: Docker (conditional)

Set up only if specified in project-knowledge or user confirms.

**Checkpoint:** `docker build` succeeds, container starts.

## Phase 4: .gitignore

Security patterns (always add):
```
.env
.env.*
!.env.example
*.key
*.pem
credentials.json
secrets/
```

Add framework-specific patterns from `architecture.md`.
Create `.env.example` with required variable names (no values).

**Checkpoint:** `git check-ignore .env` returns `.env`.

## Phase 5: Pre-commit Hooks

Convention: gitleaks for secret scanning. Target: total pre-commit time under 10 seconds.

**Gitleaks installation:** Ensure gitleaks is available before configuring the hook:
- macOS: `brew install gitleaks`
- Windows: `winget install gitleaks` (binary may land outside PATH — check `$LOCALAPPDATA\Microsoft\WinGet\Links\` or `$HOME\AppData\Local\Microsoft\WinGet\Packages\*gitleaks*`)
- Linux / CI: download from GitHub releases or use `go install github.com/gitleaks/gitleaks/v8@latest`
- Verify: `gitleaks version`

The pre-commit hook template includes a fallback path search — see `husky-pre-commit-gitleaks.sh`.

**Custom rules (.gitleaks.toml):** The default gitleaks ruleset may not catch all test patterns. Place a `.gitleaks.toml` in the project root with explicit rules for the project's secret formats. Use the template from `$AGENTS_HOME/shared/templates/infrastructure/static/.gitleaks.toml`.

Pre-commit scope (fast, staged files only):
- gitleaks (~2-5 seconds)
- Lint staged files
- Format check

Full test suites, integration tests, builds belong in CI.

**Checkpoint — clean testing approach:** Verify gitleaks blocks secrets WITHOUT polluting git history:
1. Create a temp file with a test secret: `echo "AKIA1234567890EXAMPLE" > _gitleaks_test.txt`
2. Stage it: `git add _gitleaks_test.txt`
3. Attempt commit: `git commit -m "test: gitleaks check"` — must be BLOCKED.
4. **Clean up immediately:** `git reset HEAD _gitleaks_test.txt && rm _gitleaks_test.txt`
5. Do NOT leave test secrets in committed history.

If gitleaks does not block the test secret, check `.gitleaks.toml` rules — the default ruleset may require `AKIA` patterns to have 20+ alphanumeric characters.

## Phase 6: Testing Infrastructure

Set up test framework, create smoke test: 1-2 tests verifying setup works (import main module, check environment).

**Checkpoint:** test command passes.

## Cross-platform Notes

**Windows / PowerShell gotchas:**
- Some compound or destructive shell commands may be blocked by execution policies. Break complex operations into smaller steps.
- Prefer `npm pkg set` / `npm pkg get` for editing `package.json` programmatically instead of parsing JSON via PowerShell objects (PSCustomObject has limitations with dynamic properties).
- Use forward slashes in paths for cross-platform compatibility in scripts and configs.
- If `husky` hooks don't fire on Windows, ensure git's `core.hooksPath` is set correctly: `git config core.hooksPath .husky`.

## Phase 7: Documentation & Commit

Update project-knowledge references (append, don't overwrite):
- `deployment.md` — required environment variables
- `patterns.md` (Git Workflow section) — pre-commit hooks and what they check

Commit:
```
chore: setup project infrastructure

- Initialize [framework] project
- Setup pre-commit hooks (gitleaks)
- Create folder structure
- Add testing infrastructure
- Configure .gitignore and .env.example
[- Setup Docker (if applicable)]
```

Verify before commit: `git status` shows no `.env` files (only `.env.example`).

## Final Validation

- [ ] Framework runs locally
- [ ] Folder structure matches convention
- [ ] gitleaks blocks test secret
- [ ] `.gitignore` covers `.env`, `*.key`, secrets
- [ ] `.env.example` exists (if project uses env vars)
- [ ] Smoke test passes
- [ ] Documentation updated
- [ ] All infrastructure committed
- [ ] Docker works (if applicable)
