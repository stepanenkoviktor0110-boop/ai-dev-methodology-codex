# Code Research: design-pipeline-v1

## 1. Entry Points — Design Skills Structure

### design-system-init (`skills/design-system-init/SKILL.md`)

4-phase skill: Project Readiness -> Project Scan -> Interview -> Build + Verify.

**Phases:**
- Phase 0: Project Readiness — checks web project, handles existing `.design-system/`
- Phase 1: Project Scan — scans Tailwind config, CSS custom properties, SCSS variables, hardcoded values
- Phase 2: Interview — propose-first approach, topics: mood, palette, typography, spacing, radii/shadows/breakpoints, components
- Phase 3: Build — generates `tokens.json`, CSS custom properties, component HTMLs, README
- Phase 4: Verify — JSON validation, CSS custom properties check, contrast ratio check, file name check

**Checkpoints:** after each phase. Final check is a checklist of 7 items.

**References used:**
- `../../shared/design-references/color-psychology.md` — emotional color map (Phase 2.1)
- `../../shared/design-references/color-principles.md` — palette building (Phase 2.2)
- `references/font-pairing.md` — font pair selection (Phase 2.3)
- `../../shared/design-references/design-tokens.md` — tokens.json schema (Phase 3.1)

**Where to add experience reading:** Phase 2, before interview begins (after Phase 1 scan, before Phase 2.1 mood question). The skill currently reads project scan results and proposes values. Adding `shared/design-references/designer-experience.md` reading here would inform proposals with cross-project experience. Specifically, between lines 52-56 (after "Propose-first approach" declaration and before "2.1 Project Context and Mood").

### design-generate (`skills/design-generate/SKILL.md`)

6-phase skill: Readiness Check -> Parse Request -> Select Layout -> Assemble & Generate -> Present & Iterate -> Before/After Collage.

**Phases:**
- Phase 0: reads and validates `tokens.json`
- Phase 1: parse user description — page type, components, layout, content
- Phase 2: select from 15 layout patterns (5 basic + 10 advanced)
- Phase 3: assemble page, generate HTML (via `assets/preview-template.html`), generate SVG, save files
- Phase 4: present result, iterate on feedback, handle context exhaustion
- Phase 5: before/after collage on approval + cleanup

**References used:**
- `../../shared/design-references/design-tokens.md` — token parsing (Phase 0)
- `references/generation-guide.md` — layout selection table, page assembly process
- `references/component-patterns.md` — BEM naming, variant system, slot pattern
- `references/grid-techniques.md` — advanced grid CSS techniques

**Where to add taste-profile reading:** Phase 0 (after tokens.json validation, before Phase 1). The skill should read `.design-system/taste-profile.md` (if exists) to understand project aesthetic preferences before making layout and style decisions. This would influence Phase 2 (layout selection — advanced vs basic) and Phase 3 (component assembly — preferred spacing, color usage patterns).

### design-review (`skills/design-review/SKILL.md`)

Lightweight, single-phase skill. No phases structure — activation conditions, scan, report.

**Activation:** both `.design-system/tokens.json` exists AND UI files changed.
**Reads:** only `tokens.json` (~500-1000 tokens overhead).
**Checks:** hardcoded colors, spacing, typography, non-DS custom properties in changed UI files.
**Reports:** max 3 recommendations in format `found {what} in {file}:{line} -> use {token CSS variable}`.

**References used:**
- `../../shared/design-references/design-tokens.md` — naming patterns
- `../../shared/design-references/color-principles.md` — temperature awareness for color matching

**Scope guard:** explicitly lists what the skill does NOT do (no mockups, no interviews, no file modifications, no retrospective).

### design-retrospective (`skills/design-retrospective/SKILL.md`)

5-phase skill: Collect Evidence -> Identify Patterns -> Write Lessons -> Promote Principles -> Generate Next-Session Prompt.

**Phases:**
- Phase 1: check `.design-system/` exists, collect user feedback from session, list changed files
- Phase 2: categorize corrections (color, typography, spacing, layout, component), look for repetitions within session and across sessions
- Phase 3: write to `.design-system/lessons-learned.md` — format: date, pattern title, Problem/Cause/Solution/Rule
- Phase 4: promote rules to `.design-system/design-principles.md` if appeared 3+ times across all entries
- Phase 5: generate self-contained next-session prompt

**Output files:**
- `.design-system/lessons-learned.md` — individual lessons
- `.design-system/design-principles.md` — promoted principles (3+ occurrences)
- Next-session prompt (shown to user, not saved to file)

**Where to add taste-profile and experience writing:** After Phase 4 (promote principles), add a new phase that:
1. Writes aesthetic preferences (color temperature, spacing density, typography style, layout preference) to `.design-system/taste-profile.md`
2. Writes cross-project learnings to `shared/design-references/designer-experience.md`

The current Phase 4 already analyzes patterns and promotes principles. The taste-profile writing is a natural extension — same pattern analysis, different output format (accumulated preferences vs discrete rules).

## 2. Data Layer

No database. All data is file-based:

### tokens.json schema (`shared/design-references/design-tokens.md`)
Top-level keys: `colors`, `typography`, `spacing`, `radii`, `shadows`, `breakpoints`.
- `colors`: primary/secondary (shade 50-900), neutral (0-1000), semantic (success/warning/error/info), background (default/surface/elevated), text (primary/secondary/disabled/inverse)
- `typography`: families (heading/body/mono), sizes (xs-4xl), weights (regular/medium/semibold/bold), lineHeights (tight/normal/relaxed)
- `spacing`: numeric keys (1-16) mapping to px values
- `radii`: sm/md/lg/full
- `shadows`: sm/md/lg
- `breakpoints`: sm/md/lg/xl

CSS variable naming: `--{category}-{path}` with hyphens (e.g., `--color-primary-500`, `--font-heading`).

### New files to create (from feature description)
- `.design-system/taste-profile.md` — per-project accumulated preferences
- `shared/design-references/designer-experience.md` — cross-project categorized experience
- `shared/design-references/style-profiles.md` — style reference (luxury, brutalist, editorial etc.)

## 3. Similar Features

### Lessons-learned -> design-principles promotion pattern (design-retrospective)
The retrospective skill already implements a "collect -> categorize -> promote" pipeline:
- Lessons accumulate in `lessons-learned.md` (per session)
- Rules appearing 3+ times promote to `design-principles.md`
- Deduplication check before writing
- Format: structured markdown with date, fields

This pattern can be reused for taste-profile accumulation: session feedback -> taste preferences (per-project), and for designer-experience: promoted cross-project patterns.

### design-review hook in do-task (line 49)
Pattern for lightweight integration:
```
if `.design-system/tokens.json` exists AND task changed UI files → spawn design-review subagent
if either condition false → skip silently
```
This is the exact pattern to replicate for write-code DS token reading. Conditions-based, silent skip, minimal overhead.

### Reference file format (shared/design-references/)
All existing reference files follow a consistent format:
- Russian language for descriptions and analysis
- Markdown with `##` section headers, `###` subsections
- Tables for quick lookup (emotion -> combination, principle -> effect)
- Code blocks for CSS/JSON examples
- 1-2 sentence summary per concept, then nuance/detail
- Practical "how to use" sections with concrete values

Files:
- `color-psychology.md` — 324 lines. Color profiles (Part I), emotional combinations (Part II), summary table
- `color-principles.md` — 187 lines. 10 numbered principles with CSS examples
- `design-tokens.md` — 114 lines. JSON schema, naming conventions, CSS generation rules
- `library-catalog.md` — 104 lines. Icon/UI/illustration library catalog with URL/License/Style/Use-cases
- `font-pairing.md` (skill-local) — 196 lines. Principles, weight hierarchy, scale ratios, 10 classic combinations

## 4. Integration Points

### do-task -> design-review hook
File: `skills/do-task/SKILL.md`, Step 2, item 4.
```
if `.design-system/tokens.json` exists AND task changed UI files (.tsx, .vue, .html, .css, .scss) → spawn design-review subagent
```
This is the existing pattern for inserting design pipeline hooks into code execution flow.

### write-code -> code-writing delegation
File: `skills/write-code/SKILL.md` — thin wrapper that delegates to `$AGENTS_HOME/skills/code-writing/SKILL.md`.

### code-writing skill structure
File: `skills/code-writing/SKILL.md` — 3-phase process: Preparation -> Implementation (TDD) -> Post-work.
- Phase 1.2 "Read Project Context" is where DS token reading would be added for UI tasks
- Phase 2.2 "Write Code" is where token-aware coding guidance would apply
- Phase 3.4 "Run Reviews" is where design-review already gets triggered via do-task

**Where to add DS integration in code-writing:** Phase 1.2 "Read Project Context" section (lines 22-34). Currently reads project-knowledge files. Add a conditional step: "If task involves UI files AND `.design-system/tokens.json` exists → read tokens.json, use token CSS variables instead of hardcoded values." This mirrors the design-review activation pattern but is proactive (during coding) rather than reactive (during review).

### design-system-init -> shared references
The init skill references 3 shared files and 1 local reference:
- `../../shared/design-references/color-psychology.md`
- `../../shared/design-references/color-principles.md`
- `../../shared/design-references/design-tokens.md`
- `references/font-pairing.md`

New references (`designer-experience.md`, `style-profiles.md`) would follow the same relative path pattern from skills.

### design-generate -> design-system artifacts
Reads: `.design-system/tokens.json`, `.design-system/components/`
Writes: `.design-system/pages/`, `.design-system/collages/`

### design-retrospective -> design-system artifacts
Reads: `.design-system/` structure, `tokens.json`, `components/`, `pages/`
Writes: `.design-system/lessons-learned.md`, `.design-system/design-principles.md`

## 5. Existing Tests

No test files found for design skills. The skills are declarative markdown specifications (SKILL.md), not executable code. Testing is defined within skill-test-designer and skill-tester skills, but no existing test artifacts were found for design pipeline skills.

Testing framework: the project has `skills/skill-test-designer/` and `skills/skill-tester/` for testing skills, but this is a manual/agent-driven process, not automated unit tests.

## 6. Shared Utilities

### shared/design-references/ (4 files)
- `color-psychology.md` — emotional color mapping. Used by design-system-init Phase 2.1.
- `color-principles.md` — 10 principles for professional color work. Used by design-system-init Phase 2.2 and design-review.
- `design-tokens.md` — tokens.json schema and CSS variable naming. Used by design-system-init Phase 3, design-generate Phase 0, design-review.
- `library-catalog.md` — free icon/UI/illustration library catalog. Informational reference.

### Skill-local references
- `skills/design-system-init/references/font-pairing.md` — 10 classic font combinations with tokens.json mapping
- `skills/design-generate/references/generation-guide.md` — 15 layout patterns, selection criteria, page assembly
- `skills/design-generate/references/component-patterns.md` — BEM naming, component HTML structure
- `skills/design-generate/references/grid-techniques.md` — advanced CSS grid techniques

### shared/work-templates/
- `decisions.md.template` — used by do-task for execution reports
- `session-prompt.md.template` — used by do-task for next-session prompt generation

## 7. Potential Problems

### No taste-profile deduplication strategy
The retrospective already has deduplication for lessons-learned (check existing entries before writing). The taste-profile needs a similar strategy — how to handle conflicting preferences across sessions (e.g., session 1 says "prefer warm tones", session 3 says "prefer cool tones"). Need a "latest wins" or "merge with context" approach.

### designer-experience.md is cross-project
This file lives in `shared/design-references/` and accumulates experience across ALL projects. Risk: one project's preferences contaminating another's recommendations. Needs clear categorization by project type (spa, fintech, editorial) rather than raw preference dumping.

### style-profiles.md scope creep
If style profiles become too detailed, they add significant token overhead when read by design-system-init. Need a lightweight format with quick-lookup tables (like color-psychology.md's summary table).

### write-code DS integration must be truly lightweight
The design-review skill explicitly caps overhead at ~500-1000 tokens (reads only tokens.json). The write-code integration must be similarly constrained — reading tokens.json for CSS variable names, not loading full design principles or references.

### No `.design-system/` directory in the repo
The repo itself does not contain a `.design-system/` directory. All design skills check for its existence at runtime in the target project. The new taste-profile file would also live in the target project's `.design-system/`, not in the skills repo.

## 8. Constraints & Infrastructure

### File structure convention
- Skills live in `skills/{skill-name}/SKILL.md` with optional `references/` and `assets/` subdirs
- Shared references in `shared/design-references/`
- All references are markdown files (no JSON/YAML configs)
- Relative paths from skills to shared: `../../shared/design-references/`

### SKILL.md format convention
All design skills follow:
```
---
name: {skill-name}
description: |
  {multi-line description}
  Use when: {trigger phrases in Russian and English}
---

# {Skill Title}

{One-line description}

## Phase N: {Phase Name}
{Steps with numbered lists}
**Checkpoint:** {what must be true to proceed}

## Final Check / Self-Verification
{Checklist of verification items}
```

### File naming constraint
All generated files in `.design-system/` must match `/^[a-z0-9-]+$/` (excluding extensions). Enforced in design-system-init Phase 3.1 and design-generate Phase 3.4.

### Language convention
- SKILL.md instructions: English
- User-facing messages within skills: Russian
- Reference files: Russian (color-psychology, color-principles) or English (design-tokens, font-pairing, component-patterns)
- New reference files (style-profiles, designer-experience): should follow Russian convention of existing design references

### Design skill activation pattern
Skills check for `.design-system/tokens.json` or `.design-system/` directory existence. If missing, they either stop with a Russian message redirecting to `/design-system-init`, or skip silently (design-review).

## 9. Key Answers to Research Questions

### Q1: Exact structure of each design SKILL.md
All follow: YAML frontmatter (name, description, trigger phrases) -> title -> description -> numbered phases with checkpoints -> final self-verification checklist. See Section 1 for per-skill breakdown.

### Q2: Where in design-system-init to add experience reading
**Location:** Between Phase 1 (Project Scan) checkpoint and Phase 2 (Interview) start. After scan results are collected but before the propose-first interview begins. Read `shared/design-references/designer-experience.md` and optionally `shared/design-references/style-profiles.md` to inform proposals. This is analogous to how the skill currently reads color-psychology.md and color-principles.md for palette proposals.

### Q3: Where in design-generate to add taste-profile reading
**Location:** Phase 0 (Readiness Check), after tokens.json validation (line 37-39). Add step: "Read `.design-system/taste-profile.md` (if exists) for aesthetic preferences." This informs layout selection (Phase 2) and component assembly (Phase 3).

### Q4: Where in design-retrospective to add taste-profile and experience writing
**Location:** After Phase 4 (Promote Principles) and before Phase 5 (Generate Next-Session Prompt). Add Phase 4.5 or renumber:
- Read existing taste-profile.md (if exists)
- Extract aesthetic preferences from session (color temperature preference, spacing density, layout style, typography choices)
- Merge with existing preferences (latest session takes priority for conflicting preferences)
- Write updated taste-profile.md
- Extract generalizable patterns for designer-experience.md (categorized by project type)

### Q5: Does write-code/code-writing skill exist?
**Yes, both exist:**
- `skills/write-code/SKILL.md` — thin wrapper (9 lines), delegates to code-writing
- `skills/code-writing/SKILL.md` — full 3-phase process: Preparation -> Implementation (TDD) -> Post-work

**Where to add DS integration:** Phase 1.2 "Read Project Context" in code-writing/SKILL.md. Add conditional: if UI files being written AND `.design-system/tokens.json` exists, read tokens.json for CSS variable names. Keep overhead at ~500-1000 tokens (same constraint as design-review).

### Q6: Format of shared/design-references files
Russian-language markdown. Structure varies by purpose:
- **Psychology/principles:** color profiles with headers, "what it conveys" lists, nuances, contexts, summary tables
- **Tokens schema:** JSON code blocks with naming conventions, CSS generation examples
- **Catalog:** table-like entries with URL/License/Style/Use-cases per library
- **Font pairing:** principle explanations + 10 self-contained combination entries with tokens.json mapping

New files should match this pattern: Russian text, practical structure, quick-lookup tables, concrete values/examples.

### Q7: How does do-task design-review hook work?
File: `skills/do-task/SKILL.md`, Step 2, item 4.
Condition: `.design-system/tokens.json` exists AND task changed UI files (`.tsx`, `.vue`, `.html`, `.css`, `.scss`).
Action: spawn `design-review` subagent on changed UI files.
Else: skip silently (no error, no message).
Key: calls ONLY design-review, not the full design pipeline.

Pattern for write-code integration: same dual-condition check (tokens.json exists + UI files involved), same silent skip, same minimal overhead.

## 10. Detailed SKILL.md Analysis

## Updated: 2026-03-26

### 10.1 New File Existence Check

None of the three new files exist yet anywhere in the repository:
- `.design-system/taste-profile.md` — not found (created per-project at runtime)
- `shared/design-references/designer-experience.md` — not found
- `shared/design-references/style-profiles.md` — not found

Existing `shared/design-references/` contains only: `color-psychology.md`, `color-principles.md`, `design-tokens.md`, `library-catalog.md`.

No existing SKILL.md references `taste-profile`, `designer-experience`, or `style-profiles` except one incidental mention: `design-retrospective/SKILL.md` line 107 uses the phrase "taste profile" (lowercase, in prose) inside the `design-principles.md` template header — not as a file reference.

### 10.2 design-system-init/SKILL.md (171 lines)

**Phase structure:**
| Phase | Lines | Header |
|-------|-------|--------|
| Frontmatter | 1-11 | `---` YAML block |
| Title + description | 13-15 | `# Design System Init` |
| Phase 0: Project Readiness | 17-26 | `## Phase 0: Project Readiness` |
| Phase 1: Project Scan | 28-48 | `## Phase 1: Project Scan` |
| Phase 2: Interview | 50-109 | `## Phase 2: Interview` |
| - 2.1 Project Context and Mood | 54-58 | `### 2.1 Project Context and Mood` |
| - 2.2 Color Palette | 60-73 | `### 2.2 Color Palette` |
| - 2.3 Typography | 75-84 | `### 2.3 Typography` |
| - 2.4 Spacing | 86-93 | `### 2.4 Spacing` |
| - 2.5 Radii, Shadows, Breakpoints | 94-101 | `### 2.5 Radii, Shadows, Breakpoints` |
| - 2.6 Components | 103-109 | `### 2.6 Components` |
| Phase 3: Build | 111-145 | `## Phase 3: Build` |
| Phase 4: Verify | 147-160 | `## Phase 4: Verify` |
| Final Check | 162-172 | `## Final Check` |

**Insertion point A — experience + style-profiles reading (before interview):**
Insert between line 52 and line 54. Current text at boundary:

```
Line 52: Propose-first approach: suggest concrete values based on scan results, user confirms or adjusts. One topic at a time.
Line 53: (empty)
Line 54: ### 2.1 Project Context and Mood
```

New content goes after line 52 (after "Propose-first approach" sentence, before `### 2.1`). This is where to add steps for reading `designer-experience.md` (by project category) and `style-profiles.md` (for profile matching after mood is determined).

**Insertion point B — style-profile matching (after mood, before palette):**
Alternative: insert between 2.1 (mood determined) and 2.2 (palette). After line 58, before line 60. Once mood + category are known, match to a style-profile and use its recipes for proposing palette/typography/spacing.

**Insertion point C — taste-profile reading for update flow:**
Line 22: `- If "update" — load existing tokens.json and use as defaults in interview`. Add: also read `.design-system/taste-profile.md` (if exists) and use accumulated preferences as additional defaults.

**Insertion point D — two-layer description instruction:**
Add as a general instruction near the top of Phase 2 (after line 52) or as a standalone paragraph before `### 2.1`. Instructs the agent to describe all design proposals in two layers: human-readable first, then technical details.

**Final Check additions (line 162-172):**
Add checklist items for: style-profile was offered if applicable, experience was consulted if available, two-layer descriptions used.

**Existing references in this file (4):**
- `../../shared/design-references/color-psychology.md` (line 58)
- `../../shared/design-references/color-principles.md` (line 70)
- `references/font-pairing.md` (line 76)
- `../../shared/design-references/design-tokens.md` (line 115)

### 10.3 design-generate/SKILL.md (206 lines)

**Phase structure:**
| Phase | Lines | Header |
|-------|-------|--------|
| Frontmatter | 1-12 | `---` YAML block |
| Title + description + flow diagram | 14-28 | `# Design Generate` |
| Phase 0: Readiness Check | 30-39 | `## Phase 0: Readiness Check` |
| Phase 1: Parse Request | 41-57 | `## Phase 1: Parse Request` |
| Phase 2: Select Layout | 59-74 | `## Phase 2: Select Layout` |
| Phase 3: Assemble & Generate | 76-107 | `## Phase 3: Assemble & Generate` |
| - 3.1 Assemble page | 78-84 | `### 3.1 Assemble page` |
| - 3.2 Generate HTML | 86-92 | `### 3.2 Generate HTML` |
| - 3.3 Generate SVG | 94-99 | `### 3.3 Generate SVG` |
| - 3.4 Save files | 101-107 | `### 3.4 Save files` |
| Phase 4: Present & Iterate | 109-144 | `## Phase 4: Present & Iterate` |
| Phase 5: Before/After Collage | 146-194 | `## Phase 5: Before/After Collage` |
| Final Check | 196-206 | `## Final Check` |

**Insertion point A — taste-profile reading (Phase 0):**
Insert between line 37 and line 39. Current text at boundary:

```
Line 37: 2. Parse tokens into CSS custom properties for use in generated pages (following the mapping pattern from [design-tokens.md](../../shared/design-references/design-tokens.md))
Line 38: (empty)
Line 39: **Checkpoint:** tokens.json exists, parses as valid JSON, contains colors/typography/spacing sections.
```

Add step 3: "Read `.design-system/taste-profile.md` (if exists). Extract aesthetic preferences (color temperature, spacing density, boldness level, anti-patterns). Pass to Phase 2 and Phase 3 as context. If file missing or malformed — proceed without preferences."

The checkpoint on line 39 should be updated to optionally mention taste-profile.

**Insertion point B — two-layer description instruction:**
Add to Phase 4 (Present & Iterate), around line 111-115. When presenting results and explaining layout/style choices, use two-layer descriptions.

**Final Check additions (line 196-206):**
Add checklist item: taste-profile was consulted if available.

**Existing references in this file (4):**
- `../../shared/design-references/design-tokens.md` (line 37)
- `references/generation-guide.md` (line 61)
- `references/component-patterns.md` (lines 54, 81)
- `references/grid-techniques.md` (line 67)

### 10.4 design-review/SKILL.md (111 lines)

**Section structure (no numbered phases):**
| Section | Lines | Header |
|---------|-------|--------|
| Frontmatter | 1-10 | `---` YAML block |
| Title + description | 12-14 | `# Design Review` |
| When to Activate | 16-32 | `## When to Activate` |
| What to Read | 34-43 | `## What to Read` |
| What to Check | 45-68 | `## What to Check` |
| - Colors | 49-57 | `### Colors` |
| - Spacing | 58-61 | `### Spacing` |
| - Typography | 63-66 | `### Typography` |
| - Non-DS Custom Properties | 67-68 | `### Non-DS Custom Properties` |
| How to Report | 70-94 | `## How to Report` |
| Scope Guard | 96-111 | `## Scope Guard` |

**Insertion point — two-layer description (optional):**
In "How to Report" section, after line 80 (the `found X in Y -> use Z` format examples). Add optional instruction: for non-obvious replacements (e.g., approximate color match, spacing that is close but not exact), add a brief human-readable explanation of why the suggested token is the right match. This is optional per user-spec: "design-review — опционально, только при неочевидных заменах".

Current text at boundary:
```
Line 80: Examples:
Line 81: - `found color #3B82F6 in Button.tsx:14 -> use --color-primary-500`
Line 82: - `found gap: 8px in Card.vue:32 -> use --space-2 (8px) or consider --space-4 (16px) for component spacing`
Line 83: - `found font-family: Arial in Header.css:5 -> use --font-heading (Inter, system-ui, sans-serif)`
```

Insert after line 83: instruction for optional explanation on non-obvious matches.

**No taste-profile or experience integration needed** — this skill is read-only, lightweight, and explicitly scope-guarded. No changes except the optional two-layer addition.

**Existing references in this file (2):**
- `../../shared/design-references/design-tokens.md` (line 43)
- `../../shared/design-references/color-principles.md` (line 56)

### 10.5 design-retrospective/SKILL.md (167 lines)

**Phase structure:**
| Phase | Lines | Header |
|-------|-------|--------|
| Frontmatter | 1-10 | `---` YAML block |
| Title + description | 12-19 | `# Design Retrospective` |
| Phase 1: Collect Evidence | 21-36 | `## Phase 1: Collect Evidence` |
| Phase 2: Identify Patterns | 38-54 | `## Phase 2: Identify Patterns` |
| Phase 3: Write Lessons | 56-93 | `## Phase 3: Write Lessons` |
| Phase 4: Promote Principles | 95-124 | `## Phase 4: Promote Principles` |
| Phase 5: Generate Next-Session Prompt | 126-154 | `## Phase 5: Generate Next-Session Prompt` |
| Self-Verification | 156-167 | `## Self-Verification` |

**Insertion point A — taste-profile + experience writing (new Phase 4.5):**
Insert between line 124 and line 126. Current text at boundary:

```
Line 122: 6. If no rules qualify for promotion — report: "Нет правил для промоушена (ни одно не появилось 3+ раз)."
Line 123: (empty)
Line 124: **Checkpoint:** qualifying rules promoted to `design-principles.md`, or reported that none qualify.
Line 125: (empty)
Line 126: ## Phase 5: Generate Next-Session Prompt
```

New Phase 4.5 goes after line 124 (after Phase 4 checkpoint), before line 126 (Phase 5). This phase:
1. Reads existing `.design-system/taste-profile.md` (if exists)
2. Extracts aesthetic preferences from session corrections (color temperature, spacing density, boldness, anti-patterns)
3. Merges with existing preferences (latest wins for conflicts, marks overridden preferences as "пересмотрено")
4. Writes updated `taste-profile.md`
5. Determines project category (landing/webapp/admin/portfolio)
6. Reads relevant section from `shared/design-references/designer-experience.md`
7. Appends generalizable session learnings to that section (append-only)
8. If file/section missing — creates it

**Insertion point B — two-layer description instruction:**
Add to Phase 3 (Write Lessons), after line 80 (the lesson format template). When writing Problem/Cause/Solution text, use human-readable language first, then technical specifics.

**Insertion point C — taste-profile in next-session prompt:**
Phase 5 prompt structure (lines 130-150): add a section for accumulated preferences from taste-profile.md in the prompt template, between "Накопленные принципы" and "Что было сделано в последней сессии".

**Self-Verification additions (line 156-167):**
Add checklist items: taste-profile.md created/updated, designer-experience.md updated (if applicable), two-layer descriptions used in lessons.

**Existing references in this file (0):**
No external reference links. The skill reads only `.design-system/` runtime artifacts.

### 10.6 code-writing/SKILL.md (141 lines)

**Phase structure:**
| Phase | Lines | Header |
|-------|-------|--------|
| Frontmatter | 1-10 | `---` YAML block |
| Title | 12 | `# Code Writing` |
| Phase 1: Preparation | 14-47 | `## Phase 1: Preparation` |
| - 1.1 Parse Requirements | 16-19 | `1. **Parse Requirements**` |
| - 1.2 Read Project Context | 21-34 | `2. **Read Project Context (Graceful)**` |
| - 1.3 Analyze & Review Approach | 36-47 | `3. **Analyze & Review Approach**` |
| Phase 2: Implementation (TDD) | 49-71 | `## Phase 2: Implementation (TDD)` |
| Phase 3: Post-work | 73-129 | `## Phase 3: Post-work` |
| Self-Verification | 131-141 | `## Self-Verification` |

**Insertion point — DS token integration (Phase 1.2):**
Insert after line 34, before line 36. Current text at boundary:

```
Line 34:    **No project patterns?** Apply baseline from [universal-patterns.md](references/universal-patterns.md) — naming, error handling, structure.
Line 35: (empty)
Line 36: 3. **Analyze & Review Approach**
```

New conditional step goes after line 34 (end of "Read Project Context" section). Add:

```
   **UI task with design system?** If the task involves UI files (.tsx, .vue, .html, .css, .scss)
   AND `.design-system/tokens.json` exists in the project root:
   - Read `tokens.json` — extract CSS custom property names (~500-1000 tokens overhead)
   - When writing styles, use `var(--token-name)` instead of hardcoded color/spacing/typography values
   - If `tokens.json` is missing or malformed → skip silently, do not block coding
```

**Self-Verification additions (line 131-141):**
Add checklist item: if UI task with DS, tokens were applied (no hardcoded values matching token equivalents).

### 10.7 Shared Design References Format Analysis

All four existing files follow a consistent format that new files (`designer-experience.md`, `style-profiles.md`) must match:

| File | Lines | Language | Structure |
|------|-------|----------|-----------|
| color-psychology.md | 324 | Russian | Title + subtitle -> Part I (color profiles by color, each: nature/what it conveys/nuance/contexts) -> Part II (10 combinations, each: feeling/reading/contexts/danger) -> Summary table |
| color-principles.md | 187 | Russian | Title + subtitle -> 10 numbered principles, each: essence/how it works/nuance/effect, some with CSS code blocks. Star ratings for difficulty. |
| design-tokens.md | 114 | English | Title -> JSON schema code block -> Naming conventions -> CSS generation code block -> Extraction guide -> Dark mode section |
| library-catalog.md | 104 | Russian | Title + description -> Sections by type (icons, UI elements, illustrations) -> entries: name, URL, License, Style, Use-cases |

**Pattern for new files:**
- `designer-experience.md` — Russian, structured by project category. Each category section: preferences, what worked, anti-patterns. Append-only design with dated entries.
- `style-profiles.md` — Russian, structured by style name. Each profile: core philosophy (1 sentence), specific techniques (concrete values: fonts, spacing ratios, color approaches), contexts where applicable, danger/pitfalls. Quick-lookup table at end mapping mood -> profile.

### 10.8 generation-guide.md Format (first 50 lines)

Located at `skills/design-generate/references/generation-guide.md`. Starts with a table of contents linking to sections: Page Assembly Process, Layout Selection Guide, Basic Layout Patterns, Advanced Grid Patterns, Responsive Behavior, Placeholder Content, HTML Generation, SVG Generation, Device Frames, Iterating on Designs. Contains a selection table mapping request types to layout patterns with basic/advanced categorization.

### 10.9 Summary of All Changes Required

| File | Type | Changes |
|------|------|---------|
| `skills/design-system-init/SKILL.md` | MODIFY | +experience reading (Phase 2 top), +style-profile matching (after 2.1), +taste-profile in update flow (Phase 0), +two-layer descriptions, +Final Check items |
| `skills/design-generate/SKILL.md` | MODIFY | +taste-profile reading (Phase 0), +two-layer descriptions (Phase 4), +Final Check items |
| `skills/design-review/SKILL.md` | MODIFY | +optional two-layer for non-obvious matches (How to Report section) |
| `skills/design-retrospective/SKILL.md` | MODIFY | +new Phase 4.5 (taste-profile + experience writing), +two-layer in lessons, +taste-profile in next-session prompt, +Self-Verification items |
| `skills/code-writing/SKILL.md` | MODIFY | +conditional DS token step (Phase 1.2), +Self-Verification item |
| `shared/design-references/designer-experience.md` | CREATE | Cross-project experience by category (landing, webapp, admin, portfolio). Russian. Append-only sections. |
| `shared/design-references/style-profiles.md` | CREATE | 6+ style profiles (luxury, brutalist, editorial, minimal, playful, corporate). Russian. Quick-lookup table. |
