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
