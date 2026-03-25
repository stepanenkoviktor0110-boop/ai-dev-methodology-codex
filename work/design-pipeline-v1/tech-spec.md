---
created: 2026-03-26
status: draft
branch: dev
size: M
---

# Tech Spec: Design Pipeline v1 — Learning & Experience

## Solution

Extend the existing 4 design skills (design-system-init, design-generate, design-review, design-retrospective) and code-writing skill with experience accumulation capabilities. The implementation is purely markdown-based — modifying SKILL.md instructions and creating 2 new reference files.

Three capability layers:
1. **Taste accumulation** — design-retrospective writes `.design-system/taste-profile.md` per project; design-generate and design-system-init read it.
2. **Cross-project experience** — design-retrospective appends learnings to global `shared/design-references/designer-experience.md` by project category; design-system-init reads relevant section during interview.
3. **Style recipes** — new `shared/design-references/style-profiles.md` with 6+ style profiles; design-system-init matches by mood+category during interview.

Additionally: two-layer descriptions (human-readable + technical) across all 4 design skills, and conditional DS token integration in code-writing.

## Architecture

### What we're building/modifying

- **design-retrospective SKILL.md** — new Phase 4.5: taste-profile writing + experience writing; two-layer descriptions in lessons; taste-profile in next-session prompt template
- **design-system-init SKILL.md** — read designer-experience by category, match style-profile after mood, read taste-profile in update flow, two-layer descriptions
- **design-generate SKILL.md** — read taste-profile in Phase 0, two-layer descriptions in Phase 4
- **design-review SKILL.md** — optional two-layer explanation for non-obvious token matches
- **code-writing SKILL.md** — conditional DS token reading in Phase 1, step 2 (Read Project Context)
- **style-profiles.md** (NEW) — 6+ style profiles with concrete design recipes
- **designer-experience.md** (NEW) — cross-project experience organized by project category

### How it works

```
Session flow:
                                                          ┌──────────────────────┐
                                                          │ style-profiles.md    │
                                                          │ (shared reference)   │
                                                          └──────────┬───────────┘
                                                                     │ read (match by mood)
┌─────────────────────┐  read (by category)  ┌───────────────────────▼──────────┐
│ designer-experience │◄─────────────────────│ design-system-init               │
│ .md (shared)        │                      │ (reads experience + style-profile│
│                     │◄──── append ──────┐  │  during interview)               │
└─────────────────────┘                   │  └──────────────────────────────────┘
                                          │
┌─────────────────────┐  write/update     │  ┌──────────────────────────────────┐
│ taste-profile.md    │◄──────────────────┤  │ design-generate                  │
│ (.design-system/)   │                   │  │ (reads taste-profile in Phase 0) │
│                     │──── read ─────────┼─►└──────────────────────────────────┘
└─────────────────────┘                   │
         ▲                                │  ┌──────────────────────────────────┐
         │ read (update flow)             │  │ design-retrospective             │
         └────────────────────────────────┼──│ (writes taste-profile +          │
                                          └──│  appends to experience)          │
                                             └──────────────────────────────────┘
```

1. **Write path** (design-retrospective): after analyzing session patterns (Phase 4 Promote Principles), new Phase 4.5 extracts aesthetic preferences → writes `.design-system/taste-profile.md`, determines project category → appends generalizable learnings to `shared/design-references/designer-experience.md`.
2. **Read path — per-project** (design-generate): Phase 0 reads taste-profile.md after tokens.json parsing → passes preferences as context to Phase 2 (layout selection) and Phase 3 (assembly).
3. **Read path — cross-project** (design-system-init): before interview, reads designer-experience.md section matching project category + matches style-profile by mood → uses both for informed proposals.
4. **Two-layer descriptions**: all 4 design skills describe proposals in human-readable language first, then technical specifics. Design-review — optional, only for non-obvious matches.
5. **Code-writing DS integration**: conditional step in Phase 1, step 2 (Read Project Context) — if UI files + tokens.json exists → read token CSS variable names, use in styles.

### Shared resources

None.

## Decisions

### Decision 1: Taste-profile in project-local .design-system/, experience in shared/
**Decision:** taste-profile.md is per-project (`.design-system/`), designer-experience.md is global (`shared/design-references/`).
**Rationale:** Taste is project-specific (color temperature, boldness). Experience is transferable across projects of same category.
**Alternatives considered:** Both global — rejected: taste contamination across projects. Both local — rejected: no cross-project learning.

### Decision 2: Markdown format for taste-profile (not JSON)
**Decision:** Use markdown with structured sections, not JSON.
**Rationale:** Agent works better with free-form text for subjective preferences. Easier to read/debug. Consistent with all other design artifacts.
**Alternatives considered:** JSON config — rejected: overly rigid for aesthetic preferences. YAML — rejected: no advantage over markdown for this use case.

### Decision 3: Append-only for designer-experience.md
**Decision:** Experience entries are appended to the end of category sections, never overwriting existing entries.
**Rationale:** File is global and accumulates experience from multiple projects over time. Overwriting risks losing valuable historical data.
**Alternatives considered:** Full rewrite — rejected: data loss risk. Separate files per category — rejected: too many files for 4-5 categories.

### Decision 4: Style-profile matching by mood+category (not loading all)
**Decision:** Agent reads only the matching style-profile section, not the entire file.
**Rationale:** Token budget constraint (~1000-1500 tokens overhead). Loading all profiles wastes context.
**Alternatives considered:** Load all profiles — rejected: excessive token overhead. Separate files per style — rejected: 6+ files is harder to maintain.

### Decision 5: Two-layer descriptions optional in design-review
**Decision:** Design-review adds explanation only for non-obvious token matches. Other 3 skills use two-layer always.
**Rationale:** Design-review is capped at 3 recommendations in terse "found X → use Y" format. Mandatory two-layer would bloat a lightweight skill.
**Alternatives considered:** Mandatory everywhere — rejected: overhead for design-review. Skip design-review entirely — rejected: non-obvious matches genuinely confuse users.

### Decision 6: Code-writing integration via conditional step (not subagent)
**Decision:** Add a conditional paragraph to Phase 1, step 2 (Read Project Context) of code-writing SKILL.md.
**Rationale:** Lightweight: ~500-1000 tokens overhead only when tokens.json exists AND task is UI. Subagent spawn would be disproportionate for "read token names → use var()".
**Alternatives considered:** Subagent — rejected: too heavy for simple token lookup. Post-write design-review only — rejected: reactive, not proactive.

### Decision 7: Latest-wins for conflicting taste preferences
**Decision:** When a new session contradicts a previous preference (e.g., warm → cold tones), the latest preference wins. Overridden entry is marked "пересмотрено" with date.
**Rationale:** User's taste evolves; recent sessions reflect current intent better than old ones.
**Alternatives considered:** Merge/average — rejected: meaningless for subjective preferences. Ask user — rejected: adds friction to automated retrospective.

## Data Models

N/A — no database or typed interfaces. All data is markdown files.

**taste-profile.md structure:**
```markdown
# Taste Profile

## Цветовые предпочтения
- Температура: тёплые / холодные / нейтральные
- Насыщенность: приглушённые / яркие / смешанные
- Конкретные предпочтения: [from sessions]

## Типографика
- Стиль заголовков: serif / sans-serif / mono
- Предпочтения: [from sessions]

## Стиль и смелость
- Уровень: conservative / balanced / bold / experimental
- Характер: [from sessions]

## Антипаттерны
- [что отклонялось и сколько раз]

## История изменений
- [dated entries, latest wins, overridden marked "пересмотрено"]
```

**designer-experience.md structure:**
```markdown
# Designer Experience

## Landing
### Предпочтения
### Что работало
### Антипаттерны

## Webapp
### Предпочтения
### Что работало
### Антипаттерны

## Admin
...

## Portfolio
...
```

## Dependencies

### New packages
None.

### Using existing (from project)
- `shared/design-references/` — existing reference directory, adding 2 new files
- `skills/design-system-init/SKILL.md` — modifying (171 lines → ~200 lines)
- `skills/design-generate/SKILL.md` — modifying (206 lines → ~220 lines)
- `skills/design-review/SKILL.md` — modifying (111 lines → ~115 lines)
- `skills/design-retrospective/SKILL.md` — modifying (167 lines → ~210 lines)
- `skills/code-writing/SKILL.md` — modifying (141 lines → ~150 lines)

## Testing Strategy

**Feature size:** M

### Unit tests
None — markdown-only feature, no executable code.

### Integration tests
None — no programmatic integration.

### E2E tests
None — no UI or API.

### Scenario testing (primary method)
Agent-driven verification via grep/read checks on modified SKILL.md files and created reference files. Scenarios:
1. taste-profile creation/update path exists in design-retrospective
2. taste-profile reading path exists in design-generate Phase 0
3. experience reading path exists in design-system-init Phase 2
4. style-profiles.md has 6+ profiles with concrete techniques
5. designer-experience.md has 4 category sections with correct subsections
6. two-layer instruction present in all 4 design skills
7. code-writing conditional DS step present in Phase 1, step 2 (Read Project Context)
8. all reference links in SKILL.md files resolve to existing files
9. no regression: existing phases, checkpoints, references preserved

## Agent Verification Plan

**Source:** user-spec "Как проверить" section.

### Verification approach

Per-task Verify-smoke checks cover individual file modifications. Post-implementation QA verifies cross-cutting concerns:

1. **Reference resolution:** for each SKILL.md, extract all `[text](path)` links, verify target files exist via `bash test -e`
2. **Phase/checkpoint preservation:** diff each SKILL.md against git HEAD, confirm no existing phases or checkpoints were deleted
3. **Content completeness:** grep each SKILL.md for expected keywords (taste-profile, designer-experience, style-profiles, two-layer, tokens.json)
4. **Structure validation:** verify designer-experience.md has 4 category sections, style-profiles.md has 6+ profiles, taste-profile template in retrospective has required sections

### Tools required
bash (grep, test -e, diff), read tool.

## Risks

| Risk | Mitigation |
|------|-----------|
| Experience contamination across project categories | Strict category sections in designer-experience.md; agent reads only matching section |
| Style-profiles.md grows too large, token overhead | Agent reads only matched profile section; each profile capped at ~200 words |
| Conflicting preferences in taste-profile | Latest-wins policy; overridden entries marked "пересмотрено" with date |
| Regression in existing skill phases | Verify-smoke grep checks for existing phase headers + checkpoints after each modification |
| Code-writing DS integration adds overhead for non-UI tasks | Dual condition: tokens.json exists AND UI files — silent skip otherwise |

## Acceptance Criteria

Технические критерии приёмки (дополняют пользовательские из user-spec):

- [ ] All 5 modified SKILL.md files pass structural validation: phases numbered correctly, checkpoints present, no orphaned references
- [ ] New reference links in SKILL.md files use correct relative paths (`../../shared/design-references/` from skills)
- [ ] style-profiles.md follows existing reference file format: Russian language, structured sections, concrete values, quick-lookup table
- [ ] designer-experience.md uses append-only section structure with dated entries
- [ ] taste-profile template in design-retrospective includes all required sections: color preferences, typography, style+boldness, anti-patterns, change history
- [ ] design-retrospective Phase 4.5 handles edge cases: missing taste-profile (create), corrupted taste-profile (recreate with warning), missing designer-experience (create), missing category section (create)
- [ ] code-writing conditional step explicitly lists UI file extensions (.tsx, .vue, .html, .css, .scss) and specifies silent skip behavior
- [ ] No existing phases, checkpoints, or reference links removed from any SKILL.md
- [ ] Two-layer description instruction specifies: human-readable first, then technical; design-review marked as optional

## Implementation Tasks

<!-- Tasks are brief scope descriptions. AC, TDD, and detailed steps are created during task-decomposition.

     Verify-smoke: concrete executable checks the agent runs during implementation — no deployment needed.
     Types: command (curl, python -c, docker build), MCP tool (Playwright, Telegram),
     API call (OpenRouter, external services), local server check, agent with test prompt.
     Verify-user: agent asks user to verify something (UI, behavior, experience).
     Both fields optional — omit if task is internal logic fully covered by tests. -->

### Wave 1 (независимые)

#### Task 1: Create style-profiles.md
- **Description:** Create `shared/design-references/style-profiles.md` with 6+ style profiles (luxury, brutalist, editorial, minimal, playful, corporate). Each profile contains concrete design techniques. Needed as reference for design-system-init style matching.
- **Skill:** skill-master
- **Reviewers:** skill-checker
- **Verify-smoke:** `grep -c "^## " shared/design-references/style-profiles.md` → 6+
- **Files to modify:** (new) `shared/design-references/style-profiles.md`
- **Files to read:** `shared/design-references/color-psychology.md`, `shared/design-references/color-principles.md` (format reference)

#### Task 2: Create designer-experience.md
- **Description:** Create `shared/design-references/designer-experience.md` with 4 project category sections (landing, webapp, admin, portfolio). Each section has subsections for preferences, what worked, and anti-patterns. Needed as cross-project knowledge base for design-system-init.
- **Skill:** skill-master
- **Reviewers:** skill-checker
- **Verify-smoke:** `grep -c "^## " shared/design-references/designer-experience.md` → 4+; `grep "Антипаттерны" shared/design-references/designer-experience.md` → found
- **Files to modify:** (new) `shared/design-references/designer-experience.md`
- **Files to read:** `shared/design-references/color-psychology.md` (format reference)

#### Task 3: Update design-review SKILL.md
- **Description:** Add optional two-layer explanation instruction for non-obvious token matches in How to Report section. Minimal change — only adds guidance for when the suggested token isn't an exact match.
- **Skill:** skill-master
- **Reviewers:** skill-checker
- **Verify-smoke:** `grep -i "non-obvious\|неочевидн" skills/design-review/SKILL.md` → found
- **Files to modify:** `skills/design-review/SKILL.md`
- **Files to read:** `skills/design-review/SKILL.md`

#### Task 4: Update code-writing SKILL.md
- **Description:** Add conditional DS token integration step at end of Phase 1, step 2 (Read Project Context). If UI files + tokens.json exist → read tokens, use CSS variables. Silent skip otherwise. Lightweight proactive integration complementing reactive design-review.
- **Skill:** skill-master
- **Reviewers:** skill-checker
- **Verify-smoke:** `grep "tokens.json" skills/code-writing/SKILL.md` → found; `grep -i "silent\|skip" skills/code-writing/SKILL.md` → found
- **Files to modify:** `skills/code-writing/SKILL.md`
- **Files to read:** `skills/code-writing/SKILL.md`

### Wave 2 (зависит от Wave 1)

#### Task 5: Update design-retrospective SKILL.md
- **Description:** Add new Phase 4.5 (taste-profile writing + experience writing to designer-experience.md), two-layer description instruction in Phase 3, taste-profile section in Phase 5 next-session prompt template, and Self-Verification items. This is the primary "write path" — the most critical modification.
- **Skill:** skill-master
- **Reviewers:** skill-checker
- **Verify-smoke:** `grep "taste-profile" skills/design-retrospective/SKILL.md` → found; `grep "designer-experience" skills/design-retrospective/SKILL.md` → found; `grep "Phase 4.5\|Phase 5" skills/design-retrospective/SKILL.md` → both found; `grep -i "corrupted\|повреждён\|recreate\|пересозда" skills/design-retrospective/SKILL.md` → found (edge case handling)
- **Files to modify:** `skills/design-retrospective/SKILL.md`
- **Files to read:** `skills/design-retrospective/SKILL.md`, `shared/design-references/designer-experience.md`

#### Task 6: Update design-system-init SKILL.md
- **Description:** Add experience reading (designer-experience.md by category) before interview, style-profile matching after mood determination, taste-profile reading in update flow, two-layer description instructions, and Final Check items. Depends on Tasks 1-2 for reference files to link to.
- **Skill:** skill-master
- **Reviewers:** skill-checker
- **Verify-smoke:** `grep "designer-experience" skills/design-system-init/SKILL.md` → found; `grep "style-profiles" skills/design-system-init/SKILL.md` → found; `grep "taste-profile" skills/design-system-init/SKILL.md` → found
- **Files to modify:** `skills/design-system-init/SKILL.md`
- **Files to read:** `skills/design-system-init/SKILL.md`, `shared/design-references/style-profiles.md`, `shared/design-references/designer-experience.md`

#### Task 7: Update design-generate SKILL.md
- **Description:** Add taste-profile reading step in Phase 0 (after tokens.json parsing), two-layer description instruction in Phase 4, and Final Check items. Taste-profile format is defined in tech-spec Data Models section.
- **Skill:** skill-master
- **Reviewers:** skill-checker
- **Verify-smoke:** `grep "taste-profile" skills/design-generate/SKILL.md` → found; `grep -c "Phase 0\|Phase 1\|Phase 2\|Phase 3\|Phase 4\|Phase 5" skills/design-generate/SKILL.md` → 6 (all original phases preserved)
- **Files to modify:** `skills/design-generate/SKILL.md`
- **Files to read:** `skills/design-generate/SKILL.md`

### Audit Wave

<!-- Full-feature audit: 3 auditors review all code in parallel. Always present. -->
<!-- Auditors read code and write reports. If issues found — lead spawns a fixer, auditors become reviewers. -->

#### Task 8: Code Audit
- **Description:** Full-feature code quality audit. Read all SKILL.md files and reference files created/modified in this feature. Review holistically for cross-component issues: consistent terminology, correct relative paths, instruction clarity, checkpoint completeness. Write audit report.
- **Skill:** code-reviewing
- **Reviewers:** none

#### Task 9: Security Audit
- **Description:** Full-feature security audit. Read all files created/modified in this feature. Analyze for path traversal risks in relative references, information leakage in experience files, unsafe file operations in SKILL.md instructions. Write audit report.
- **Skill:** security-auditor
- **Reviewers:** none

#### Task 10: Test Audit
- **Description:** Full-feature test quality audit. Review Verify-smoke commands across all tasks, verify Agent Verification Plan coverage, check scenario testing completeness against acceptance criteria. Write audit report.
- **Skill:** test-master
- **Reviewers:** none

### Final Wave

<!-- QA is always present. Deploy and Post-deploy — only if applicable for this feature. -->

#### Task 11: Pre-deploy QA
- **Description:** Acceptance testing: verify all acceptance criteria from user-spec and tech-spec. Run all Verify-smoke checks. Validate reference link resolution. Confirm no regression in existing skill phases.
- **Skill:** pre-deploy-qa
- **Reviewers:** none
