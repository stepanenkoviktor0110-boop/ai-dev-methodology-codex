---
status: planned
depends_on: [1, 2]
wave: 2
skills: [skill-master]
verify: [smoke]
reviewers: [skill-checker]
teammate_name:
estimated_loc: 20
---

# Task 7: Update design-generate SKILL.md

## Required Skills

Перед выполнением задачи загрузи:
- `/skill:skill-master` — навык редактирования SKILL.md файлов

## Description

Обновить `skills/design-generate/SKILL.md` — добавить поддержку taste-profile в пайплайн генерации дизайна. Taste-profile содержит предпочтения пользователя по цветам, типографике, стилю и анти-паттернам. Его нужно читать в Phase 0 и использовать как контекст в Phase 2 (выбор лейаута) и Phase 3 (сборка). Также добавить two-layer description инструкцию в Phase 4 и обновить Final Check.

## What to do

1. **Phase 0 — добавить шаг чтения taste-profile** (после парсинга tokens.json, перед Checkpoint):
   - Проверить наличие `.design-system/taste-profile.md`
   - Если файл существует — прочитать и извлечь предпочтения: color preferences, typography, style+boldness, anti-patterns
   - Если файла нет — продолжить без него (первая сессия, профиль ещё не создан)
   - Передать предпочтения как контекст в Phase 2 и Phase 3

2. **Phase 2 — учитывать taste-profile при выборе лейаута**:
   - Если taste-profile загружен — учитывать style и boldness предпочтения при выборе между basic и advanced layouts
   - Упомянуть в описании фазы, что taste-profile влияет на выбор

3. **Phase 3 — учитывать taste-profile при сборке**:
   - Применять color preferences и typography из taste-profile при сборке компонентов
   - Anti-patterns из taste-profile — избегать указанных паттернов

4. **Phase 4 — добавить two-layer description**:
   - После генерации страницы создавать двухуровневое описание:
     - Layer 1: краткое описание (что за страница, какой лейаут)
     - Layer 2: детальное описание (компоненты, цвета, типографика, отступы)
   - Описание сохраняется рядом с файлами для будущих итераций

5. **Final Check — добавить пункты проверки**:
   - taste-profile прочитан (если существует) и предпочтения применены
   - Two-layer description сгенерировано для финального варианта

6. **Все существующие фазы (0-5) должны быть сохранены** — только дополнения, без удаления или переименования.

## Acceptance Criteria

- [ ] Phase 0 содержит шаг чтения taste-profile.md после парсинга tokens.json
- [ ] Phase 0 описывает graceful degradation (файла нет → продолжить без него)
- [ ] Phase 2 упоминает использование taste-profile при выборе лейаута
- [ ] Phase 3 упоминает использование taste-profile при сборке
- [ ] Phase 4 содержит инструкцию two-layer description
- [ ] Final Check содержит пункты про taste-profile и two-layer description
- [ ] Все 6 оригинальных фаз (Phase 0 – Phase 5) сохранены без переименования

## Context Files

- [user-spec.md](../user-spec.md)
- [tech-spec.md](../tech-spec.md)
- [decisions.md](../decisions.md)
- [SKILL.md](../../skills/design-generate/SKILL.md)

## Verification Steps

### Smoke

- `grep "taste-profile" skills/design-generate/SKILL.md` → found
- `grep -c "^## Phase" skills/design-generate/SKILL.md` → 6 (all original phase headers preserved)

## Details

**Files:** `skills/design-generate/SKILL.md` — добавить taste-profile reading, two-layer description, Final Check items
**Files to read:** `skills/design-generate/SKILL.md` — текущее содержимое для понимания структуры
**Dependencies:** wave 2 per tech-spec (taste-profile format is in tech-spec Data Models, no direct file dependency on tasks 1-2)
**Edge cases:**
- taste-profile.md не существует → graceful degradation, продолжить без него
- taste-profile.md существует но пустой → обработать как отсутствующий
**Implementation hints:**
- Taste-profile structure (из tech-spec Data Models): color preferences, typography, style+boldness, anti-patterns, change history
- Вставка в Phase 0 идёт после пункта 2 (Parse tokens into CSS custom properties) и перед Checkpoint
- Two-layer description: Layer 1 = краткое (тип страницы + лейаут), Layer 2 = детальное (компоненты, стили, параметры)
- Не менять существующий текст фаз — только добавлять новые пункты/абзацы

## Reviewers

- **skill-checker** → `logs/working/task-7/skill-checker-{round}.json`

## Post-completion

- [ ] Записать краткий отчёт в decisions.md по шаблону (Summary: 1-3 предложения, ревью со ссылками на JSON, без таблиц файндингов и дампов)
- [ ] Если отклонились от спека — описать отклонение и причину
- [ ] Обновить user-spec/tech-spec если что-то изменилось
