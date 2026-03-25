---
status: planned                    # planned -> in_progress -> done
depends_on: [1, 2, 3, 4, 5, 6, 7] # номера задач-зависимостей
wave: 3                            # волна параллельного выполнения
skills: [test-master]              # МАССИВ скиллов для загрузки
verify: []                         # типы верификации: smoke, user (опционально)
reviewers: []                      # Audit Wave — auditor IS the review
teammate_name:                     # косметическое имя тиммейта для agent teams
estimated_loc: 0                   # оценка строк кода для session planning
---

# Task 10: Test Audit

## Required Skills

Перед выполнением задачи загрузи:
- `/skill:test-master` — [skills/test-master/SKILL.md](~/.claude/skills/test-master/SKILL.md)

## Description

Full-feature test quality audit. Проверяем качество тестового покрытия по всей фиче design-pipeline-v1. Ревьюим Verify-smoke команды во всех задачах, проверяем покрытие Agent Verification Plan, оцениваем полноту scenario testing относительно acceptance criteria.

Это markdown-only фича — scenario testing является основным методом тестирования. Аудитор сам является ревью — отдельные ревьюеры не нужны.

## What to do

1. Прочитать все task-файлы (tasks/1.md через tasks/7.md) — собрать все Verify-smoke команды и acceptance criteria
2. Прочитать tech-spec.md — секции Testing Strategy и Agent Verification Plan
3. Проверить что каждый acceptance criterion из tech-spec покрыт хотя бы одним Verify-smoke или scenario test
4. Проверить что Verify-smoke команды во всех задачах корректны, исполнимы и достаточны
5. Оценить полноту scenario testing — все ли ключевые сценарии покрыты
6. Выявить gaps: непокрытые критерии, слабые проверки, отсутствующие edge cases
7. Записать результаты аудита в отчёт

## Acceptance Criteria

- [ ] Все task-файлы (1–7) проверены на наличие и качество Verify-smoke команд
- [ ] Покрытие Agent Verification Plan из tech-spec проанализировано
- [ ] Scenario testing completeness оценена относительно acceptance criteria
- [ ] Gaps и рекомендации задокументированы
- [ ] Audit report записан в `logs/working/task-10/test-audit-report.json`

## Context Files

- [tech-spec.md](../tech-spec.md) — Testing Strategy + Agent Verification Plan sections
- [tasks/task-1.md](task-1.md)
- [tasks/task-2.md](task-2.md)
- [tasks/task-3.md](task-3.md)
- [tasks/task-4.md](task-4.md)
- [tasks/task-5.md](task-5.md)
- [tasks/task-6.md](task-6.md)
- [tasks/task-7.md](task-7.md)
- [user-spec.md](../user-spec.md)
- [decisions.md](../decisions.md)

## Verification Steps

<!-- Audit task — no automated tests, no smoke checks. The audit report itself is the deliverable. -->

## Details

**Files:** нет файлов для модификации — только audit report
**Output:** `logs/working/task-10/test-audit-report.json`
**Dependencies:** все задачи волн 1–2 (tasks 1–7) должны быть завершены
**Scope:** scenario testing — основной метод тестирования для markdown-only фичи

## Reviewers

<!-- Audit Wave — auditor IS the review. No separate reviewers needed. -->

## Post-completion

- [ ] Записать краткий отчёт в decisions.md по шаблону (Summary: 1-3 предложения, ревью со ссылками на JSON, без таблиц файндингов и дампов)
- [ ] Если отклонились от спека — описать отклонение и причину
- [ ] Обновить user-spec/tech-spec если что-то изменилось
