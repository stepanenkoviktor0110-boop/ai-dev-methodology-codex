# Execution Plan: design-pipeline-v1

**Создан:** 2026-03-26

---

## Wave 1 (независимые)

### Task 1: Create style-profiles.md
- **Skill:** skill-master
- **Reviewers:** skill-checker
- **Verify-smoke:** `grep -c "^## " shared/design-references/style-profiles.md` → 6+

### Task 2: Create designer-experience.md
- **Skill:** skill-master
- **Reviewers:** skill-checker
- **Verify-smoke:** `grep -c "^## " shared/design-references/designer-experience.md` → 4+

### Task 3: Update design-review SKILL.md
- **Skill:** skill-master
- **Reviewers:** skill-checker
- **Verify-smoke:** `grep -i "non-obvious\|неочевидн" skills/design-review/SKILL.md` → found

### Task 4: Update code-writing SKILL.md
- **Skill:** skill-master
- **Reviewers:** skill-checker
- **Verify-smoke:** `grep "tokens.json" skills/code-writing/SKILL.md` → found

## Wave 2 (зависит от Wave 1)

### Task 5: Update design-retrospective SKILL.md
- **Skill:** skill-master
- **Reviewers:** skill-checker
- **Depends on:** Task 2
- **Verify-smoke:** `grep "taste-profile" skills/design-retrospective/SKILL.md` → found; `grep "designer-experience" skills/design-retrospective/SKILL.md` → found

### Task 6: Update design-system-init SKILL.md
- **Skill:** skill-master
- **Reviewers:** skill-checker
- **Depends on:** Tasks 1, 2
- **Verify-smoke:** `grep "designer-experience" skills/design-system-init/SKILL.md` → found; `grep "style-profiles" skills/design-system-init/SKILL.md` → found

### Task 7: Update design-generate SKILL.md
- **Skill:** skill-master
- **Reviewers:** skill-checker
- **Verify-smoke:** `grep "taste-profile" skills/design-generate/SKILL.md` → found

--- Session 1 boundary ---

## Audit Wave (зависит от Wave 1-2)

### Task 8: Code Audit
- **Skill:** code-reviewing
- **Reviewers:** none (auditor IS the review)

### Task 9: Security Audit
- **Skill:** security-auditor
- **Reviewers:** none (auditor IS the review)

### Task 10: Test Audit
- **Skill:** test-master
- **Reviewers:** none (auditor IS the review)

## Final Wave (зависит от Audit Wave)

### Task 11: Pre-deploy QA
- **Skill:** pre-deploy-qa
- **Reviewers:** none (QA is self-verification)

## Проверки, требующие участия пользователя

- [ ] После всех волн: запустить `/design-system-init` в тестовом проекте — агент должен спросить категорию и предложить стилевой профиль
- [ ] После 2-3 сессий `/design-generate` + `/design-retrospective` — проверить что taste-profile.md появился
- [ ] Описания решений понятны без уточнений
