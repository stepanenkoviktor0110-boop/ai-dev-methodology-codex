---
status: done                    # planned -> in_progress -> done
depends_on: []                     # номера задач-зависимостей
wave: 1                            # волна параллельного выполнения
skills: [skill-master]             # МАССИВ скиллов для загрузки
verify: [smoke]                    # типы верификации: smoke, user (опционально)
reviewers: [skill-checker]         # явно указать. Пусто = fallback на дефолтные три
teammate_name:                     # косметическое имя тиммейта для agent teams
estimated_loc: 10                  # оценка строк кода для session planning
---

# Task 3: Update design-review SKILL.md

## Required Skills

Перед выполнением задачи загрузи:
- `/skill:skill-master` — навык редактирования SKILL.md файлов

## Description

Добавить опциональную инструкцию двухслойного пояснения (two-layer explanation) для неочевидных совпадений токенов в секцию How to Report файла `skills/design-review/SKILL.md`.

Контекст решения (Decision 5): design-review добавляет пояснение **только** для неочевидных (non-obvious) совпадений токенов, в отличие от остальных 3 скиллов, которые используют two-layer всегда. Это минимальное изменение — design-review остаётся лёгким скиллом с лимитом в 3 рекомендации в лаконичном формате "found X → use Y". Обязательный two-layer раздувал бы лёгкий скилл.

Two-layer = сначала человекочитаемое описание, затем технические подробности.

## What to do

1. В секции **How to Report → Findings present** добавить указание: если предложенный токен не является точным совпадением (non-obvious match), дать краткое двухслойное пояснение — сначала почему этот токен подходит (человекочитаемо), затем техническая деталь.
2. Сохранить существующий лаконичный формат `found {what} in {file}:{line} -> use {token}` как основной.
3. Не трогать остальные секции SKILL.md.

## Acceptance Criteria

- [ ] В секции How to Report есть инструкция про two-layer explanation для non-obvious token matches
- [ ] Инструкция явно помечена как опциональная (только для неочевидных совпадений)
- [ ] Существующий формат `found ... -> use ...` не изменён
- [ ] Лимит 3 рекомендации сохранён
- [ ] Остальные секции SKILL.md не затронуты

## Context Files

- [user-spec.md](../user-spec.md)
- [tech-spec.md](../tech-spec.md)
- [decisions.md](../decisions.md)
- [skills/design-review/SKILL.md](../../../skills/design-review/SKILL.md) — файл для изменения

## Verification Steps

### Smoke

- `grep -i "non-obvious\|неочевидн" skills/design-review/SKILL.md` → found

## Details

**Files:** `skills/design-review/SKILL.md` — добавить инструкцию two-layer explanation для non-obvious matches в секцию How to Report
**Dependencies:** нет
**Edge cases:** не добавлять two-layer как обязательный — только опционально для non-obvious matches
**Implementation hints:** вставить после блока примеров в How to Report, перед блоком приоритизации; ~2-4 строки текста достаточно

## Reviewers

- **skill-checker** → `logs/working/task-3/skill-checker-{round}.json`

## Post-completion

- [ ] Записать краткий отчёт в decisions.md по шаблону (Summary: 1-3 предложения, ревью со ссылками на JSON, без таблиц файндингов и дампов)
- [ ] Если отклонились от спека — описать отклонение и причину
- [ ] Обновить user-spec/tech-spec если что-то изменилось
