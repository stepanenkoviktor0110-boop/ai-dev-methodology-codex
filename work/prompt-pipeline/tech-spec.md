---
created: 2026-03-25
status: draft
branch: feature/prompt-pipeline
size: M
---

# Tech Spec: Prompt Pipeline

## Solution

n8n workflow на VPS, реализующий пошаговый пайплайн разработки проектов через copy-paste в бесплатные LLM. Workflow состоит из цепочки Form Trigger нод — каждый шаг показывает сгенерированный промт, принимает ответ LLM, парсит результат. Состояние сессии хранится в JSON-файле на диске. Промты хранятся в отдельных .md файлах. На выходе — набор файлов проекта на VPS + скачиваемый .zip архив.

Пайплайн: выбор шаблона → генерация спеки → валидация спеки → декомпозиция на файлы (5-8 MVP + бэклог) → генерация каждого файла с контекстом предыдущих → опциональный code review → создание файлов + архив.

## Architecture

### What we're building/modifying

- **n8n Workflow (JSON)** — основной workflow с цепочкой форм, логикой шагов, парсингом ответов, файловыми операциями
- **Prompt Templates (.md)** — набор .md файлов с промтами для каждого шага (спека, валидация, декомпозиция, генерация файла, code review) + per-template контекст для каждого из 12 шаблонов
- **Template Configs (JSON)** — конфигурации шаблонов проектов: структура файлов, стек, типичные заглушки, ограничения
- **Parser (JS in Code node)** — парсер ответов LLM по маркерам `=== FILE ===` / `=== END FILE ===`
- **State Manager (JS in Code node)** — чтение/запись JSON-файла состояния сессии
- **Archive Generator (JS in Code node)** — создание .zip из готовых файлов проекта

### How it works

```
[Form: New/Resume] → [Load template config] → [Generate spec prompt]
  → [Form: paste LLM response] → [Parse spec] → [Generate validation prompt]
  → [Form: paste validation] → [Update spec] → [Generate decomposition prompt]
  → [Form: paste decomposition] → [Parse file list]
  → [Loop for each file]:
      [Generate file prompt with context] → [Form: paste code]
      → [Parse file] → [Optional: review prompt → Form: paste review → fix prompt → Form: paste fix]
      → [Save state]
  → [Write all files to disk] → [Generate .zip] → [Form: download link + backlog + session log]
```

Каждый Form Trigger — отдельный webhook URL. n8n Code nodes содержат JS-логику (парсер, state manager). Промты собираются из .md шаблонов с подстановкой переменных (описание проекта, спека, готовые файлы).

### Shared resources

None.

## Decisions

### Decision 1: Цепочка Form Triggers вместо single-page UI
**Decision:** Каждый шаг = отдельный Form Trigger с уникальным webhook URL
**Rationale:** n8n не поддерживает динамические многошаговые формы в одном trigger. Цепочка форм — единственный нативный способ реализовать пошаговый flow.
**Alternatives considered:** Custom UI (отдельное веб-приложение) — слишком сложно, противоречит идее "всё на n8n". Sub-workflows — добавляют сложность без выгоды.

### Decision 2: JSON-файл на диске для состояния
**Decision:** Состояние сессии (текущий шаг, спека, готовые файлы) хранится в JSON-файле на диске VPS
**Rationale:** Простой, надёжный, дебажится руками. n8n Static Data ограничен по объёму и привязан к execution.
**Alternatives considered:** n8n Static Data — ограничен, неудобен для больших объёмов. SQLite — оверкилл для одного пользователя.

### Decision 3: Промты в .md файлах, не в n8n нодах
**Decision:** Все промты хранятся в отдельных .md файлах, n8n читает их через fs.readFileSync
**Rationale:** Проще редактировать, версионировать, итерировать. Не нужно открывать n8n editor для правки промта.
**Alternatives considered:** Промты внутри Code node — сложно редактировать, теряется форматирование.

### Decision 4: ZIP-архив вместо прямой записи на локальную машину
**Decision:** MVP: файлы создаются на VPS, пользователь скачивает .zip. SSHFS — в бэклоге.
**Rationale:** Простой, не требует настройки сетевых дисков. Работает из любого браузера.
**Alternatives considered:** SSHFS монтирование — требует настройки, не всегда удобно. Запланировано как улучшение.

### Decision 5: 12 шаблонов с предопределённой структурой
**Decision:** Фиксированный набор из 12 шаблонов, каждый определяет стек, структуру файлов (5-8), типичные заглушки
**Rationale:** Ограничение размера проекта гарантирует что контекст влезает в LLM. Шаблоны покрывают основные use-cases автора.
**Alternatives considered:** Произвольный проект — невозможно гарантировать размер и качество промтов.

## Data Models

### Session State (JSON file)

```json
{
  "id": "uuid",
  "status": "in_progress | completed | abandoned",
  "template": "landing-page",
  "description": "Сайт моей кофейни",
  "outputPath": "/home/user/projects/coffee-site",
  "currentStep": "file_generation",
  "currentFileIndex": 2,
  "spec": "... полный текст спеки ...",
  "fileList": [
    { "path": "index.html", "description": "Main page", "status": "done" },
    { "path": "styles.css", "description": "Styles", "status": "pending" }
  ],
  "generatedFiles": {
    "index.html": "<!DOCTYPE html>..."
  },
  "backlog": ["Подключить БД", "Добавить авторизацию"],
  "stubs": ["// TODO: форма обратной связи — пока заглушка"],
  "reviewEnabled": false,
  "log": {
    "startedAt": "2026-03-25T10:00:00Z",
    "steps": [
      {
        "step": "spec_generation",
        "prompt": "...",
        "response": "...",
        "attempts": 1,
        "timestamp": "2026-03-25T10:05:00Z"
      }
    ]
  }
}
```

### Template Config (JSON file per template)

```json
{
  "id": "landing-page",
  "name": "Визитка / лендинг",
  "category": "sites",
  "description": "Одностраничный сайт с информацией о чём угодно",
  "example": "Сайт моей кофейни с меню и контактами",
  "stack": ["HTML", "CSS", "JavaScript"],
  "maxFiles": 5,
  "typicalFiles": [
    "index.html",
    "styles.css",
    "script.js",
    "assets/README.md"
  ],
  "typicalStubs": [
    "// TODO: форма обратной связи — пока заглушка",
    "// TODO: подключение аналитики — пока без трекинга"
  ],
  "promptContext": "Это одностраничный лендинг. Используй семантический HTML5, CSS custom properties, vanilla JS. Без фреймворков, без сборщиков."
}
```

## Dependencies

### New packages
- `archiver` (npm) — создание .zip архивов в n8n Code node (или встроенный `zlib` модуля Node.js)

### Using existing (from project)
- `n8n` — платформа workflow, Form Trigger, Code nodes, Webhook
- `fs` (Node.js built-in) — чтение/запись файлов, создание директорий
- `path` (Node.js built-in) — работа с путями, защита от path traversal

## Testing Strategy

**Feature size:** M

### Unit tests
- Parser: корректные маркеры `=== FILE ===` / `=== END FILE ===` → массив файлов
- Parser: ответ без маркеров → один файл
- Parser: вложенные блоки кода (``` внутри файла) → корректный парсинг
- Parser: обрезанный ответ (незакрытый маркер) → сохранение до конца + предупреждение
- Parser: path traversal (`../../../etc/passwd`) → отклонение
- Parser: пустой ответ → ошибка
- State Manager: сохранение/загрузка состояния → JSON корректен
- State Manager: отсутствие файла состояния → новая сессия

### Integration tests
None — n8n workflow тестируется вручную.

### E2E tests
None — интерфейс это формы n8n.

## Agent Verification Plan

### Verification approach
1. Импорт workflow в n8n через CLI/API, проверить что workflow появился
2. Запуск парсера на тестовых данных (корректные маркеры, без маркеров, path traversal)
3. Проверка наличия всех .md файлов промтов и 12 JSON-конфигов шаблонов
4. Проверка что архиватор создаёт валидный .zip

### Tools required
- bash — запуск тестов, проверка файлов
- curl — импорт workflow в n8n через API

## Risks

| Risk | Mitigation |
|------|-----------|
| LLM игнорирует формат маркеров | Чёткая инструкция с примером в каждом промте + кнопка "Повторить" |
| Контекст переполняется при подстановке предыдущих файлов | Шаблоны ограничивают размер: 5-8 маленьких файлов |
| n8n Code node не поддерживает require('archiver') | Fallback: использовать встроенный zlib + tar, или генерировать архив через bash команду zip |
| Path traversal в путях файлов из ответа LLM | Валидация в парсере: путь нормализуется, проверяется что не выходит за outputPath |
| Потеря состояния при крэше n8n | JSON-файл сохраняется после каждого шага, восстановление при следующем запуске |

## Acceptance Criteria

- [ ] n8n workflow импортируется и запускается без ошибок
- [ ] Парсер корректно разбирает ответы LLM по маркерам (все unit-test сценарии проходят)
- [ ] Path traversal в путях файлов отклоняется
- [ ] Все 12 шаблонов имеют конфигурации и промт-контекст
- [ ] Промты подставляют контекст проекта (описание, спека, готовые файлы)
- [ ] .zip архив создаётся и скачивается
- [ ] JSON-лог сессии записывается с промтами, ответами, таймстемпами
- [ ] Состояние сохраняется после каждого шага, восстанавливается при перезапуске
- [ ] Все unit-тесты парсера проходят

## Implementation Tasks

### Wave 1 (независимые)

#### Task 1: Project Structure & Parser
- **Description:** Создать структуру проекта (папки для промтов, конфигов, тестов) и реализовать парсер ответов LLM. Парсер — критический компонент: разбирает текст по маркерам `=== FILE ===` / `=== END FILE ===`, валидирует пути (path traversal protection). Покрыть unit-тестами.
- **Skill:** code-writing
- **Reviewers:** code-reviewer, security-auditor, test-reviewer
- **Verify-smoke:** `node -e "const p = require('./src/parser'); console.log(JSON.stringify(p.parse('=== FILE: test.js ===\nconsole.log(1)\n=== END FILE ===')))"` → `[{path: "test.js", content: "console.log(1)"}]`
- **Files to modify:** `src/parser.js`, `src/parser.test.js`
- **Files to read:** —

#### Task 2: Template Configs
- **Description:** Создать JSON-конфигурации для всех 12 шаблонов проектов. Каждый конфиг определяет: name, category, stack, maxFiles, typicalFiles, typicalStubs, promptContext. Шаблоны сгруппированы по категориям (sites, bots, webapps, automations, spreadsheets, utilities).
- **Skill:** code-writing
- **Reviewers:** code-reviewer
- **Files to modify:** `templates/*.json` (12 files)
- **Files to read:** —

#### Task 3: Prompt Templates
- **Description:** Написать .md файлы промтов для каждого шага пайплайна: spec generation, spec validation, decomposition, file generation, code review. Каждый промт содержит инструкцию по формату ответа (маркеры), placeholders для контекста ({description}, {spec}, {previousFiles}, {templateContext}).
- **Skill:** prompt-master
- **Reviewers:** prompt-reviewer
- **Files to modify:** `prompts/spec.md`, `prompts/validate-spec.md`, `prompts/decompose.md`, `prompts/generate-file.md`, `prompts/review-file.md`
- **Files to read:** `templates/*.json` (для promptContext)

#### Task 4: State Manager
- **Description:** Реализовать модуль управления состоянием сессии: создание, загрузка, обновление, проверка наличия незавершённой сессии. Хранение в JSON-файле на диске. Используется n8n Code nodes для вызова.
- **Skill:** code-writing
- **Reviewers:** code-reviewer, test-reviewer
- **Files to modify:** `src/state.js`, `src/state.test.js`
- **Files to read:** —

### Wave 2 (зависит от Wave 1)

#### Task 5: n8n Workflow — Core Flow
- **Description:** Собрать основной n8n workflow: Form Trigger для выбора шаблона → цепочка шагов (спека, валидация, декомпозиция, генерация файлов). Code nodes вызывают парсер, state manager, читают промты. Реализовать логику подстановки контекста в промты и цикл по файлам.
- **Skill:** code-writing
- **Reviewers:** code-reviewer
- **Verify-user:** Открыть workflow в n8n → запустить → пройти первые 3 шага (выбор шаблона, спека, валидация)
- **Files to modify:** `workflow/prompt-pipeline.json`
- **Files to read:** `src/parser.js`, `src/state.js`, `prompts/*.md`, `templates/*.json`

#### Task 6: Archive & File Writer
- **Description:** Реализовать запись файлов проекта на диск по указанному пути (с созданием поддиректорий) и генерацию .zip архива. Финальный шаг workflow показывает ссылку на скачивание + бэклог + список заглушек.
- **Skill:** code-writing
- **Reviewers:** code-reviewer, security-auditor
- **Verify-smoke:** `node -e "const w = require('./src/writer'); w.writeFiles('/tmp/test-project', [{path:'index.html',content:'<h1>test</h1>'}])"` → файл создан
- **Files to modify:** `src/writer.js`, `src/writer.test.js`
- **Files to read:** `src/state.js`

### Wave 3 (зависит от Wave 2)

#### Task 7: Session Recovery & Logging
- **Description:** Добавить в workflow: при запуске проверка незавершённой сессии (предложить продолжить/начать новую), кнопку "Повторить шаг", опциональный code review (галочка). Добавить запись JSON-лога сессии (промты, ответы, таймстемпы, количество попыток).
- **Skill:** code-writing
- **Reviewers:** code-reviewer
- **Verify-user:** Запустить workflow → пройти 2 шага → закрыть → запустить снова → выбрать "Продолжить" → убедиться что продолжает с правильного шага
- **Files to modify:** `workflow/prompt-pipeline.json`, `src/state.js`, `src/logger.js`
- **Files to read:** `src/parser.js`

### Audit Wave

#### Task 8: Code Audit
- **Description:** Full-feature code quality audit. Read all source files created/modified in this feature. Review holistically for cross-component issues: duplicate resource initialization, shared resources compliance with Architecture decisions, architectural consistency. Write audit report.
- **Skill:** code-reviewing
- **Reviewers:** none

#### Task 9: Security Audit
- **Description:** Full-feature security audit. Read all source files created/modified in this feature. Analyze for OWASP Top 10 across all components, cross-component auth/data flow (path traversal protection in parser and writer). Write audit report.
- **Skill:** security-auditor
- **Reviewers:** none

#### Task 10: Test Audit
- **Description:** Full-feature test quality audit. Read all test files created in this feature. Verify coverage of parser edge cases, state manager scenarios, meaningful assertions. Write audit report.
- **Skill:** test-master
- **Reviewers:** none

### Final Wave

#### Task 11: Pre-deploy QA
- **Description:** Acceptance testing: run all unit tests, verify acceptance criteria from user-spec and tech-spec. Manual walkthrough of full pipeline with "Визитка / лендинг" template.
- **Skill:** pre-deploy-qa
- **Reviewers:** none

#### Task 12: Deploy
- **Description:** Deploy workflow and supporting files to VPS: import workflow into n8n, copy prompt templates and template configs to configured directory, verify workflow starts.
- **Skill:** deploy-pipeline
- **Reviewers:** none
- **Verify-smoke:** `curl -s http://VPS_URL:5678/api/v1/workflows | grep prompt-pipeline` → workflow found
