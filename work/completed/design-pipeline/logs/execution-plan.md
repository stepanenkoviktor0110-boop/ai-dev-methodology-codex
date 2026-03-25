# Execution Plan: design-pipeline

**Создан:** 2026-03-25

---

## Session 1: Shared References & Font Pairing

### Wave 1 (независимые)

#### Task 1: Create shared design-references directory
- **Skill:** skill-master
- **Reviewers:** skill-checker
- **Verify-smoke:** `ls -la shared/design-references/` — 4 файла существуют и не пусты

#### Task 2: Create font-pairing.md reference
- **Skill:** skill-master
- **Reviewers:** skill-checker

---

## Session 2: Core Design Skills

### Wave 2 (зависит от Wave 1)

#### Task 3: Create design-system-init skill
- **Skill:** skill-master
- **Reviewers:** skill-checker
- **Verify-smoke:** spawn agent с промтом "create design system for a calm spa website"

#### Task 4: Create design-generate skill
- **Skill:** skill-master
- **Reviewers:** skill-checker
- **Verify-smoke:** spawn agent с промтом "generate a login page"

#### Task 5: Create design-review skill
- **Skill:** skill-master
- **Reviewers:** skill-checker

#### Task 6: Create design-retrospective skill
- **Skill:** skill-master
- **Reviewers:** skill-checker

---

## Session 3: Integration, Audit & QA

### Wave 3 (зависит от Wave 2)

#### Task 7: Integrate design-review into do-task
- **Skill:** skill-master
- **Reviewers:** skill-checker
- **Verify-smoke:** читаем do-task SKILL.md — design-review hook между Step 2.3 и 2.4

#### Task 8: Delete like-figma skill
- **Skill:** code-writing
- **Reviewers:** code-reviewer, security-auditor, test-reviewer
- **Verify-smoke:** like-figma не существует; все 4 новых скилла + shared references на месте

### Wave 4 — Audit Wave

#### Task 9: Code Audit
- **Skill:** code-reviewing
- **Reviewers:** none (аудитор IS ревью)

#### Task 10: Security Audit
- **Skill:** security-auditor
- **Reviewers:** none

#### Task 11: Test Audit
- **Skill:** test-master
- **Reviewers:** none

### Wave 5 — Final Wave

#### Task 12: Pre-deploy QA
- **Skill:** pre-deploy-qa
- **Reviewers:** none

---

## Проверки, требующие участия пользователя

- [ ] После Session 2: визуальная проверка HTML-компонентов в браузере (если будет smoke test)
- [ ] После всех волн: финальное тестирование на реальном проекте
