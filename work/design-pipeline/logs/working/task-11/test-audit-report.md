# Test Audit Report: design-pipeline

**Date:** 2026-03-25
**Auditor:** test-master (test-auditor role)
**Feature:** design-pipeline (L-size, 4 markdown skills)
**Testing method:** Scenario tests via skill-test-designer / skill-tester pipeline

---

## Executive Summary

The testing strategy defines 22 scenario tests across 4 skills. The approach of scenario testing is appropriate for a feature consisting entirely of markdown-based skills with no executable code. Coverage of happy paths and primary error cases is solid. However, the audit identifies **7 coverage gaps** in edge cases, boundary conditions, and cross-skill interactions that reduce confidence in full behavioral coverage.

**Status: needs_improvement** (0 critical, 2 major, 5 minor findings)

---

## Per-Skill Coverage Assessment

### design-system-init (6 scenarios)

| # | Scenario | Covers | Assessment |
|---|----------|--------|------------|
| 1 | Fresh project, no styles | Happy path, Phase 1-3 from scratch | Good |
| 2 | Project with Tailwind config | Source priority #1 extraction | Good |
| 3 | `.design-system/` already exists | Phase 0 conflict resolution (update/recreate/cancel) | Good |
| 4 | Non-web project | Phase 0 readiness gate | Good |
| 5 | Conflicting style sources | Phase 1 conflict detection + user resolution | Good |
| 6 | CSS vars without Tailwind | Source priority #2 extraction | Good |

**Coverage grade: Good.** All 4 phases are touched. Phase 0 gates (non-web, existing DS) covered. Phase 1 scan sources covered for Tailwind, CSS vars, and "no styles" cases.

**Gaps identified:**
- SCSS variable extraction (source priority #3) has no dedicated scenario
- Hardcoded repeated values in component files (source priority #4) has no dedicated scenario
- Phase 4 Verify: contrast ratio failure path not explicitly tested (what happens when contrast < 4.5:1?)
- Phase 4 Verify: invalid file names in component names not tested
- Interview flow: user rejecting proposed values and providing alternatives not explicitly tested (only "confirms or adjusts" implied)

### design-generate (7 scenarios)

| # | Scenario | Covers | Assessment |
|---|----------|--------|------------|
| 1 | Valid DS, generates HTML+SVG | Happy path, Phase 1-4 | Good |
| 2 | No DS | Phase 0 readiness gate | Good |
| 3 | Missing simple component | Phase 1 on-the-fly creation | Good |
| 4 | Missing complex component | Phase 1 decomposition path | Good |
| 5 | Context exhaustion | Phase 4.3 continuation prompt | Good |
| 6 | Corrupted tokens.json | Phase 0 JSON validation gate | Good |
| 7 | SVG renders incorrectly | Phase 4.2 iteration with screenshot | Good |

**Coverage grade: Good.** Best covered skill. Both readiness gates tested. Component gap handling covered for both simple and complex cases. Iteration and context exhaustion paths covered.

**Gaps identified:**
- File name validation: user provides a name with path separators or special characters (e.g., `../../evil`) -- the SKILL.md explicitly validates this but no scenario tests it
- Mobile viewport generation (`viewBox="0 0 390 844"`) not explicitly tested
- Ambiguous user description requiring clarification (Phase 1 step 2) not explicitly tested
- Layout selection disagreement (user rejects proposed layout) not tested

### design-review (4 scenarios)

| # | Scenario | Covers | Assessment |
|---|----------|--------|------------|
| 1 | UI files with hardcoded colors | Happy path -- findings present | Good |
| 2 | Files using DS tokens | No findings path | Good |
| 3 | No tokens.json | Skip condition #1 | Good |
| 4 | No UI files changed | Skip condition #2 | Good |

**Coverage grade: Adequate.** All 4 cells of the decision framework are covered. However, the scope is thin for the most complex behavior.

**Gaps identified:**
- Only colors mentioned in scenario 1; hardcoded spacing and typography violations are not explicitly tested as separate scenarios
- Prioritization behavior (>3 findings, cap at 3, "N more findings" message) not tested
- Approximate color matching (e.g., `#3a82f5` vs token `#3b82f6`) not tested
- Non-DS custom properties that duplicate token semantics not tested

### design-retrospective (5 scenarios)

| # | Scenario | Covers | Assessment |
|---|----------|--------|------------|
| 1 | Session with corrections | Phase 1-3 happy path | Good |
| 2 | Lesson repeated 3+ times | Phase 4 promotion | Good |
| 3 | Generates next-session prompt | Phase 5 | Good |
| 4 | First run, no lessons-learned.md | Phase 3 file creation | Good |
| 5 | Empty session | Phase 2 early exit | Good |

**Coverage grade: Adequate.** Core flow and key branches covered.

**Gaps identified:**
- Missing `.design-system/` directory (Phase 1 step 1 -- should refuse) has no scenario
- Duplicate lesson detection (Phase 3 step 2 -- same problem already recorded) not tested
- Already-promoted rule not duplicated in design-principles.md (Phase 4 step 5) not tested
- Promotion threshold boundary: exactly 2 occurrences (should NOT promote) vs exactly 3 (should promote) not tested as boundary pair

---

## Findings

### Finding 1: Missing scenario for design-retrospective without `.design-system/` directory

- **Severity:** major
- **Category:** missing_coverage
- **Location:** Tech-spec Testing Strategy > design-retrospective scenarios
- **Issue:** The SKILL.md Phase 1 step 1 explicitly states: if `.design-system/` is missing, stop and suggest running `/design-system-init`. This is a readiness gate identical in pattern to design-generate's "No DS" scenario (#2), but design-retrospective has no equivalent scenario. Every other skill that depends on `.design-system/` has a "missing DS" scenario except retrospective.
- **Recommendation:** Add scenario 6: "No `.design-system/` directory -- refuses with suggestion to run `/design-system-init`." This mirrors design-generate scenario #2 and ensures the guard clause is tested.

### Finding 2: Contrast ratio failure path untested in design-system-init

- **Severity:** major
- **Category:** missing_coverage
- **Location:** Tech-spec Testing Strategy > design-system-init scenarios
- **Issue:** SKILL.md Phase 4 step 3 specifies contrast ratio check with target >= 4.5:1 and explicit behavior: "If any pair fails, adjust the color and update tokens.json." This is a concrete behavioral branch (detect failure -> auto-fix -> re-verify) with no scenario covering it. The acceptance criteria also list "Contrast ratio >= 4.5:1" as a verification point. A scenario where the interview produces low-contrast colors and the skill must auto-correct is missing.
- **Recommendation:** Add scenario 7: "Generated palette has low contrast (text.primary on background.default < 4.5:1) -- skill auto-adjusts colors and updates tokens.json." This tests the Phase 4 remediation loop, not just the check.

### Finding 3: File name validation (path traversal prevention) untested

- **Severity:** minor
- **Category:** missing_coverage
- **Location:** Tech-spec Testing Strategy > design-generate scenarios
- **Issue:** Both design-system-init (Phase 3.1) and design-generate (Phase 3.4) explicitly validate file names against `/^[a-z0-9-]+$/` to prevent path traversal. Tech-spec Decision 9 specifically calls this out as a security measure. No scenario tests what happens when a user provides a name like `../../etc/passwd` or `my page` (with spaces). The security audit (Task 10) likely flags this too, but from a testing perspective, negative input validation should have scenario coverage.
- **Recommendation:** Add a scenario to design-generate: "User provides page name with special characters or path separators -- skill rejects the name and asks for a valid one." This can also be a cross-cutting scenario applicable to design-system-init component naming.

### Finding 4: design-review finding prioritization and cap behavior untested

- **Severity:** minor
- **Category:** missing_coverage
- **Location:** Tech-spec Testing Strategy > design-review scenarios
- **Issue:** SKILL.md specifies: when >3 findings exist, cap output at 3 with message "N more findings -- run `/design-review` for full report." Prioritization order: colors > typography > spacing. Scenario 1 tests "hardcoded colors" but does not test the behavior when multiple violation types coexist and must be prioritized and capped.
- **Recommendation:** Add scenario 5: "UI files with 5+ violations across colors, typography, and spacing -- review outputs 3 highest-priority findings (colors first) with 'N more findings' message." This tests prioritization logic and the cap mechanism.

### Finding 5: Cross-skill interaction coverage is absent

- **Severity:** minor
- **Category:** missing_coverage
- **Location:** Tech-spec Testing Strategy (overall)
- **Issue:** All 22 scenarios test skills in isolation. No scenario tests the integration chain: (1) do-task triggering design-review as subagent; (2) design-retrospective consuming output from a design-generate session (corrections, feedback); (3) design-generate reading tokens.json produced by design-system-init. While each skill has its own readiness checks, the hand-off between skills (especially do-task -> design-review and the session data flow into retrospective) is untested.
- **Recommendation:** Consider adding 1-2 cross-skill scenarios: (a) "do-task with UI changes and existing DS triggers design-review, receives token recommendations" -- tests the integration hook; (b) "design-retrospective analyzes a design-generate session with 3 color corrections -- identifies color pattern, writes lesson." These can be manual verification scenarios if the skill-tester pipeline cannot orchestrate multi-skill flows.

### Finding 6: Boundary value for lesson promotion threshold untested

- **Severity:** minor
- **Category:** missing_coverage
- **Location:** Tech-spec Testing Strategy > design-retrospective scenario 2
- **Issue:** Scenario 2 tests "lesson repeated 3+ times -- promotes to design-principles.md." The boundary is 3. There is no scenario verifying that 2 occurrences do NOT trigger promotion. Boundary testing requires testing both sides of a threshold.
- **Recommendation:** Scenario 2 should explicitly include the boundary pair: (a) 2 occurrences of same lesson type -- no promotion; (b) 3 occurrences -- promotion triggered. This can be a single scenario with two assertions or two separate sub-scenarios.

### Finding 7: SCSS and hardcoded value extraction sources not covered by any init scenario

- **Severity:** minor
- **Category:** missing_coverage
- **Location:** Tech-spec Testing Strategy > design-system-init scenarios
- **Issue:** SKILL.md Phase 1 lists 4 source types in priority order: (1) Tailwind config, (2) CSS custom properties, (3) SCSS variables, (4) hardcoded values. Scenarios cover sources #1, #2, and "no styles." Sources #3 (SCSS) and #4 (hardcoded repeated values) have no dedicated scenario. While these may be exercised implicitly, the scan behavior for SCSS `$color-*` patterns and repeated hex values in component files is distinct enough to warrant coverage.
- **Recommendation:** Consider adding scenario 7: "Project with SCSS variables (`$color-primary`, `$font-heading`) but no Tailwind or CSS vars -- extracts from SCSS patterns." Source #4 (hardcoded values) can remain implicitly covered since it is the fallback when no structured source exists.

---

## Testing Strategy Fitness Assessment

### Is scenario testing alone sufficient for an L-size feature with 4 markdown skills?

**Yes, with caveats.**

**Why scenario testing is appropriate:**
- Skills are markdown instructions, not executable code -- unit/integration/E2E tests are literally not applicable
- The skill-test-designer / skill-tester pipeline is the standard mechanism for testing markdown skills
- Scenario tests verify behavioral responses to specific inputs, which is the correct abstraction level

**Caveats:**
1. **Manual verification is critical.** The tech-spec correctly includes manual verification on a real web project. For visual/aesthetic skills, human judgment is irreplaceable. Scenario tests verify structural correctness (right phase executed, right error message shown) but cannot verify aesthetic quality.
2. **Cross-skill scenarios are missing.** The pipeline is designed as interconnected skills (init -> generate -> review -> retrospective). Testing only in isolation misses integration failures.
3. **22 scenarios for 4 skills is reasonable density** (5.5 per skill average). design-generate has the most (7), design-review the fewest (4). This distribution roughly matches skill complexity.

### Test Pyramid Assessment

For this feature type, the traditional pyramid does not apply. Instead:

| Layer | Count | Assessment |
|-------|-------|------------|
| Scenario tests | 22 | Primary and only automated layer -- appropriate |
| Manual verification | 4 checkpoints | Defined in tech-spec -- appropriate |
| Cross-skill tests | 0 | Gap -- should have 1-2 |
| Unit/Integration/E2E | 0 | N/A for markdown skills -- correct |

**Pyramid assessment: unbalanced** (missing cross-skill layer, but acceptable given the testing method constraints)

---

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation in test plan |
|------|-----------|--------|------------------------|
| Contrast ratio auto-fix fails silently | Medium | High (accessibility violation) | NOT covered -- Finding 2 |
| Path traversal via file names | Low | High (security) | NOT covered -- Finding 3 |
| design-review misses spacing/typography violations | Medium | Medium | Partially covered (only colors tested) -- Finding 4 |
| Retrospective runs without .design-system/ | Low | Medium | NOT covered -- Finding 1 |
| Cross-skill handoff breaks | Medium | Medium | NOT covered -- Finding 5 |
| Promotion triggers at wrong threshold | Low | Low | Boundary not tested -- Finding 6 |

---

## Recommendations Summary

### Must-add scenarios (major findings):
1. **design-retrospective #6:** Missing `.design-system/` directory -- refuses with suggestion
2. **design-system-init #7:** Low-contrast palette triggers auto-adjustment in Phase 4

### Should-add scenarios (minor findings):
3. **design-generate:** Invalid file name rejected (path traversal prevention)
4. **design-review #5:** Multiple violation types, prioritization + cap at 3
5. **Cross-skill:** do-task -> design-review integration; retrospective consuming generate session data
6. **design-retrospective:** Boundary pair for promotion threshold (2 vs 3 occurrences)
7. **design-system-init:** SCSS variable extraction source

### Total scenario recommendation: 22 existing + 2 must-add + 5 should-add = 29 scenarios

---

## Metrics

- **Skills reviewed:** 4
- **Scenarios reviewed:** 22
- **Coverage gaps identified:** 7
- **Severity breakdown:** 0 critical, 2 major, 5 minor
- **Coverage assessment:** adequate (gaps in edge cases and cross-skill interactions, but core behaviors well covered)
- **Testing strategy fit:** appropriate for feature type, with noted caveats about cross-skill coverage
