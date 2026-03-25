---
status: planned                    # planned -> in_progress -> done
depends_on: []                     # номера задач-зависимостей
wave: 1                            # волна параллельного выполнения
skills: [skill-master]             # МАССИВ скиллов для загрузки
verify: [smoke]                    # типы верификации: smoke, user (опционально)
reviewers: [skill-checker]         # явно указать. Пусто = fallback на дефолтные три
teammate_name:                     # косметическое имя тиммейта для agent teams
estimated_loc: 80                  # оценка строк кода для session planning
---

# Task 2: Create designer-experience.md

## Required Skills

Перед выполнением задачи загрузи:
- `/skill:skill-master` — [skills/skill-master/SKILL.md](~/.claude/skills/skill-master/SKILL.md)

## Description

Создать файл `shared/design-references/designer-experience.md` — глобальную (кросс-проектную) базу знаний дизайнерского опыта. Файл организован по 4 категориям проектов (Landing, Webapp, Admin, Portfolio). Каждая категория содержит подсекции: Предпочтения, Что работало, Антипаттерны.

Файл создаётся как scaffold (пустая структура) — наполняется скиллом design-retrospective во время реальной работы. Режим записи — append-only: записи добавляются в конец секции категории, никогда не перезаписываются (Decision 3 из tech-spec). Агент при чтении ищет только свою секцию по категории проекта.

## What to do

1. Создать файл `shared/design-references/designer-experience.md`
2. Добавить заголовок файла и описание формата (append-only, by category)
3. Создать 4 секции второго уровня (`## Landing`, `## Webapp`, `## Admin`, `## Portfolio`)
4. В каждой секции создать 3 подсекции третьего уровня:
   - `### Предпочтения` — что пользователь предпочитает для этой категории
   - `### Что работало` — решения, которые хорошо себя показали
   - `### Антипаттерны` — что не работало, чего избегать
5. Каждую подсекцию оставить пустой (только placeholder-комментарий)

## Acceptance Criteria

- [ ] Файл `shared/design-references/designer-experience.md` существует
- [ ] 4 секции второго уровня: Landing, Webapp, Admin, Portfolio
- [ ] В каждой секции 3 подсекции: Предпочтения, Что работало, Антипаттерны
- [ ] Файл содержит описание формата и правил append-only
- [ ] Формат совместим с чтением агентом по категории (grep по `## Category`)

## Context Files

- [user-spec.md](../user-spec.md)
- [tech-spec.md](../tech-spec.md)
- [decisions.md](../decisions.md)
- [color-psychology.md](../../../shared/design-references/color-psychology.md) — пример существующего файла в shared/design-references/

## Verification Steps

### Smoke

- `grep -c "^## " shared/design-references/designer-experience.md` → 4+ (4 категории)
- `grep "Антипаттерны" shared/design-references/designer-experience.md` → found (подсекции существуют)

## Details

**Files:** `shared/design-references/designer-experience.md` — создать новый файл (scaffold)
**Dependencies:** нет (wave 1, depends_on пуст)
**Edge cases:**
- Файл должен быть чисто scaffold — никакого реального контента, только структура и placeholder-комментарии
- Формат заголовков должен быть единообразным для надёжного парсинга агентом (exact match `## Landing` и т.д.)
**Implementation hints:**
- Посмотреть существующие файлы в `shared/design-references/` для консистентности стиля
- Append-only означает: design-retrospective добавляет записи в конец подсекции, не трогая существующие

## Reviewers

- **skill-checker** → `logs/working/task-2/skill-checker-{round}.json`

## Post-completion

- [ ] Записать краткий отчёт в decisions.md по шаблону (Summary: 1-3 предложения, ревью со ссылками на JSON, без таблиц файндингов и дампов)
- [ ] Если отклонились от спека — описать отклонение и причину
- [ ] Обновить user-spec/tech-spec если что-то изменилось
