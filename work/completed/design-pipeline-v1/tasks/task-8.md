---
status: done
depends_on: [1, 2, 3, 4, 5, 6, 7]
wave: 3
skills: [code-reviewing]
verify: []
reviewers: []
teammate_name:
estimated_loc: 0
---

# Task 8: Code Audit

## Required Skills

Перед выполнением задачи загрузи:
- `/skill:code-reviewing` — [skills/code-reviewing/SKILL.md](~/.claude/skills/code-reviewing/SKILL.md)

## Description

Full-feature code quality audit для design-pipeline-v1. Читаем все SKILL.md и reference-файлы, созданные или изменённые в этой фиче. Проверяем кросс-компонентные проблемы: единообразие терминологии, корректность относительных путей, ясность инструкций, полноту чекпоинтов. Результат — audit report.

Это Audit Wave задача — аудитор сам является ревью. Отдельные ревьюеры не нужны.

## What to do

1. Прочитать все файлы из списка Files to read (ниже) — это полный набор артефактов фичи.
2. Проверить каждый файл на:
   - Корректность и консистентность терминологии между скиллами
   - Правильность относительных путей (все ссылки между файлами должны работать)
   - Ясность и однозначность инструкций
   - Полноту чекпоинтов и acceptance criteria
   - Соответствие общему стилю и паттернам проекта
3. Проверить кросс-компонентную интеграцию:
   - Пайплайн design-retrospective → design-system-init → design-generate → design-review работает как цепочка
   - Входы/выходы скиллов стыкуются
   - Ссылки на shared ресурсы (style-profiles, designer-experience) корректны из всех скиллов
4. Написать audit report в формате code-review-report.json

## Acceptance Criteria

- [ ] Все файлы из списка прочитаны и проверены
- [ ] Терминология консистентна между всеми скиллами пайплайна
- [ ] Все относительные пути между файлами корректны
- [ ] Инструкции в SKILL.md ясны и однозначны
- [ ] Чекпоинты полны и проверяемы
- [ ] Audit report записан в `logs/working/task-8/code-review-report.json`

## Context Files

- [user-spec.md](../user-spec.md)
- [tech-spec.md](../tech-spec.md)
- [decisions.md](../decisions.md)

## Files to read

- `skills/design-retrospective/SKILL.md`
- `skills/design-system-init/SKILL.md`
- `skills/design-generate/SKILL.md`
- `skills/design-review/SKILL.md`
- `skills/code-writing/SKILL.md`
- `shared/design-references/style-profiles.md`
- `shared/design-references/designer-experience.md`

## Verification Steps

### Automated

Нет автоматических тестов — задача аудиторская.

## Details

**Files:** нет файлов для модификации — только audit report
**Output:** `logs/working/task-8/code-review-report.json`
**Scope:** holistic cross-component review всех артефактов design-pipeline-v1

Аудит должен покрывать:
- **Terminology consistency** — одинаковые концепции называются одинаково во всех скиллах
- **Path correctness** — все relative paths резолвятся корректно
- **Instruction clarity** — инструкции можно выполнить без дополнительных вопросов
- **Checkpoint completeness** — все важные шаги имеют чекпоинты
- **Integration correctness** — скиллы корректно ссылаются друг на друга и на shared ресурсы

## Reviewers

Audit Wave — аудитор сам является ревью. Report → `logs/working/task-8/code-review-report.json`

## Post-completion

- [ ] Записать краткий отчёт в decisions.md по шаблону (Summary: 1-3 предложения, ревью со ссылками на JSON, без таблиц файндингов и дампов)
- [ ] Если отклонились от спека — описать отклонение и причину
- [ ] Обновить user-spec/tech-spec если что-то изменилось
