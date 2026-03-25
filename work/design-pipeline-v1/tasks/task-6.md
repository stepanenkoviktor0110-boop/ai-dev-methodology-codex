---
status: planned
depends_on: [1, 2]
wave: 2
skills: [skill-master]
verify: [smoke]
reviewers: [skill-checker]
teammate_name:
estimated_loc: 40
---

# Task 6: Update design-system-init SKILL.md

## Required Skills

Перед выполнением задачи загрузи:
- `/skill:skill-master` — навык работы со скиллами

## Description

Расширить `skills/design-system-init/SKILL.md` тремя новыми возможностями: чтение кросс-проектного опыта (designer-experience.md по категории проекта) перед интервью, матчинг стилевого профиля (style-profiles.md) после определения настроения, чтение taste-profile при обновлении существующей дизайн-системы. Также добавить инструкции двухслойных описаний (человеко-читаемое + техническое) и пункты Final Check.

Зависит от задач 1 и 2 — файлы style-profiles.md и designer-experience.md должны существовать для корректных ссылок.

## What to do

1. В Phase 2.1 (Project Context and Mood) — **перед** вопросом пользователю добавить шаг: прочитать секцию designer-experience.md, соответствующую категории проекта (landing/webapp/admin/portfolio), чтобы учесть накопленный опыт при формулировке предложений.
2. В Phase 2.1 — **после** определения настроения добавить шаг: прочитать только соответствующую секцию style-profiles.md (матчинг по mood+category, не весь файл — Decision 4) и использовать рецепты профиля при формировании предложений по палитре и типографике.
3. В Phase 0 — в ветке "update" добавить шаг: прочитать `.design-system/taste-profile.md` (если существует) и использовать предпочтения как контекст для обновления.
4. Добавить инструкцию двухслойных описаний: при предложении решений сначала описывать человеко-понятным языком ("тёплая уютная палитра с акцентом на натуральные тона"), затем технические детали (hex-коды, font-weight, px).
5. Добавить пункты в Final Check: проверка что designer-experience был учтён, style-profile использован, taste-profile прочитан (при update), описания двухслойные.
6. Добавить ссылки на новые reference-файлы с корректными относительными путями (`../../shared/design-references/`).

## Acceptance Criteria

- [ ] Phase 0 (update flow) содержит шаг чтения taste-profile.md
- [ ] Phase 2.1 содержит шаг чтения designer-experience.md по категории проекта перед интервью
- [ ] Phase 2.1 содержит шаг матчинга style-profile по mood после определения настроения
- [ ] Ссылки на designer-experience.md и style-profiles.md используют корректные относительные пути (`../../shared/design-references/`)
- [ ] Инструкция двухслойных описаний присутствует (human-readable first, then technical)
- [ ] Final Check содержит пункты проверки новых шагов
- [ ] Существующие фазы, чекпоинты и ссылки сохранены без изменений
- [ ] Агент читает только соответствующую секцию style-profiles.md, а не весь файл (Decision 4)

## Context Files

- [user-spec.md](../user-spec.md)
- [tech-spec.md](../tech-spec.md)
- [decisions.md](../decisions.md)
- [design-system-init SKILL.md](../../../skills/design-system-init/SKILL.md) — modify
- [style-profiles.md](../../../shared/design-references/style-profiles.md) — read (создаётся Task 1)
- [designer-experience.md](../../../shared/design-references/designer-experience.md) — read (создаётся Task 2)

## Verification Steps

### Smoke

- `grep "designer-experience" skills/design-system-init/SKILL.md` → found
- `grep "style-profiles" skills/design-system-init/SKILL.md` → found
- `grep "taste-profile" skills/design-system-init/SKILL.md` → found

## Details

**Files:** `skills/design-system-init/SKILL.md` — добавить шаги чтения experience/style-profile/taste-profile, двухслойные описания, пункты Final Check
**Dependencies:** Task 1 (style-profiles.md), Task 2 (designer-experience.md) — файлы должны существовать для ссылок
**Edge cases:**
- designer-experience.md может не содержать секцию для данной категории проекта — агент должен продолжить без неё
- taste-profile.md может отсутствовать при update — агент должен продолжить без неё
- style-profiles.md может не содержать точного матча по mood — агент выбирает ближайший профиль
**Implementation hints:**
- Ссылки из skills: `../../shared/design-references/style-profiles.md` (два уровня вверх от skills/design-system-init/)
- Шаг чтения designer-experience вставляется в начало Phase 2.1, до вопроса пользователю
- Шаг матчинга style-profile вставляется после получения ответа на вопрос о настроении в 2.1
- Чтение taste-profile в Phase 0 — только в ветке "update", рядом с загрузкой существующего tokens.json
- Decision 4: читать только секцию matching profile, не весь файл style-profiles.md

## Reviewers

- **skill-checker** → `logs/working/task-6/skill-checker-{round}.json`

## Post-completion

- [ ] Записать краткий отчёт в decisions.md по шаблону (Summary: 1-3 предложения, ревью со ссылками на JSON, без таблиц файндингов и дампов)
- [ ] Если отклонились от спека — описать отклонение и причину
- [ ] Обновить user-spec/tech-spec если что-то изменилось
