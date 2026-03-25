---
status: done                       # planned -> in_progress -> done
depends_on: [8, 9, 10]            # номера задач-зависимостей
wave: 4                            # волна параллельного выполнения
skills: [pre-deploy-qa]            # МАССИВ скиллов для загрузки
verify: []                         # типы верификации: smoke, user (опционально)
reviewers: []                      # QA is its own verification
teammate_name:                     # косметическое имя тиммейта для agent teams
estimated_loc: 0                   # оценка строк кода для session planning
---

# Task 11: Pre-deploy QA

## Required Skills

Перед выполнением задачи загрузи:
- `/skill:pre-deploy-qa` — [skills/pre-deploy-qa/SKILL.md](~/.claude/skills/pre-deploy-qa/SKILL.md)

## Description

Финальное приёмочное тестирование фичи design-pipeline-v1. Проверить все acceptance criteria из user-spec и tech-spec, выполнить все Verify-smoke проверки из задач 1-7, валидировать резолвинг reference-ссылок во всех SKILL.md, убедиться в отсутствии регрессий в существующих фазах скиллов.

Это Final Wave задача — QA является самостоятельной верификацией, отдельные ревьюеры не нужны. Результат записывается в `logs/working/task-11/qa-report.json`.

## What to do

1. **Acceptance Criteria из user-spec** — пройти каждый пункт из раздела "Критерии приёмки" user-spec.md, проверить выполнение через grep/read
2. **Acceptance Criteria из tech-spec** — пройти каждый пункт из раздела "Acceptance Criteria" tech-spec.md
3. **Verify-smoke из всех задач** — выполнить все smoke-проверки из задач 1-7:
   - Task 1: `grep -c "^## " shared/design-references/style-profiles.md` → 6+
   - Task 2: `grep -c "^## " shared/design-references/designer-experience.md` → 4+; `grep "Антипаттерны" shared/design-references/designer-experience.md`
   - Task 3: `grep -i "non-obvious\|неочевидн" skills/design-review/SKILL.md`
   - Task 4: `grep "tokens.json" skills/code-writing/SKILL.md`; `grep -i "silent\|skip" skills/code-writing/SKILL.md`
   - Task 5: `grep "taste-profile" skills/design-retrospective/SKILL.md`; `grep "designer-experience" skills/design-retrospective/SKILL.md`; `grep "Phase 4.5\|Phase 5" skills/design-retrospective/SKILL.md`; `grep -i "corrupted\|повреждён\|recreate\|пересозда" skills/design-retrospective/SKILL.md`
   - Task 6: `grep "designer-experience" skills/design-system-init/SKILL.md`; `grep "style-profiles" skills/design-system-init/SKILL.md`; `grep "taste-profile" skills/design-system-init/SKILL.md`
   - Task 7: `grep "taste-profile" skills/design-generate/SKILL.md`; `grep -c "^## Phase" skills/design-generate/SKILL.md` → 6
4. **Reference resolution** — для каждого модифицированного SKILL.md извлечь все `[text](path)` ссылки, проверить `bash test -e` что целевые файлы существуют
5. **Phase/checkpoint preservation** — diff каждого SKILL.md против git HEAD, убедиться что существующие фазы и чекпоинты не удалены
6. **Content completeness** — grep каждого SKILL.md на ожидаемые ключевые слова (taste-profile, designer-experience, style-profiles, two-layer, tokens.json)
7. **Structure validation** — проверить:
   - designer-experience.md имеет 4 секции категорий
   - style-profiles.md имеет 6+ профилей
   - taste-profile template в retrospective содержит обязательные секции
8. **Audit report review** — прочитать отчёты аудиторов из задач 8-10, убедиться что все critical/high findings устранены
9. Записать результаты в `logs/working/task-11/qa-report.json`

## Acceptance Criteria

- [ ] Все acceptance criteria из user-spec (16 пунктов) проверены и отмечены pass/fail
- [ ] Все acceptance criteria из tech-spec (9 пунктов) проверены и отмечены pass/fail
- [ ] Все Verify-smoke проверки из задач 1-7 выполнены и прошли
- [ ] Все reference-ссылки во всех модифицированных SKILL.md резолвятся в существующие файлы
- [ ] Ни одна существующая фаза или чекпоинт не удалена из модифицированных SKILL.md
- [ ] designer-experience.md имеет 4 категории (landing, webapp, admin, portfolio) с подсекциями
- [ ] style-profiles.md имеет 6+ стилевых профилей с конкретными приёмами
- [ ] Двухслойные описания присутствуют во всех 4 дизайн-скиллах
- [ ] code-writing содержит условный шаг с dual-condition (tokens.json + UI files)
- [ ] qa-report.json записан в logs/working/task-11/

## Context Files

- [user-spec.md](../user-spec.md)
- [tech-spec.md](../tech-spec.md)
- [decisions.md](../decisions.md)
- [skills/design-retrospective/SKILL.md](../../../skills/design-retrospective/SKILL.md)
- [skills/design-system-init/SKILL.md](../../../skills/design-system-init/SKILL.md)
- [skills/design-generate/SKILL.md](../../../skills/design-generate/SKILL.md)
- [skills/design-review/SKILL.md](../../../skills/design-review/SKILL.md)
- [skills/code-writing/SKILL.md](../../../skills/code-writing/SKILL.md)
- [shared/design-references/style-profiles.md](../../../shared/design-references/style-profiles.md)
- [shared/design-references/designer-experience.md](../../../shared/design-references/designer-experience.md)
- [skills/design-generate/references/component-patterns.md](../../../skills/design-generate/references/component-patterns.md)
- [skills/design-generate/references/grid-techniques.md](../../../skills/design-generate/references/grid-techniques.md)
- [skills/design-generate/references/generation-guide.md](../../../skills/design-generate/references/generation-guide.md)

## Verification Steps

### Smoke

QA задача является самостоятельной верификацией. Все проверки описаны в секции "What to do".

## Details

**Files:** нет файлов для модификации — только чтение и валидация
**Dependencies:** задачи 8, 9, 10 (аудиты) должны быть завершены, findings устранены
**Edge cases:** если аудиты выявили critical findings — проверить что исправления внесены до QA
**Implementation hints:** все пути относительно `C:/tmp/ai-dev-repo/`. Для reference resolution — извлечь ссылки regex `\[.*?\]\((.*?)\)`, нормализовать пути относительно файла, проверить `test -e`. Для phase preservation — сравнить количество `^## Phase` / `^### Phase` заголовков до и после.

## Reviewers

Нет — QA Final Wave является самостоятельной верификацией.

## Post-completion

- [ ] Записать краткий отчёт в decisions.md по шаблону (Summary: 1-3 предложения, ревью со ссылками на JSON, без таблиц файндингов и дампов)
- [ ] Если отклонились от спека — описать отклонение и причину
- [ ] Обновить user-spec/tech-spec если что-то изменилось
