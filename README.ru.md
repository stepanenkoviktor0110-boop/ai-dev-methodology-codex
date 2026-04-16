# AI-First Development Methodology v2.0 — Codex-версия

[English version](README.md)

Структурированная AI-First методология разработки для [OpenAI Codex CLI](https://github.com/openai/codex). Каждая фича проходит через спек-пайплайн с автоматическими валидаторами и quality gates на каждом этапе.

## Что это делает

Полный фреймворк разработки, где AI-агенты ведут весь цикл: требования → архитектура → задачи → код → ревью → документация. Ты направляешь процесс, агенты делают работу.

**Какие проблемы решает:**
- **Потеря контекста между сессиями** — распределённая база знаний сохраняется между сессиями
- **Качество без ручного ревью** — автоматические валидаторы на каждом этапе
- **Расползание скоупа** — спеки утверждаются до начала кодирования
- **Устаревшие знания о библиотеках** — Context7 MCP подтягивает актуальную документацию

## Установка

### Требования

- [Codex CLI](https://github.com/openai/codex) установлен и настроен
- [GitHub CLI](https://cli.github.com/) (`gh`) для инициализации проектов
- [gitleaks](https://github.com/gitleaks/gitleaks) для pre-commit сканирования секретов
- Git Bash на Windows (для хуков)

### Шаг 1: Клонировать фреймворк

```bash
git clone https://github.com/stepanenkoviktor0110-boop/ai-dev-methodology-codex.git ~/.agents
```

Это размещает все скиллы, агенты и шаблоны там, где Codex их ожидает (`~/.agents/`).

### Шаг 2: Настроить Codex

Добавить в `~/.codex/config.toml`:

```toml
model = "gpt-5.4"

# Мульти-агентная оркестрация (feature-execution запускает параллельных воркеров + ревьюеров)
[agents]
max_threads = 10                    # параллельные воркеры на волну (по умолчанию 6)
max_depth = 2                       # воркер может запустить ревьюера-субагента
job_max_runtime_seconds = 3600      # 1 час для больших волн

# Хук SessionStart для checkpoint recovery (опционально, для долгих фич)
[[hooks]]
event = "SessionStart"
command = "cat work/*/logs/checkpoint.yml 2>/dev/null || echo 'No active checkpoint'"
```

### Шаг 3: Настроить MCP (опционально, но рекомендуется)

Добавить [Context7](https://github.com/upstash/context7) MCP-сервер для актуальной документации библиотек. В `~/.codex/config.toml`:

```toml
[mcp_servers.context7]
command = "npx"
args = ["-y", "@upstash/context7-mcp"]
```

### Шаг 4: Проверить установку

```bash
# Проверить структуру фреймворка
ls ~/.agents/skills/ ~/.agents/AGENTS.md
# Проверить gitleaks
gitleaks version
```

## Использование

### Новый проект (с нуля)

```
/init-project                  # Шаблон + git + GitHub репо
/init-project-knowledge        # Интервью → заполнить всю документацию проекта
```

После этого будет:
- GitHub репо с ветками `master` + `dev`
- `.agents/skills/project-knowledge/` заполнен документацией
- `AGENTS.md` указывает на базу знаний

### Новая фича (полный пайплайн)

```
/new-user-spec                 # Шаг 1: Интервью → user-spec.md (требования)
                               #   ⛔ пользователь утверждает спек
/new-tech-spec                 # Шаг 2: Исследование → tech-spec.md (архитектура)
                               #   ⛔ пользователь утверждает спек
/decompose-tech-spec           # Шаг 3: Разбивка на задачи → tasks/*.md
                               #   ⛔ GATE 1: пользователь утверждает декомпозицию
                               #   ⛔ GATE 2: пользователь утверждает план сессий (LOC бюджет)
                               #   ⛔ HARD STOP — нет авто-перехода к коду
/do-feature                    # Шаг 4: Выполнение задач по волнам
                               #   ⛔ GATE 3: пользователь подтверждает скоуп сессии + LOC
                               #   ⛔ GATE 4: конец сессии → отчёт + промт + СТОП
/done                          # Шаг 5: Обновить документацию проекта → архивировать фичу
```

Каждый шаг имеет валидаторы и **блокирующие гейты** — ни один шаг не продолжается без явного одобрения пользователя:

| Шаг | Команда | Валидаторы | Гейты | Результат |
|-----|---------|-----------|-------|-----------|
| Требования | `/new-user-spec` | quality + adequacy (2) | пользователь утверждает спек | `user-spec.md` |
| Архитектура | `/new-tech-spec` | skeptic + completeness + security + test + template (5) | пользователь утверждает спек | `tech-spec.md` |
| Задачи | `/decompose-tech-spec` | template + reality (2) | утверждение задач, затем план сессий | `tasks/*.md` + `session-plan.md` |
| Код | `/do-feature` | code + security + test ревьюеры (3) | подтверждение скоупа, СТОП в конце сессии | коммиты |
| Аудит | (авто, последние волны) | holistic code + security + test аудит (3) | — | отчёты аудита |
| QA | (авто, финальная волна) | pre-deploy + deploy + post-deploy | — | проверенная фича |

### Одна задача (ручной контроль)

```
/do-task                       # Выполнить одну задачу с quality gates
```

Используй когда нужно дебажить, итерировать или контролировать выполнение вручную.

### Ad-hoc кодирование (без спека)

```
/write-code                    # TDD-цикл: план → тесты → код → ревью
```

Быстрое кодирование с автоматическим code review. Спеки не нужны.

### Другие команды

| Команда | Назначение |
|---------|-----------|
| `/init-project-knowledge` | Заполнить документацию проекта через интервью |
| `/quick-learning` | Извлечение reasoning-паттернов (авто на границах сессий, ручной в любой момент) |
| `/done` | Финализировать фичу, обновить документацию, архивировать |
| `/sketch` | Лёгкое прототипирование без валидаторов |
| `/pause` | Остановить работу, сохранить состояние сессии |

## Как это работает

### Структура проекта

```
your-project/
├── .agents/                        # Контекст AI-агентов (создаётся /init-project)
│   └── skills/
│       └── project-knowledge/      # База знаний проекта
│           ├── SKILL.md
│           └── references/
│               ├── project.md      # Цель, аудитория, скоуп
│               ├── architecture.md # Стек, структура, модель данных
│               ├── patterns.md     # Конвенции кода, тестирование, бизнес-правила
│               └── deployment.md   # Платформа, CI/CD, мониторинг
├── work/                           # Рабочие элементы фич
│   ├── my-feature/
│   │   ├── user-spec.md           # Требования (для человека)
│   │   ├── tech-spec.md           # Архитектура (для агентов)
│   │   ├── decisions.md           # Решения, принятые при реализации
│   │   ├── tasks/                 # Атомарные файлы задач
│   │   └── logs/                  # Планы сессий, чекпоинты, отчёты ревью
│   └── completed/                 # Архив завершённых фич
├── AGENTS.md                      # Минимальный — указывает на project-knowledge
└── README.md
```

### Глобальный фреймворк (`~/.agents/`)

```
~/.agents/                          # Этот репозиторий
├── skills/                        # 20+ скиллов (методология, выполнение, качество)
├── agents/                        # 20 агентов (валидаторы, ревьюеры, генераторы)
├── shared/
│   ├── work-templates/            # Шаблоны для спеков, задач, сессий
│   ├── templates/                 # Скаффолдинг новых проектов
│   ├── interview-templates/       # Планы интервью (YAML)
│   ├── scripts/                   # Диспетчер, smoke test, init скрипты
│   └── runtime/                   # MCP алиасы
└── AGENTS.md                      # Глобальные настройки
```

### Ключевые принципы

- **Spec-Driven** — пиши спеки до кода. Иерархия: User Spec → Tech Spec → Tasks → Code
- **Blocking Gates** — 6 обязательных HARD STOP в пайплайне. Ни один шаг не продолжается без явного одобрения. Агент никогда не переходит автоматически между декомпозицией → выполнением или между сессиями
- **Многоуровневая валидация** — автоматические валидаторы на каждом этапе (2 → 5 → 2 → 3)
- **Планирование сессий** — волны сгруппированы по ~1200 LOC бюджету на сессию (помещается в контекстное окно). LOC бюджет как communication gate перед каждой сессией/задачей
- **Handoff сессий** — в конце сессии: структурированный отчёт (сделано/не сделано/проверки/риски) + сгенерированный промт для следующей сессии. Выполнение останавливается пока пользователь явно не начнёт следующую сессию
- **Checkpoint Recovery** — `checkpoint.yml` сохраняет состояние; SessionStart хук возобновляет после компактификации
- **Just-In-Time Context** — агенты читают только то, что нужно для текущей задачи
- **Единая система знаний** — triad-based буфер reasoning-patterns.md, pruning, промоушен паттернов в скиллы
- **Quick Learning** — reasoning-паттерны автоматически извлекаются на границах сессий, встраиваются в скиллы через `/skill-trainer`

### Мульти-агентная архитектура

Фреймворк использует встроенную систему субагентов Codex CLI (`spawn_agent`, `wait_agent`, `send_input`, `close_agent`).

**Как `/do-feature` оркестрирует волну:**
```
Оркестратор (feature-execution)
├─ spawn_agent: Worker 1 (task 1, tier_high)     ─┐
├─ spawn_agent: Worker 2 (task 2, tier_high)      ├─ параллельно
├─ spawn_agent: Worker 3 (task 3, tier_high)     ─┘
│
├─ wait_agent: собрать результаты от всех воркеров
│
├─ spawn_agent: code-reviewer (tier_medium)      ─┐
├─ spawn_agent: security-auditor (tier_medium)    ├─ параллельные ревью
├─ spawn_agent: test-reviewer (tier_medium)      ─┘
│
├─ wait_agent: собрать отчёты ревью
├─ исправить findings → перезапустить ревьюеров (макс 3 раунда)
└─ закоммитить волну
```

**Другие паттерны субагентов:**
- `/decompose-tech-spec`: запускает `task-creator` на задачу (параллельно) + `task-validator` + `reality-checker` (параллельно)
- `/new-tech-spec`: запускает 5 валидаторов параллельно (skeptic, completeness, security, test, template)
- `/new-user-spec`: запускает 2 валидатора параллельно (quality, adequacy)

**Конфиг** (`~/.codex/config.toml`):
```toml
[agents]
max_threads = 10    # сколько субагентов могут работать параллельно
max_depth = 2       # воркер (depth 1) может запустить ревьюера (depth 2)
```

### Модельные тиры

| Тир | Модель | Использование |
|-----|--------|--------------|
| `tier_high` | `gpt-5.4` | Воркеры, сложная архитектура, security-критичная работа |
| `tier_medium` | `gpt-5.3-codex` | Ревьюеры, валидаторы, средние задачи (специализированная модель для кода) |
| `tier_low` | `gpt-5.4-mini` (low reasoning) | Простые проверки, форматирование |

### Скиллы и агенты

**Скиллы** содержат методологию (ЧТО делать). **Агенты** добавляют изоляцию + контракты на выход (КАК доставить).

| Категория | Скиллы | Агенты |
|-----------|--------|--------|
| Планирование | user-spec-planning, tech-spec-planning, task-decomposition, project-planning | interview-completeness-checker, task-creator |
| Выполнение | code-writing, feature-execution, pre-deploy-qa, post-deploy-qa | code-researcher |
| Качество | code-reviewing, security-auditor, test-master | code-reviewer, security-auditor, test-reviewer, prompt-reviewer |
| Валидация | — | userspec-quality-validator, userspec-adequacy-validator, tech-spec-validator, skeptic, completeness-validator, task-validator, reality-checker |
| Мета | methodology, quick-learning, documentation-writing, skill-trainer | documentation-reviewer, deploy-reviewer, infrastructure-reviewer, skill-checker |
| Утилиты | sketch, pause, progress | — |

Полные детали любого скилла — в его `SKILL.md`:
```
cat ~/.agents/skills/{skill-name}/SKILL.md
```

## Совместимость с Codex Runtime

Скиллы резолвятся напрямую из SKILL.md — скрипты-диспетчеры не нужны:

- **Shim-скиллы** (напр., `do-feature/SKILL.md`) перенаправляют на полную процедуру в целевом скилле
- **AGENTS.md** — правило "No Scripts": агенты читают и следуют SKILL.md напрямую
- **MCP-алиасы**: `shared/runtime/mcp-aliases.json` — маппинг имён инструментов Context7

## Основано на

Эволюционный форк [molyanov-ai-dev](https://github.com/pavel-molyanov/molyanov-ai-dev) Павла Молянова (MIT License).

## Changelog

### v2.0 — Полная синхронизация с Claude Code (2026-04-16)

Полная синхронизация всех скиллов с основной версией Claude Code (v2.0). Каждый скилл обновлён до последнего контента с адаптацией под Codex CLI.

**Скиллы:**
- **35 скиллов** (было 34): удалены `retrospective`, `skill-master`, `skill-test-designer`, `skill-tester`; добавлены `pause`, `progress`, `project-knowledge`, `sketch`, `skill-trainer`
- **`quick-learning` заменяет `retrospective`** — извлечение reasoning-паттернов запускается автоматически на границах сессий + доступен как ручная команда `/quick-learning`
- **`skill-trainer`** — встраивает накопленные триады из quick-learning в скиллы как постоянные инструкции
- **Все скиллы синхронизированы**: CRITICAL rules, Post-Generation Guards, Promoted/Learned Patterns перенесены из основной версии

**Методология:**
- Пайплайн упрощён: 6 шагов → 5 (retrospective заменён авто-триггером quick-learning)
- Обновлена экосистема скиллов с категорией Utility (sketch, pause, progress)
- `.gitignore` обновлён для исключения артефактов Claude Code

### v1.4 — Единая система знаний (2026-03-27)

- **Unified knowledge system** — единый буфер reasoning-patterns.md с triad-based dedup вместо разрозненных lessons-learned.md
- **Pruning trigger** — автоматическая очистка при >25 записей
- **Mechanical pre-filter** — 3+ content words в Goal = Near match candidate
- **Design categories** — design-taste, design-process, design-iteration в retrospective и quick-learning

### v1.3 — Battle-Tested (2026-03-24)

Все фиксы из запуска полного пайплайна на health-dashboard-mvp (Windows, Codex CLI).

<details>
<summary>Детали v1.3</summary>

**Кросс-платформа и окружение:**
- Обработка непустых директорий в `/infrastructure-setup`
- Кросс-платформенный gitleaks
- Шаблон `.gitleaks.toml`
- Windows/PowerShell секция
- Fallback для `rg` Access denied
- npm install таймауты

**Пайплайн и выполнение:**
- Проверка существования файлов перед каждой волной
- Ясность Audit Wave между сессиями
- `git add -f` для logs/ артефактов
- Обязательный Session End Protocol

**Дисциплина агентов:**
- Строгий тон инструкций (should → MUST)
- Propose-first стиль интервью
- Промежуточные summary на каждом чекпоинте
</details>

<details>
<summary>v1.2 — Blocking Gates (2026-03-24)</summary>

- 6 blocking gates в пайплайне
- Session End Protocol: отчёт + промт + HARD STOP
- Session Scope Confirmation: LOC бюджет перед каждой сессией
- Pre-flight checks в `/do-feature` и `/do-task`
- Нет авто-перехода между этапами
</details>
