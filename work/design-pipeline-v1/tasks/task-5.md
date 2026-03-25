---
status: planned
depends_on: [2]
wave: 2
skills: [skill-master]
verify: [smoke]
reviewers: [skill-checker]
teammate_name:
estimated_loc: 50
---

# Task 5: Update design-retrospective SKILL.md

## Required Skills

Перед выполнением задачи загрузи:
- `/skill:skill-master` — навык работы со скиллами

## Description

Модифицировать `skills/design-retrospective/SKILL.md` — добавить "write path" для накопления вкусовых предпочтений и кросс-проектного опыта. Это самая критичная модификация фичи: именно design-retrospective является основным источником записи в taste-profile.md и designer-experience.md.

Ключевые изменения:
- **Phase 4.5** (новая) — после промоушена принципов: извлечение эстетических предпочтений → запись в `.design-system/taste-profile.md`, определение категории проекта → append в `shared/design-references/designer-experience.md`.
- **Phase 3** — инструкция двухслойного описания (human-readable + technical) при записи уроков.
- **Phase 5** — секция taste-profile в шаблоне next-session промпта.
- **Self-Verification** — новые пункты проверки для taste-profile и experience записей.

## What to do

1. Прочитать текущий `skills/design-retrospective/SKILL.md` и `shared/design-references/designer-experience.md` (формат категорий).
2. В **Phase 3** (Write Lessons) добавить инструкцию двухслойного описания: сначала описание понятным языком (что и почему), затем технические детали (конкретные значения токенов, CSS-свойства). Инструкция добавляется к шагу записи уроков.
3. Добавить новую **Phase 4.5: Write Taste Profile & Experience** между Phase 4 (Promote Principles) и Phase 5 (Generate Next-Session Prompt):
   - **Taste-profile запись:** извлечь из сессии эстетические предпочтения → записать/обновить `.design-system/taste-profile.md` по структуре из tech-spec (цветовые предпочтения, типографика, стиль+смелость, антипаттерны, история изменений).
   - **Experience запись:** определить категорию проекта (landing, webapp, admin, portfolio) → append обобщаемые уроки в соответствующую секцию `shared/design-references/designer-experience.md`.
   - Обработка edge cases: missing taste-profile (create), corrupted/повреждённый (recreate с предупреждением), missing designer-experience (create), missing category section (create).
   - Конфликтующие предпочтения: latest-wins, старая запись помечается "пересмотрено" с датой.
4. В **Phase 5** (Generate Next-Session Prompt) добавить секцию `## Вкусовой профиль` в шаблон промпта — краткое резюме из taste-profile.md.
5. В **Self-Verification** добавить пункты:
   - taste-profile.md содержит все обязательные секции
   - experience запись добавлена append-only (не перезаписывает существующее)
   - конфликтующие предпочтения помечены "пересмотрено"
   - edge cases обработаны (missing/corrupted файлы)
6. Обновить description в YAML frontmatter скилла, включив taste-profile и experience writing.
7. Убедиться, что все существующие фазы, чекпоинты и ссылки сохранены без изменений.

## Acceptance Criteria

- [ ] Phase 4.5 добавлена с полным описанием taste-profile записи и experience записи
- [ ] taste-profile.md шаблон включает все секции: цветовые предпочтения, типографика, стиль+смелость, антипаттерны, история изменений
- [ ] Edge cases обработаны: missing taste-profile (create), corrupted (recreate с предупреждением), missing designer-experience (create), missing category section (create)
- [ ] Append-only для designer-experience.md (не перезаписывает существующие записи)
- [ ] Latest-wins для конфликтующих предпочтений — пометка "пересмотрено" с датой
- [ ] Phase 3 содержит инструкцию двухслойного описания (human-readable + technical)
- [ ] Phase 5 шаблон промпта включает секцию taste-profile
- [ ] Self-Verification содержит пункты для taste-profile и experience
- [ ] Все существующие фазы, чекпоинты и ссылки сохранены (нет регрессии)
- [ ] Ссылка на `designer-experience.md` использует корректный относительный путь (`../../shared/design-references/designer-experience.md`)

## Context Files

- [user-spec.md](../user-spec.md)
- [tech-spec.md](../tech-spec.md)
- [decisions.md](../decisions.md)
- [design-retrospective SKILL.md](../../../skills/design-retrospective/SKILL.md) — modify
- [designer-experience.md](../../../shared/design-references/designer-experience.md) — read (формат категорий)

## Verification Steps

### Smoke

- `grep "taste-profile" skills/design-retrospective/SKILL.md` → found
- `grep "designer-experience" skills/design-retrospective/SKILL.md` → found
- `grep "Phase 4.5\|Phase 5" skills/design-retrospective/SKILL.md` → both found
- `grep -i "corrupted\|повреждён\|recreate\|пересозда" skills/design-retrospective/SKILL.md` → found (edge case handling)

## Details

**Files:** `skills/design-retrospective/SKILL.md` — добавить Phase 4.5, двухслойные описания в Phase 3, taste-profile секцию в Phase 5, пункты Self-Verification
**Dependencies:** Task 1 (style-profiles.md) и Task 2 (designer-experience.md) — нужен формат designer-experience.md для корректной записи
**Edge cases:**
- taste-profile.md отсутствует — создать с полной структурой из tech-spec
- taste-profile.md повреждён/corrupted (невалидная структура) — пересоздать с предупреждением пользователю
- designer-experience.md отсутствует — создать с базовой структурой категорий
- Секция категории отсутствует в designer-experience.md — создать секцию
- Конфликтующие предпочтения (тёплые → холодные тона) — latest-wins, старое помечается "пересмотрено" с датой (Decision 7)
**Implementation hints:**
- Структура taste-profile.md определена в tech-spec, секция Data Models
- Append-only для experience (Decision 3) — всегда добавлять в конец секции категории
- Phase 4.5 располагается строго между Phase 4 (Promote Principles) и Phase 5 (Generate Next-Session Prompt)
- Двухслойное описание: сначала "Тёплые приглушённые тона создают ощущение уюта", затем "hsl(30, 40%, 85%) → --color-bg-warm"
- Не удалять и не переименовывать существующие фазы — Phase 5 остаётся Phase 5 (не становится Phase 6)

## Reviewers

- **skill-checker** → `logs/working/task-5/skill-checker-{round}.json`

## Post-completion

- [ ] Записать краткий отчёт в decisions.md по шаблону (Summary: 1-3 предложения, ревью со ссылками на JSON, без таблиц файндингов и дампов)
- [ ] Если отклонились от спека — описать отклонение и причину
- [ ] Обновить user-spec/tech-spec если что-то изменилось
