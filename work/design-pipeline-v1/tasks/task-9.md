---
status: done
depends_on: [1, 2, 3, 4, 5, 6, 7]
wave: 3
skills: [security-auditor]
verify: []
reviewers: []
teammate_name:
estimated_loc: 0
---

# Task 9: Security Audit

## Required Skills

Перед выполнением задачи загрузи:
- `/skill:security-auditor` — навык аудита безопасности

## Description

Full-feature security audit для design-pipeline-v1. Прочитать все файлы, созданные или изменённые в рамках этой фичи. Проанализировать на:
- **Path traversal** — риски в относительных ссылках между файлами
- **Information leakage** — утечки данных в experience-файлах
- **Unsafe file operations** — небезопасные файловые операции в инструкциях SKILL.md

Это Audit Wave задача — аудитор сам является ревьюером. Отдельные ревьюеры не нужны.

## What to do

1. Прочитать все SKILL.md файлы, затронутые фичей:
   - `skills/design-retrospective/SKILL.md`
   - `skills/design-system-init/SKILL.md`
   - `skills/design-generate/SKILL.md`
   - `skills/design-review/SKILL.md`
   - `skills/code-writing/SKILL.md`
2. Прочитать reference-файлы:
   - `shared/design-references/style-profiles.md`
   - `shared/design-references/designer-experience.md`
3. Проверить все относительные пути и ссылки на path traversal (выход за пределы репо, `../` цепочки).
4. Проверить experience-файлы на information leakage (чувствительные данные, внутренние пути, credentials).
5. Проверить SKILL.md инструкции на unsafe file operations (неограниченная запись, перезапись критичных файлов, shell injection через переменные).
6. Написать отчёт аудита в `logs/working/task-9/security-audit-report.json`.

## Acceptance Criteria

- [ ] Все файлы из списка прочитаны и проанализированы
- [ ] Path traversal риски задокументированы (или отмечено их отсутствие)
- [ ] Information leakage проверен в experience-файлах
- [ ] Unsafe file operations проверены в SKILL.md инструкциях
- [ ] Отчёт записан в `logs/working/task-9/security-audit-report.json`

## Context Files

- [user-spec.md](../user-spec.md)
- [tech-spec.md](../tech-spec.md)
- [decisions.md](../decisions.md)
- [design-retrospective/SKILL.md](../../../skills/design-retrospective/SKILL.md) — read
- [design-system-init/SKILL.md](../../../skills/design-system-init/SKILL.md) — read
- [design-generate/SKILL.md](../../../skills/design-generate/SKILL.md) — read
- [design-review/SKILL.md](../../../skills/design-review/SKILL.md) — read
- [code-writing/SKILL.md](../../../skills/code-writing/SKILL.md) — read
- [style-profiles.md](../../../shared/design-references/style-profiles.md) — read
- [designer-experience.md](../../../shared/design-references/designer-experience.md) — read

## Details

**Files:** (none — audit report only) `logs/working/task-9/security-audit-report.json` — записать отчёт аудита
**Dependencies:** задачи 1–7 (все файлы должны быть созданы до аудита)
**Edge cases:**
- Файлы могут содержать шаблонные переменные (placeholders) — не считать их за уязвимость
- Относительные пути внутри SKILL.md — нормально, если не выходят за пределы репозитория
**Implementation hints:**
- Это Audit Wave — аудитор IS the review, отдельные ревьюеры не назначаются
- Отчёт пишется в JSON-формат, совместимый с форматом ревью-отчётов

## Reviewers

(Audit Wave — аудитор сам является ревьюером. Отчёт: `logs/working/task-9/security-audit-report.json`)

## Post-completion

- [ ] Записать краткий отчёт в decisions.md по шаблону (Summary: 1-3 предложения, ревью со ссылками на JSON, без таблиц файндингов и дампов)
- [ ] Если отклонились от спека — описать отклонение и причину
- [ ] Обновить user-spec/tech-spec если что-то изменилось
