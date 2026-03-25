---
created: 2026-03-25
status: approved
branch: feature/prompt-pipeline
size: M
---

# Tech Spec: Prompt Pipeline

## Solution

n8n workflow на VPS, реализующий пошаговый пайплайн разработки проектов через copy-paste в бесплатные LLM. Workflow использует 1 Form Trigger + цепочку Form nodes для многошагового flow. Каждый шаг показывает сгенерированный промт, принимает ответ LLM, парсит результат. Состояние сессии хранится в JSON-файле на диске. Промты хранятся в отдельных .md файлах. На выходе — набор файлов проекта на VPS + скачиваемый .zip архив.

Пайплайн: выбор шаблона → генерация спеки → валидация спеки → декомпозиция на файлы (5-8 MVP + бэклог) → генерация каждого файла с контекстом предыдущих → опциональный code review → создание файлов + архив.

## Architecture

### What we're building/modifying

- **n8n Workflow (JSON)** — основной workflow: 1 Form Trigger (точка входа) + цепочка Form nodes (шаги), Code nodes (логика), Execute Command node (архивация)
- **Prompt Templates (.md)** — набор .md файлов с промтами для каждого шага (спека, валидация, декомпозиция, генерация файла, code review) + per-template контекст для каждого из 12 шаблонов
- **Template Configs (JSON)** — конфигурации шаблонов проектов: структура файлов, стек, типичные заглушки, ограничения
- **Parser (JS in Code node)** — парсер ответов LLM по маркерам `=== FILE ===` / `=== END FILE ===` с path traversal protection
- **State Manager (JS in Code node)** — чтение/запись JSON-файла состояния сессии
- **Logger (JS in Code node)** — запись JSON-лога сессии (промты, ответы, таймстемпы) в отдельный файл для ретроспективы
- **File Writer (JS in Code node)** — запись файлов на диск с созданием поддиректорий, валидацией путей (defense in depth)

### How it works

```
[Form Trigger: New/Resume] → [Code: check pending session]
  → [Form: choose template + describe project + set output path]
  → [Code: load template config, generate spec prompt]
  → [Form: show prompt, accept LLM response]
  → [Code: parse spec, save state]
  → [Code: generate validation prompt]
  → [Form: show prompt, accept validation response]
  → [Code: update spec, save state]
  → [Code: generate decomposition prompt]
  → [Form: show prompt, accept decomposition response]
  → [Code: parse file list, save state]
  → [Loop for each file]:
      → [Code: generate file prompt with context (spec + files 1..N-1)]
      → [Form: show prompt, accept code response]
      → [Code: validate non-empty, parse file, save state]
      → [Optional if review enabled]:
          → [Code: generate review prompt]
          → [Form: show review prompt, accept review response]
          → [Code: generate fix prompt]
          → [Form: show fix prompt, accept fixed code]
          → [Code: parse fixed file, save state]
  → [Code: write all files to disk (with path validation)]
  → [Execute Command: zip -r archive.zip project-dir]
  → [Form: download link + backlog + stubs list + session summary]
  → [Code: finalize log, mark session completed]
```

**Input validation:** каждый Form node с полем для ответа LLM проверяет что ответ непустой (n8n Form node required field). Если пустой — форма не отправляется.

**Path traversal protection (defense in depth):**
1. Parser: нормализует путь через `path.normalize()`, отклоняет пути начинающиеся с `..`, содержащие `..` после нормализации, абсолютные пути, null-bytes (`\0`), URL-encoded sequences (`%2e%2e`)
2. Writer: перед записью проверяет что `path.resolve(outputPath, filePath)` начинается с `path.resolve(outputPath)` — финальная проверка после нормализации

**outputPath validation:** путь ввода ограничен настраиваемой базовой директорией (allowedBasePath в конфиге). Writer проверяет что outputPath начинается с allowedBasePath. По умолчанию: `/home/user/projects/`.

**Error handling при записи файлов:** при ошибке записи (нет прав, диск полон) — показывается ошибка с конкретным путём, уже записанные файлы не удаляются. Пользователь видит какие файлы записаны, какие нет.

### Shared resources

None.

## Decisions

### Decision 1: Многошаговые формы через Form Trigger + Form nodes
**Decision:** 1 Form Trigger (точка входа) + цепочка Form nodes внутри workflow для каждого шага
**Rationale:** n8n нативно поддерживает многошаговые формы: Form Trigger начинает flow, Form nodes продолжают его. Это даёт единый workflow с пошаговым UI без отдельных webhook URL.
**Alternatives considered:** Отдельный Form Trigger на каждый шаг — неверная архитектура, n8n не так работает. Custom UI — слишком сложно.

### Decision 2: JSON-файл на диске для состояния
**Decision:** Состояние сессии (текущий шаг, спека, готовые файлы) хранится в JSON-файле на диске VPS
**Rationale:** Простой, надёжный, дебажится руками. n8n Static Data персистентен между запусками, но неудобен для больших объёмов данных (полный текст спеки + всех файлов).
**Alternatives considered:** n8n Static Data — персистентен, но ограничен по объёму для нашего use-case. SQLite — оверкилл для одного пользователя.

### Decision 3: Промты в .md файлах, не в n8n нодах
**Decision:** Все промты хранятся в отдельных .md файлах, n8n читает их через fs.readFileSync
**Rationale:** Проще редактировать, версионировать, итерировать. Не нужно открывать n8n editor для правки промта.
**Alternatives considered:** Промты внутри Code node — сложно редактировать, теряется форматирование.

### Decision 4: ZIP через Execute Command node вместо archiver
**Decision:** Архив создаётся через Execute Command node: `zip -r archive.zip project-dir`
**Rationale:** Встроенный `zlib` Node.js создаёт только gzip, не ZIP. npm-пакет `archiver` требует настройки `NODE_FUNCTION_ALLOW_EXTERNAL`. Команда `zip` доступна на VPS из коробки, проще всего.
**Alternatives considered:** `archiver` npm — требует конфигурации n8n env vars. `zlib` — не создаёт .zip. `tar.gz` — менее удобен для пользователя (Windows).

### Decision 5: 12 шаблонов с предопределённой структурой
**Decision:** Фиксированный набор из 12 шаблонов, каждый определяет стек, структуру файлов (5-8), типичные заглушки
**Rationale:** Ограничение размера проекта гарантирует что контекст влезает в LLM. Шаблоны покрывают основные use-cases автора.
**Alternatives considered:** Произвольный проект — невозможно гарантировать размер и качество промтов.

### Decision 6: Webhook security — Basic Auth
**Decision:** Form Trigger защищён Basic Auth (login/password в n8n credentials). Личный инструмент — сложная аутентификация не нужна.
**Rationale:** Без защиты webhook публично доступен — любой может инициировать запись файлов на VPS. Basic Auth — минимально достаточная защита для single-user tool.
**Alternatives considered:** IP whitelist — неудобно при смене IP. OAuth — оверкилл. Без защиты — неприемлемо.

### Decision 7: При "Начать новый" — файлы на диске остаются
**Decision:** При выборе "Начать новый проект" удаляется только JSON-файл состояния сессии. Файлы проекта, созданные на предыдущей сессии, остаются на диске.
**Rationale:** Предотвращает случайную потерю работы. Пользователь сам решает что удалять.

## Data Models

### Session State (JSON file: `~/.prompt-pipeline/state.json`)

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
  "stubs": [
    { "file": "script.js", "stub": "// TODO: форма обратной связи", "resolved": false }
  ],
  "reviewEnabled": false
}
```

### Session Log (JSON file: `~/.prompt-pipeline/logs/{id}.json`)

```json
{
  "sessionId": "uuid",
  "template": "landing-page",
  "description": "Сайт моей кофейни",
  "startedAt": "2026-03-25T10:00:00Z",
  "completedAt": "2026-03-25T10:45:00Z",
  "steps": [
    {
      "step": "spec_generation",
      "prompt": "...",
      "response": "...",
      "attempts": 1,
      "timestamp": "2026-03-25T10:05:00Z"
    }
  ],
  "totalAttempts": 12,
  "filesGenerated": ["index.html", "styles.css", "script.js"],
  "stubs": [
    { "file": "script.js", "stub": "// TODO: форма обратной связи", "resolved": false }
  ],
  "backlog": ["Подключить БД"]
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

### Config (JSON: `~/.prompt-pipeline/config.json`)

```json
{
  "allowedBasePath": "/home/user/projects",
  "stateDir": "/home/user/.prompt-pipeline",
  "promptsDir": "/path/to/prompts",
  "templatesDir": "/path/to/templates"
}
```

## Dependencies

### New packages
None — всё реализуется встроенными средствами n8n и Node.js.

### Using existing (from project)
- `n8n` — платформа: Form Trigger, Form nodes, Code nodes, Execute Command node
- `fs` (Node.js built-in) — чтение/запись файлов, создание директорий. Требует `NODE_FUNCTION_ALLOW_BUILTIN=fs,path` в env n8n.
- `path` (Node.js built-in) — работа с путями, path traversal protection
- `zip` (system command) — создание .zip архивов через Execute Command node

### n8n configuration required
- Env variable: `NODE_FUNCTION_ALLOW_BUILTIN=fs,path` — разрешает использование fs и path в Code nodes

## Testing Strategy

**Feature size:** M

### Unit tests

**Parser:**
- Корректные маркеры `=== FILE: path ===` / `=== END FILE ===` → массив файлов с path и content
- Несколько файлов в одном ответе → все распарсены
- Ответ без маркеров → один файл (весь ответ как content)
- Вложенные блоки кода (``` внутри файла) → корректный парсинг, маркеры не путаются с markdown
- Обрезанный ответ (незакрытый маркер) → сохранение до конца + warning flag
- Path traversal: `../etc/passwd`, `....//etc/passwd`, `/etc/passwd`, `%2e%2e/etc/passwd`, `path\0.js` → все отклоняются
- Пустой ответ → ошибка
- Пробелы/переносы строк вокруг маркеров → корректный парсинг (trim)

**State Manager:**
- Создание новой сессии → JSON-файл создан с корректной структурой
- Загрузка существующей сессии → данные корректны
- Обновление шага → currentStep/currentFileIndex обновлены
- Добавление сгенерированного файла → generatedFiles обновлен, fileList status = done
- Отсутствие файла состояния → возвращает null (новая сессия)
- Повреждённый JSON → ошибка с понятным сообщением
- Завершение сессии → status = completed

**Writer:**
- Запись файла → файл создан с правильным содержимым
- Запись в несуществующую поддиректорию → директория создана автоматически
- Path traversal в filePath → отклоняется (defense in depth)
- outputPath за пределами allowedBasePath → отклоняется
- Ошибка записи (readonly директория) → ошибка с путём, остальные файлы не затронуты

**Logger:**
- Запись шага → JSON-лог содержит prompt, response, timestamp, attempts
- Финализация → completedAt заполнен, итоги подсчитаны
- Чтение лога → корректный JSON

### Integration tests
None — n8n workflow тестируется вручную.

### E2E tests
None — интерфейс это формы n8n.

## Agent Verification Plan

### Verification approach
1. Импорт workflow в n8n через CLI/API, проверить что workflow появился
2. Запуск unit-тестов парсера, state manager, writer, logger
3. Проверка наличия всех .md файлов промтов и 12 JSON-конфигов шаблонов
4. Проверка что zip создаётся через Execute Command

### Tools required
- bash — запуск тестов, проверка файлов
- curl — импорт workflow в n8n через API

## Risks

| Risk | Mitigation |
|------|-----------|
| LLM игнорирует формат маркеров | Чёткая инструкция с примером в каждом промте + кнопка "Повторить" |
| Контекст переполняется при подстановке предыдущих файлов | Шаблоны ограничивают размер: 5-8 маленьких файлов |
| Команда zip не установлена на VPS | Проверка при деплое, fallback: `apt install zip` |
| Path traversal в путях файлов из ответа LLM | Defense in depth: валидация в парсере + валидация в writer |
| Потеря состояния при крэше n8n | JSON-файл сохраняется после каждого шага, восстановление при следующем запуске |
| Webhook публично доступен | Basic Auth на Form Trigger |
| outputPath за пределами разрешённой директории | Валидация против allowedBasePath в config |

## Acceptance Criteria

- [ ] n8n workflow импортируется и запускается без ошибок
- [ ] Парсер корректно разбирает ответы LLM по маркерам (все unit-test сценарии проходят)
- [ ] Path traversal в путях файлов отклоняется (parser + writer, defense in depth)
- [ ] outputPath за пределами allowedBasePath отклоняется
- [ ] Пустой ответ в форме не принимается (required field)
- [ ] Все 12 шаблонов имеют конфигурации и промт-контекст
- [ ] Промты подставляют контекст проекта (описание, спека, готовые файлы)
- [ ] .zip архив создаётся и скачивается
- [ ] При ошибке записи файлов — сообщение с путём, уже записанные файлы сохраняются
- [ ] JSON-лог сессии записывается в отдельный файл с промтами, ответами, таймстемпами
- [ ] Состояние сохраняется после каждого шага, восстанавливается при перезапуске
- [ ] При "Начать новый" — файлы на диске остаются, удаляется только state
- [ ] Заглушки в stubs[] имеют поле resolved для отслеживания
- [ ] Basic Auth настроен на Form Trigger
- [ ] Все unit-тесты проходят (parser, state, writer, logger)

## Implementation Tasks

### Wave 1 (независимые)

#### Task 1: Project Structure & Parser
- **Description:** Создать структуру проекта и реализовать парсер ответов LLM. Парсер — критический компонент: разбирает текст по маркерам `=== FILE ===` / `=== END FILE ===`, валидирует пути (path traversal protection с null-byte, URL-encoding, нормализацией). Покрыть unit-тестами.
- **Skill:** code-writing
- **Reviewers:** code-reviewer, security-auditor, test-reviewer
- **Verify-smoke:** `node -e "const p = require('./src/parser'); console.log(JSON.stringify(p.parse('=== FILE: test.js ===\nconsole.log(1)\n=== END FILE ===')))"` → `[{path: "test.js", content: "console.log(1)"}]`
- **Files to modify:** `src/parser.js`, `src/parser.test.js`
- **Files to read:** —

#### Task 2: Template Configs
- **Description:** Создать JSON-конфигурации для всех 12 шаблонов проектов. Каждый конфиг определяет: name, category, stack, maxFiles, typicalFiles, typicalStubs, promptContext. Шаблоны сгруппированы по категориям.
- **Skill:** code-writing
- **Reviewers:** code-reviewer
- **Verify-smoke:** `node -e "const fs=require('fs'); const t=JSON.parse(fs.readFileSync('./templates/landing-page.json')); console.log(t.name, t.maxFiles)"` → `Визитка / лендинг 5`
- **Files to modify:** `templates/*.json` (12 files)
- **Files to read:** —

#### Task 3: Prompt Templates
- **Description:** Написать .md файлы промтов для каждого шага пайплайна: spec generation, spec validation, decomposition, file generation, code review. Каждый промт содержит инструкцию по формату ответа (маркеры), placeholders для контекста.
- **Skill:** prompt-master
- **Reviewers:** prompt-reviewer
- **Files to modify:** `prompts/spec.md`, `prompts/validate-spec.md`, `prompts/decompose.md`, `prompts/generate-file.md`, `prompts/review-file.md`
- **Files to read:** `templates/*.json`

#### Task 4: State Manager & Logger
- **Description:** Реализовать модуль состояния сессии (создание, загрузка, обновление, проверка pending) и модуль логирования (запись шагов, финализация лога в отдельный файл). Хранение в JSON-файлах. Покрыть unit-тестами.
- **Skill:** code-writing
- **Reviewers:** code-reviewer, test-reviewer
- **Verify-smoke:** `node -e "const s = require('./src/state'); const sess = s.create('landing-page','test','./tmp'); console.log(sess.status)"` → `in_progress`
- **Files to modify:** `src/state.js`, `src/state.test.js`, `src/logger.js`, `src/logger.test.js`
- **Files to read:** —

### Wave 2 (зависит от Wave 1)

#### Task 5: File Writer
- **Description:** Реализовать модуль записи файлов на диск: создание поддиректорий, валидация путей (defense in depth — проверка что resolved path внутри outputPath, outputPath внутри allowedBasePath). Обработка ошибок записи. Покрыть unit-тестами.
- **Skill:** code-writing
- **Reviewers:** code-reviewer, security-auditor, test-reviewer
- **Verify-smoke:** `node -e "const w = require('./src/writer'); w.writeFiles('/tmp/test-pp', [{path:'index.html',content:'<h1>test</h1>'}], '/tmp'); console.log('ok')"` → `ok` + файл создан
- **Files to modify:** `src/writer.js`, `src/writer.test.js`
- **Files to read:** `src/parser.js`, `src/state.js`

#### Task 6: n8n Workflow — Core Flow
- **Description:** Собрать основной n8n workflow: Form Trigger (с Basic Auth) + Form nodes для каждого шага. Code nodes вызывают parser, state manager, logger. Логика подстановки контекста в промты. Цикл по файлам. Валидация непустого ввода через required fields.
- **Skill:** code-writing
- **Reviewers:** code-reviewer
- **Verify-user:** Открыть workflow в n8n → запустить → пройти первые 3 шага (выбор шаблона, спека, валидация)
- **Files to modify:** `workflow/prompt-pipeline.json`
- **Files to read:** `src/parser.js`, `src/state.js`, `src/logger.js`, `src/writer.js`, `prompts/*.md`, `templates/*.json`

### Wave 3 (зависит от Wave 2)

#### Task 7: Session Recovery, Review & Archive
- **Description:** Добавить в workflow: при запуске проверка незавершённой сессии (продолжить/начать новую — при "новая" state удаляется, файлы на диске остаются). Кнопка "Повторить шаг". Опциональный code review (галочка). Финальный шаг: запись файлов, Execute Command `zip`, ссылка на скачивание + бэклог + stubs.
- **Skill:** code-writing
- **Reviewers:** code-reviewer
- **Verify-user:** Запустить workflow → пройти 2 шага → закрыть → запустить снова → выбрать "Продолжить" → убедиться что продолжает с правильного шага
- **Files to modify:** `workflow/prompt-pipeline.json`, `src/state.js`
- **Files to read:** `src/writer.js`, `src/logger.js`, `src/parser.js`

### Audit Wave

#### Task 8: Code Audit
- **Description:** Full-feature code quality audit. Review holistically for cross-component issues: duplicate logic, architectural consistency, error handling patterns. Write audit report.
- **Skill:** code-reviewing
- **Reviewers:** none
- **Files to read:** `src/parser.js`, `src/state.js`, `src/logger.js`, `src/writer.js`, `workflow/prompt-pipeline.json`

#### Task 9: Security Audit
- **Description:** Full-feature security audit. Focus on: path traversal defense in depth (parser + writer), outputPath validation, Basic Auth config, webhook exposure, file system access scope. Write audit report.
- **Skill:** security-auditor
- **Reviewers:** none
- **Files to read:** `src/parser.js`, `src/writer.js`, `src/state.js`, `workflow/prompt-pipeline.json`

#### Task 10: Test Audit
- **Description:** Full-feature test quality audit. Verify coverage of parser edge cases, state manager scenarios, writer security cases, logger correctness. Write audit report.
- **Skill:** test-master
- **Reviewers:** none
- **Files to read:** `src/parser.test.js`, `src/state.test.js`, `src/writer.test.js`, `src/logger.test.js`

### Final Wave

#### Task 11: Pre-deploy QA
- **Description:** Acceptance testing: run all unit tests, verify acceptance criteria from user-spec and tech-spec. Manual walkthrough of full pipeline with "Визитка / лендинг" template.
- **Skill:** pre-deploy-qa
- **Reviewers:** none
- **Files to read:** `src/*.test.js`, `workflow/prompt-pipeline.json`, `templates/*.json`, `prompts/*.md`

#### Task 12: Deploy
- **Description:** Deploy to VPS: import workflow into n8n, copy prompt templates and template configs, set NODE_FUNCTION_ALLOW_BUILTIN=fs,path in n8n env, configure Basic Auth credentials, verify workflow starts and form is accessible.
- **Skill:** deploy-pipeline
- **Reviewers:** deploy-reviewer
- **Verify-smoke:** `curl -s -u user:pass http://VPS_URL:5678/api/v1/workflows | grep prompt-pipeline` → workflow found
- **Files to read:** `workflow/prompt-pipeline.json`, `templates/*.json`, `prompts/*.md`
