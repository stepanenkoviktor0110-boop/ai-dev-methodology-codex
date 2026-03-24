---
created: 2026-03-25
status: draft
branch: dev
size: L
---

# Tech Spec: design-pipeline

## Solution

Create a Design Pipeline — a framework of 4 MVP skills for visual aesthetics work in web projects. The pipeline operates as a design assistant: initializes design systems through interview, generates HTML+SVG mockups, reviews UI code against design tokens, and learns from feedback through retrospective.

The pipeline migrates and replaces the existing `like-figma` monolithic skill, splitting it into focused skills with enriched design knowledge (color psychology, color principles, grid techniques, font pairing). Integration with the development methodology is minimal (~5 lines in do-task) to keep token cost low.

All design artifacts are stored as code in the project's `.design-system/` directory — no Figma API, no external tools.

## Architecture

### What we're building/modifying

- **`skills/design-system-init/`** — procedural skill: project scan → interview (mood, palette, typography, spacing, components) → generates `.design-system/tokens.json` + component HTML files. Migrates like-figma Phase 1-2 with color psychology and font pairing enrichment.
- **`skills/design-generate/`** — procedural skill: parses user request → selects layout → assembles components → generates HTML + SVG. Migrates like-figma Phase 3 with advanced grid techniques.
- **`skills/design-review/`** — informational skill: reads `tokens.json` + changed UI files → gives 2-3 concrete token-based recommendations. Lightweight (~500-1000 tokens overhead).
- **`skills/design-retrospective/`** — procedural skill: collects design session feedback → identifies aesthetic patterns → writes lessons → promotes repeated lessons to principles → generates next-session prompt.
- **`shared/design-references/`** — shared reference directory for files used by multiple design skills (design-tokens.md, color-psychology.md, color-principles.md).
- **`skills/do-task/SKILL.md`** — minimal modification (~5 lines): after code commit, check for `.design-system/tokens.json` + UI files → call design-review subagent.
- **`skills/design-system-init/references/font-pairing.md`** — new reference: font pairing rules, classic combinations, weight contrasts, typographic scale.
- **`shared/design-references/library-catalog.md`** — new reference: catalog of free icon/element libraries with descriptions and use-case guidance. Links only, no asset storage.

### How it works

**Design System Init flow:**
1. Scan project for existing colors/fonts/spacing (Tailwind config > CSS vars > SCSS > hardcoded)
2. Interview: project context & mood (using color-psychology.md emotional map) → palette proposal → typography pairs (using font-pairing.md) → spacing (golden ratio/Fibonacci options from grid-techniques) → components
3. Build: generate `tokens.json` (per design-tokens.md schema) + component HTML files (per component-patterns.md) → each opens standalone in browser
4. Verify: validate JSON, check CSS custom properties usage, contrast ratio ≥ 4.5:1

**Design Generate flow:**
1. Read `.design-system/tokens.json` + component files
2. Parse user description → select layout from 15 patterns (5 basic from generation-guide + 10 advanced from grid-techniques)
3. Assemble page from components, apply tokens
4. Generate HTML (interactive, uses preview-template.html) + SVG (static, 1440x900 or 390x844)
5. Present → user sends screenshot with feedback → iterate. If context exhausted → generate continuation prompt

**Design Review flow (from do-task):**
1. Check: `.design-system/tokens.json` exists AND task changed UI files (.tsx/.vue/.html/.css/.scss)
2. Read only `tokens.json` (not full DS — token economy)
3. Scan changed files for hardcoded values, non-DS colors/spacing/fonts
4. Output 2-3 concrete recommendations: "color #3B82F6 → use --color-primary-500", "gap 8px → DS minimum --space-4 (16px)"
5. Apply fixes → commit

**Design Retrospective flow:**
1. Collect evidence: user feedback during session, corrections made, files changed
2. Identify patterns: repeated color corrections, rejected font pairs, spacing adjustments, layout reworks
3. Write lessons to `.design-system/lessons-learned.md` (Problem/Cause/Solution/Rule format)
4. Count occurrences — if lesson appears 3+ times → promote to `.design-system/design-principles.md`
5. Generate next-session prompt (self-contained, no prior context needed)

### Shared resources

None.

## Decisions

### Decision 1: Shared design-references directory
**Decision:** Create `shared/design-references/` for references used by multiple design skills.
**Rationale:** design-tokens.md is needed by init, generate, and review. color-principles.md by init and review. Duplicating violates DRY; cross-skill path references are fragile. `shared/` directory pattern already exists (`shared/work-templates/`).
**Alternatives considered:** (1) Duplicate references in each skill — simple but maintenance burden. (2) Cross-reference via `$AGENTS_HOME/skills/{other-skill}/references/` — fragile coupling.

### Decision 2: No interview template file
**Decision:** Interview logic embedded directly in design-system-init SKILL.md, not in `shared/interview-templates/design.yml`.
**Rationale:** Interview templates (`feature.yml`, `skill.yml`) are used by `user-spec-planning` which has a generic interview loop. Design-system-init has a domain-specific interview flow with topic-by-topic progression (scan → mood → palette → typography → spacing → components). This doesn't fit the generic template structure.
**Alternatives considered:** Creating `design.yml` template — adds indirection without benefit since no other skill would reuse it.

### Decision 3: No shim skills for MVP
**Decision:** Each design-* skill is its own canonical skill with full procedural content. No shims.
**Rationale:** Shims exist when a user-facing command (`/new-user-spec`) redirects to a deeper skill (`user-spec-planning`). MVP design skills are invoked directly (`/design-system-init`, `/design-generate`, etc.) — there's no deeper layer to redirect to.
**Alternatives considered:** Creating `/design` umbrella command with shims — premature abstraction for 4 independent skills.

### Decision 4: design-review is informational, not procedural
**Decision:** design-review uses informational skill type (sections by logic, decision frameworks) rather than procedural (strict phases).
**Rationale:** Review has no strict phase sequence — it reads tokens, scans files, applies criteria, outputs recommendations. The order is flexible. Contrast with design-system-init which must follow scan → interview → build → verify strictly.
**Alternatives considered:** Procedural with 3 phases — overcomplicates a straightforward check.

### Decision 5: Minimal do-task integration
**Decision:** Add ~5 lines to do-task SKILL.md between Step 2.3 (git commit) and Step 2.4 (reviewers) for design-review hook.
**Rationale:** User-spec explicitly requires token economy (~500-1000 tokens overhead). Reading only `tokens.json` + scanning only changed files keeps cost at ~1-2% of task budget. Full DS analysis, mockup generation, and component creation are post-MVP.
**Alternatives considered:** (1) Hook at write-code level — too invasive, harder to control. (2) Separate post-review step — adds latency without benefit.

### Decision 6: Reference deployment — copy, not symlink
**Decision:** Copy reference files from like-figma and work/design-pipeline/references/ to their target locations. Delete originals after migration verified.
**Rationale:** Skills must be self-contained. Symlinks are fragile on Windows. File copies are simple, verified by skill-checker.
**Alternatives considered:** Git symlinks or relative paths — platform-dependent, adds complexity.

### Decision 7: Reference naming: user-spec names → tech-spec files
**Decision:** User-spec AC mentions `color-theory.md`, `composition.md`, `font-pairing.md`. Tech-spec maps these to: `color-theory.md` → `color-psychology.md` + `color-principles.md` (split into psychology and technique); `composition.md` → `grid-techniques.md` (composition = grid/layout techniques); `font-pairing.md` → `font-pairing.md` (1:1).
**Rationale:** During user-spec interview, names were placeholders for content areas. Actual content was researched and written with more precise names reflecting the material. Color theory splits naturally into emotional psychology and combination principles. Composition is specifically about grid/layout techniques.
**Alternatives considered:** Renaming files to match user-spec literally — would lose semantic precision.

### Decision 8: CSP protection for generated HTML
**Decision:** preview-template.html must include `<meta http-equiv="Content-Security-Policy" content="script-src 'none'; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com">` to prevent XSS in generated HTML files opened in browser.
**Rationale:** Generated HTML uses placeholder substitution (`{{PAGE_CONTENT}}`) — unsanitized content could execute scripts. CSP with `script-src 'none'` blocks all JS execution, making XSS impossible even if content is injected.
**Alternatives considered:** HTML-entity encoding of all content — fragile, relies on every code path encoding correctly.

### Decision 9: File naming validation for .design-system/ paths
**Decision:** Component and page names derived from user input must be validated: alphanumeric + hyphens only, no path separators. Pattern: `/^[a-z0-9-]+$/`.
**Rationale:** Prevents path traversal (`../../etc/passwd`) when creating files in `.design-system/components/` and `.design-system/pages/`.
**Alternatives considered:** Sanitizing paths — error-prone, validation is simpler and safer.

### Decision 10: design-principles.md lives in project, not in skill
**Decision:** `.design-system/design-principles.md` is a project-level file, populated by design-retrospective. Not a skill reference.
**Rationale:** Design principles are project-specific taste accumulated from user feedback. Different projects have different aesthetic profiles. Skill references contain universal design knowledge.
**Alternatives considered:** Skill-level reference — would mix universal knowledge with project-specific taste.

## Data Models

### tokens.json schema (from design-tokens.md)

```json
{
  "colors": {
    "primary": { "50": "#hex", "100": "#hex", ..., "900": "#hex" },
    "secondary": { "50-900": "..." },
    "neutral": { "0-1000": "..." },
    "semantic": { "success": "#hex", "warning": "#hex", "error": "#hex", "info": "#hex" },
    "background": { "default": "#hex", "surface": "#hex", "elevated": "#hex" },
    "text": { "primary": "#hex", "secondary": "#hex", "disabled": "#hex", "inverse": "#hex" }
  },
  "typography": {
    "families": { "heading": "font", "body": "font", "mono": "font" },
    "sizes": { "xs": "rem", ..., "4xl": "rem" },
    "weights": { "regular": 400, "medium": 500, "semibold": 600, "bold": 700 },
    "lineHeights": { "tight": 1.25, "normal": 1.5, "relaxed": 1.75 }
  },
  "spacing": { "1": "4px", "2": "8px", ..., "16": "64px" },
  "radii": { "sm": "4px", "md": "8px", "lg": "16px", "full": "9999px" },
  "shadows": { "sm": "...", "md": "...", "lg": "..." },
  "breakpoints": { "sm": "640px", "md": "768px", "lg": "1024px", "xl": "1280px" }
}
```

### .design-system/ directory structure (in user's project)

```
.design-system/
├── tokens.json                    # design tokens
├── design-principles.md           # accumulated taste (from retrospective)
├── lessons-learned.md             # raw lessons (from retrospective)
├── components/
│   ├── button.html                # standalone component showcase
│   ├── card.html
│   ├── input.html
│   └── ...
└── pages/
    ├── {name}.html                # generated page (interactive)
    └── {name}.svg                 # generated page (static)
```

## Dependencies

### New packages

None — skills are markdown instructions, no runtime dependencies.

### Using existing (from project)

- `skills/like-figma/references/design-tokens.md` — token schema, copied to shared/design-references/
- `skills/like-figma/references/component-patterns.md` — component catalog, copied to design-generate/references/
- `skills/like-figma/references/generation-guide.md` — page assembly guide, copied and enriched to design-generate/references/
- `skills/like-figma/assets/preview-template.html` — HTML template, copied to design-generate/assets/
- `work/design-pipeline/references/color-psychology.md` — emotional color map, moved to shared/design-references/
- `work/design-pipeline/references/color-principles.md` — 10 color principles, moved to shared/design-references/
- `work/design-pipeline/references/grid-techniques.md` — 10 grid techniques, moved to design-generate/references/

## Testing Strategy

**Feature size:** L

### Unit tests

Not applicable — skills are markdown instructions, not executable code.

### Integration tests

Not applicable — no runtime code to integrate.

### E2E tests

Not applicable — skills don't have a testable runtime.

### Scenario tests (primary method)

Via skill-test-designer → skill-tester pipeline:

- **design-system-init:** (1) Fresh project with no styles — creates tokens.json from scratch. (2) Project with Tailwind config — extracts existing values. (3) `.design-system/` already exists — asks update/recreate/cancel. (4) Non-web project — refuses with explanation. (5) Conflicting style sources (Tailwind config vs CSS vars) — shows conflict, asks priority. (6) Project with existing CSS vars but no Tailwind — extracts from vars.
- **design-generate:** (1) Valid DS exists — generates HTML+SVG from description. (2) No DS — refuses, suggests `/design-system-init`. (3) Missing simple component (badge) — creates on-the-fly. (4) Missing complex component (carousel) — decomposes, separate session. (5) Context exhaustion — stops, generates continuation prompt. (6) Corrupted/invalid tokens.json — error with suggestion to recreate via `/design-system-init`. (7) SVG renders incorrectly — asks for screenshot, simplifies problematic elements.
- **design-review:** (1) UI files with hardcoded colors — gives concrete token recommendations. (2) Files already using DS tokens — reports "styles match DS". (3) No `.design-system/tokens.json` — skips silently. (4) No UI files changed — skips silently.
- **design-retrospective:** (1) Session with corrections — creates lessons in lessons-learned.md. (2) Lesson repeated 3+ times — promotes to design-principles.md. (3) Generates next-session prompt. (4) First run (no existing lessons-learned.md) — creates file from scratch. (5) Empty session (no corrections) — reports "no patterns detected", skips lesson creation.

### Manual verification

On a real web project (e.g., hookah webapp):
- User opens HTML components in browser — visually correct and aesthetic
- User opens generated page — layout matches description, colors/fonts consistent
- User sends screenshot with feedback — agent iterates appropriately
- User checks retrospective lessons — meaningful, not boilerplate

## Agent Verification Plan

**Source:** user-spec "Как проверить" section.

### Verification approach

1. After design-system-init: validate `tokens.json` with `node -e "JSON.parse(require('fs').readFileSync('.design-system/tokens.json'))"` — must parse without errors, contain colors/typography/spacing sections.
2. After design-system-init: grep component HTML files for hardcoded hex/px values — must use CSS custom properties (`--color-*`, `--space-*`, `--font-*`).
3. After design-system-init: run contrast ratio check script on text/background color pairs — all must be ≥ 4.5:1.
4. After design-review: verify recommendations contain specific token names (not generic "make it prettier").
5. After design-generate: validate generated HTML references tokens from DS.

### Tools required

bash (node, grep, contrast check script). No MCP tools needed — all verification is local.

## Risks

| Risk | Mitigation |
|------|-----------|
| Generated HTML/SVG looks bad without browser rendering feedback | User sends screenshots with comments, agent iterates. design-retrospective accumulates taste profile. |
| Token budget exceeded on large projects | MVP integration reads only tokens.json (~500-1000 tokens). Large pages decomposed into sessions. |
| Subjective "beauty" disagreements | User provides references, retrospective accumulates project-specific taste, lessons promote to principles. |
| font-pairing.md quality — new reference, no existing source | Create based on established typography knowledge (Google Fonts combinations, contrast/concordance principles). Validate with skill-checker. |
| like-figma deletion breaks existing users | Delete only after all 4 skills verified. like-figma stays until Final Wave QA passes. |
| Windows path issues with shared references | Use forward slashes in all skill references. Tested by smoke check on creation. |

## Acceptance Criteria

Technical acceptance criteria (complement user-spec criteria):

- [ ] 4 skill directories created: `skills/design-system-init/`, `skills/design-generate/`, `skills/design-review/`, `skills/design-retrospective/`
- [ ] Each SKILL.md passes skill-checker validation (frontmatter, <500 lines, Pattern A/B links, procedural phases/checkpoints where applicable)
- [ ] `shared/design-references/` contains design-tokens.md, color-psychology.md, color-principles.md
- [ ] `design-system-init/references/font-pairing.md` created with concrete font pair recommendations
- [ ] `design-generate/references/` contains component-patterns.md, generation-guide.md (enriched), grid-techniques.md
- [ ] `design-generate/assets/preview-template.html` copied from like-figma with CSP meta tag added (`script-src 'none'`)
- [ ] do-task SKILL.md updated with design-review integration (~5 lines between Step 2.3 and 2.4)
- [ ] All reference links in SKILL.md files resolve to existing files
- [ ] like-figma skill directory deleted after migration verified
- [ ] No regression in existing do-task flow (design-review is additive, skips silently when no DS)
- [ ] Component/page file names validated: alphanumeric + hyphens only (no path traversal)
- [ ] preview-template.html includes CSP `script-src 'none'` to prevent XSS
- [ ] `shared/design-references/library-catalog.md` created with free icon/element library links

## Implementation Tasks

<!-- Tasks are brief scope descriptions. AC, TDD, and detailed steps are created during task-decomposition.

     Verify-smoke: concrete executable checks the agent runs during implementation — no deployment needed.
     Types: command (curl, python -c, docker build), MCP tool (Playwright, Telegram),
     API call (OpenRouter, external services), local server check, agent with test prompt.
     Verify-user: agent asks user to verify something (UI, behavior, experience).
     Both fields optional — omit if task is internal logic fully covered by tests. -->

### Wave 1 (independent — shared references + font-pairing)

#### Task 1: Create shared design-references directory
- **Description:** Create `shared/design-references/` and populate with design-tokens.md (from like-figma), color-psychology.md, color-principles.md (from work/design-pipeline/references/), and library-catalog.md (new — catalog of free icon/element libraries). These shared references are used by multiple design skills.
- **Skill:** skill-master
- **Reviewers:** skill-checker
- **Verify-smoke:** check all 4 files exist and are non-empty: `ls -la shared/design-references/`
- **Files to modify:** `shared/design-references/design-tokens.md` (new), `shared/design-references/color-psychology.md` (new), `shared/design-references/color-principles.md` (new), `shared/design-references/library-catalog.md` (new)
- **Files to read:** `skills/like-figma/references/design-tokens.md`, `work/design-pipeline/references/color-psychology.md`, `work/design-pipeline/references/color-principles.md`

#### Task 2: Create font-pairing.md reference
- **Description:** Write `font-pairing.md` — rules for heading+body font combinations using Google Fonts. Covers contrast vs concordance pairing, weight hierarchy, typographic scale ratios, classic combinations with rationale. Required by design-system-init for typography interview.
- **Skill:** skill-master
- **Reviewers:** skill-checker
- **Files to modify:** `skills/design-system-init/references/font-pairing.md` (new)
- **Files to read:** `skills/like-figma/references/design-tokens.md` (typography section for schema alignment)

### Wave 2 (depends on Wave 1 — core skills creation)

#### Task 3: Create design-system-init skill
- **Description:** Procedural skill for design system creation through project scan and interview. Migrates like-figma Phase 1-2, enriched with color psychology (palette by mood), font pairing guidance, and golden ratio/Fibonacci spacing options. Outputs `.design-system/tokens.json` + component HTML files.
- **Skill:** skill-master
- **Reviewers:** skill-checker
- **Verify-smoke:** spawn agent with prompt "create design system for a calm spa website" → verify it follows skill phases
- **Files to modify:** `skills/design-system-init/SKILL.md` (new)
- **Files to read:** `skills/like-figma/SKILL.md` (Phase 1-2), `shared/design-references/design-tokens.md`, `shared/design-references/color-psychology.md`, `shared/design-references/color-principles.md`, `skills/design-system-init/references/font-pairing.md`, `skills/skill-master/SKILL.md`

#### Task 4: Create design-generate skill
- **Description:** Procedural skill for generating HTML+SVG mockups from text descriptions. Migrates like-figma Phase 3 with enriched layout selection (15 patterns: 5 basic + 10 advanced grids). Assembles pages from DS components, uses preview template, supports iteration via user screenshots.
- **Skill:** skill-master
- **Reviewers:** skill-checker
- **Verify-smoke:** spawn agent with prompt "generate a login page" (with mock DS) → verify it follows skill phases
- **Files to modify:** `skills/design-generate/SKILL.md` (new), `skills/design-generate/references/component-patterns.md` (new, from like-figma), `skills/design-generate/references/generation-guide.md` (new, enriched from like-figma), `skills/design-generate/references/grid-techniques.md` (new, from work/), `skills/design-generate/assets/preview-template.html` (new, from like-figma)
- **Files to read:** `skills/like-figma/SKILL.md` (Phase 3), `skills/like-figma/references/component-patterns.md`, `skills/like-figma/references/generation-guide.md`, `skills/like-figma/assets/preview-template.html`, `work/design-pipeline/references/grid-techniques.md`, `shared/design-references/design-tokens.md`, `skills/skill-master/SKILL.md`

#### Task 5: Create design-review skill
- **Description:** Informational skill for lightweight UI code review against design tokens. Reads only tokens.json and changed files, gives 2-3 concrete token-based recommendations. Designed for minimal token overhead (~500-1000 tokens) when called from do-task.
- **Skill:** skill-master
- **Reviewers:** skill-checker
- **Files to modify:** `skills/design-review/SKILL.md` (new)
- **Files to read:** `shared/design-references/design-tokens.md`, `shared/design-references/color-principles.md`, `skills/skill-master/SKILL.md`

#### Task 6: Create design-retrospective skill
- **Description:** Procedural skill for design session retrospective. Collects user feedback and corrections, identifies aesthetic patterns, writes lessons to `.design-system/lessons-learned.md`, promotes lessons repeated 3+ times to `design-principles.md`, generates self-contained next-session prompt.
- **Skill:** skill-master
- **Reviewers:** skill-checker
- **Files to modify:** `skills/design-retrospective/SKILL.md` (new)
- **Files to read:** `skills/retrospective/SKILL.md` (pattern to mirror), `skills/skill-master/SKILL.md`

### Wave 3 (depends on Wave 2 — integration + cleanup)

#### Task 7: Integrate design-review into do-task
- **Description:** Add design-review hook to do-task SKILL.md between Step 2.3 (git commit) and Step 2.4 (reviewers). Check `.design-system/tokens.json` existence + UI file changes → call design-review subagent. ~5 lines addition, no existing flow disruption. Guard: no full pipeline, no mockups, no interview, no new components.
- **Skill:** skill-master
- **Reviewers:** skill-checker
- **Verify-smoke:** read updated do-task SKILL.md → verify design-review hook present between Step 2.3 and 2.4
- **Files to modify:** `skills/do-task/SKILL.md`
- **Files to read:** `skills/design-review/SKILL.md`

#### Task 8: Delete like-figma skill
- **Description:** Remove `skills/like-figma/` directory after verifying all content has been migrated to new skills. This is the final cleanup — all references, assets, and logic are now in design-system-init, design-generate, and shared/design-references.
- **Skill:** code-writing
- **Reviewers:** code-reviewer, security-auditor, test-reviewer
- **Verify-smoke:** verify like-figma directory no longer exists; verify all 4 new skills + shared references exist
- **Files to modify:** `skills/like-figma/` (delete entire directory)
- **Files to read:** `skills/design-system-init/SKILL.md`, `skills/design-generate/SKILL.md`, `skills/design-review/SKILL.md`, `skills/design-retrospective/SKILL.md`, `shared/design-references/`

### Audit Wave

#### Task 9: Code Audit
- **Description:** Full-feature code quality audit. Read all SKILL.md files and references created/modified in this feature. Review holistically for cross-skill consistency: reference links validity, naming conventions, tone consistency, structural patterns compliance with skill-master. Write audit report.
- **Skill:** code-reviewing
- **Reviewers:** none

#### Task 10: Security Audit
- **Description:** Full-feature security audit. Read all SKILL.md files for potential prompt injection vectors, unsafe command patterns in verify-smoke instructions, path traversal risks in file operations. Write audit report.
- **Skill:** security-auditor
- **Reviewers:** none

#### Task 11: Test Audit
- **Description:** Full-feature test quality audit. Review scenario test coverage across all 4 skills: edge cases, error paths, boundary conditions. Verify testing strategy adequacy for L-size feature. Write audit report.
- **Skill:** test-master
- **Reviewers:** none

### Final Wave

#### Task 12: Pre-deploy QA
- **Description:** Acceptance testing: verify all acceptance criteria from user-spec and tech-spec. Run skill-checker on all 4 skills. Validate all reference file links. Verify do-task integration doesn't break existing flow.
- **Skill:** pre-deploy-qa
- **Reviewers:** none
