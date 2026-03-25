---
status: done
depends_on: []
wave: 1
skills: [skill-master]
verify: [smoke]
reviewers: [skill-checker]
teammate_name:
estimated_loc: 15
---

# Task 4: Update code-writing SKILL.md

## Required Skills

Перед выполнением задачи загрузи:
- `/skill:skill-master` — навык редактирования скиллов

## Description

Добавить условный шаг интеграции DS-токенов в конец Phase 1, step 2 (Read Project Context) скилла code-writing. Это лёгкая проактивная интеграция, дополняющая реактивную проверку design-review.

Логика: если задача затрагивает UI-файлы (.tsx, .vue, .html, .css, .scss) И в проекте существует tokens.json → прочитать токены, использовать CSS-переменные из них. Если условия не выполняются → тихий пропуск, нулевой overhead. Если tokens.json невалиден → тихий пропуск, не блокировать кодинг.

Decision 6 из tech-spec: ~500-1000 токенов overhead только когда tokens.json существует И задача — UI.

## What to do

1. Открыть `skills/code-writing/SKILL.md`
2. Найти Phase 1, step 2 "Read Project Context (Graceful)"
3. В конец step 2 (после параграфа "No project patterns?") добавить условный параграф про DS-токены:
   - Условие: задача затрагивает UI-файлы (.tsx, .vue, .html, .css, .scss) И существует tokens.json
   - Действие: прочитать tokens.json, использовать CSS-переменные для цветов/spacing/typography
   - Если tokens.json не существует или невалиден → silent skip, не блокировать кодинг
4. Формулировка должна быть лаконичной (~10-15 строк), в стиле остальных параграфов step 2

## Acceptance Criteria

- [ ] В `skills/code-writing/SKILL.md` Phase 1, step 2 содержит условный параграф про tokens.json
- [ ] Упомянуты UI-расширения файлов: .tsx, .vue, .html, .css, .scss
- [ ] Явно указано: silent skip если tokens.json не существует или невалиден
- [ ] Стиль и форматирование консистентны с остальным SKILL.md
- [ ] Не нарушена существующая структура Phase 1

## Context Files

- [user-spec.md](../user-spec.md)
- [tech-spec.md](../tech-spec.md)
- [decisions.md](../decisions.md)
- [skills/code-writing/SKILL.md](../../../skills/code-writing/SKILL.md)

## Verification Steps

### Smoke

- `grep "tokens.json" skills/code-writing/SKILL.md` → found
- `grep -i "silent\|skip" skills/code-writing/SKILL.md` → found

## Details

**Files:** `skills/code-writing/SKILL.md` — добавить условный параграф в конец Phase 1, step 2

**Implementation hints:**
- Параграф добавляется после "**No project patterns?** Apply baseline from..." и перед step 3 "Analyze & Review Approach"
- Использовать bold-заголовок в стиле соседних параграфов (например: **UI task with Design System?**)
- Условие проверки: файлы задачи содержат UI-расширения + tokens.json существует в проекте
- При невалидном tokens.json — silent skip, не блокировать основной процесс кодинга
- Overhead: ~500-1000 токенов только при срабатывании условия

## Reviewers

- **skill-checker** → `logs/working/task-4/skill-checker-{round}.json`

## Post-completion

- [ ] Записать краткий отчёт в decisions.md по шаблону (Summary: 1-3 предложения, ревью со ссылками на JSON, без таблиц файндингов и дампов)
- [ ] Если отклонились от спека — описать отклонение и причину
- [ ] Обновить user-spec/tech-spec если что-то изменилось
