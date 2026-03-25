---
status: done
depends_on: []
wave: 1
skills: [skill-master]
verify: [smoke]
reviewers: [skill-checker]
teammate_name:
estimated_loc: 200
---

# Task 1: Create style-profiles.md

## Required Skills

Перед выполнением задачи загрузи:
- `/skill:skill-master` — навык работы со скиллами

## Description

Создать справочный файл `shared/design-references/style-profiles.md` с 6+ стилевыми профилями (luxury, brutalist, editorial, minimal, playful, corporate). Каждый профиль содержит конкретные дизайн-рецепты — наборы техник, которые агент design-system-init использует при генерации дизайн-системы после интервью с пользователем.

Файл читается агентом секционно: загружается только секция выбранного стиля, а не весь файл. Поэтому каждый профиль должен быть самодостаточным и уложиться в ~200 слов.

## What to do

1. Изучить существующие reference-файлы `color-psychology.md` и `color-principles.md` для согласованности терминологии и структуры.
2. Создать файл `shared/design-references/style-profiles.md`.
3. Добавить 6+ стилевых профилей, каждый как `## Heading`:
   - **Luxury** — большие пустоты, serif заголовки, приглушённые цвета, тонкие линии, асимметрия
   - **Brutalist** — грубая типографика, высокий контраст, монохром, raw элементы
   - **Editorial** — журнальная сетка, крупные изображения, выверенная типографика
   - **Minimal** — максимум воздуха, ограниченная палитра, простые формы
   - **Playful** — яркие цвета, скруглённые формы, анимации, нестандартные layouts
   - **Corporate** — надёжность, читаемость, нейтральные цвета, стандартная сетка
4. Каждый профиль должен содержать конкретные дизайн-техники: типографика, цветовая палитра, spacing, формы, характерные приёмы.
5. Ограничить каждый профиль ~200 словами.

## Acceptance Criteria

- [ ] Файл `shared/design-references/style-profiles.md` создан
- [ ] Содержит минимум 6 стилевых профилей (6+ секций `## `)
- [ ] Каждый профиль содержит конкретные дизайн-техники (не абстрактные описания)
- [ ] Каждый профиль самодостаточен (можно читать изолированно)
- [ ] Каждый профиль ~200 слов (не превышает существенно)
- [ ] Терминология согласована с `color-psychology.md` и `color-principles.md`

## Context Files

- [user-spec.md](../user-spec.md)
- [tech-spec.md](../tech-spec.md)
- [decisions.md](../decisions.md)
- [color-psychology.md](../../../shared/design-references/color-psychology.md) — read
- [color-principles.md](../../../shared/design-references/color-principles.md) — read

## Verification Steps

### Smoke

- `grep -c "^## " shared/design-references/style-profiles.md` → 6+

## Details

**Files:** (new) `shared/design-references/style-profiles.md` — создать файл с 6+ стилевыми профилями
**Dependencies:** нет
**Edge cases:**
- Профили не должны пересекаться слишком сильно — каждый должен давать уникальный набор техник
- Названия профилей должны быть на английском (используются как ключи при матчинге)
**Implementation hints:**
- Структура каждого профиля: краткое описание эстетики → типографика → цвета → spacing/layout → характерные приёмы
- Ориентироваться на формат существующих reference-файлов в `shared/design-references/`
- Профили — это рецепты для агента, не документация для человека; писать конкретно и actionable

## Reviewers

- **skill-checker** → `logs/working/task-1/skill-checker-{round}.json`

## Post-completion

- [ ] Записать краткий отчёт в decisions.md по шаблону (Summary: 1-3 предложения, ревью со ссылками на JSON, без таблиц файндингов и дампов)
- [ ] Если отклонились от спека — описать отклонение и причину
- [ ] Обновить user-spec/tech-spec если что-то изменилось
