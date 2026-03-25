# Code Research: design-pipeline

Date: 2026-03-24

## 1. Existing Like-Figma Prototype

The `like-figma` skill is the closest existing work to the design pipeline. It is a fully functional single procedural skill for design generation.

### Files

| File | Summary |
|------|---------|
| `skills/like-figma/SKILL.md` (230 lines) | Procedural skill with 5 phases: Project Readiness, Design System Discovery (interview), Build Design System, Generate Design, Code-to-Design. 4 modes: Init, Generate, Code-to-Design, Evolve. |
| `skills/like-figma/references/design-tokens.md` | Token schema (`tokens.json`): colors (primary/secondary/neutral/semantic), typography (families, sizes, weights, line-heights), spacing, radii, shadows, breakpoints. CSS custom property generation patterns. Dark mode support. |
| `skills/like-figma/references/component-patterns.md` | BEM-like component structure. 8 core components: Button, Input, Card, Badge, Alert, Avatar, Navigation, Table. Variant system (block/variant/size/state/child). Slot pattern for composable content. |
| `skills/like-figma/references/generation-guide.md` | Page assembly process: parse request, select layout, place components, add content, generate HTML + SVG. 5 layout patterns (single column, sidebar+content, grid, hero+sections, split screen). Responsive behavior. Placeholder content guidelines. SVG generation rules. Iteration vocabulary ("bigger", "more contrast", "simpler"). |
| `skills/like-figma/assets/preview-template.html` | HTML template with CSS reset, full token set as CSS custom properties, placeholders for page styles and content. |

### What Can Be Reused

- **Design tokens schema** (`references/design-tokens.md`) -- complete token format, naming conventions, CSS variable generation. Directly reusable by `design-system-init`.
- **Component patterns** (`references/component-patterns.md`) -- BEM variant system, slot pattern, core component catalog. Reusable by `design-generate`.
- **Generation guide** (`references/generation-guide.md`) -- layout patterns, responsive behavior, placeholder content rules. Reusable by `design-generate`.
- **Preview template** (`assets/preview-template.html`) -- HTML template with tokens. Reusable by `design-generate`.
- **Interview structure** (Phase 1 of SKILL.md) -- design system discovery interview pattern: project scan, brand/colors, typography, spacing, components. Reusable by `design-spec` or `design-system-init`.

### What like-figma Lacks (Design Pipeline Must Add)

- No concept of BEAUTY/aesthetics evaluation -- purely mechanical token application.
- No color theory: no palette generation from a seed color, no harmony rules (complementary, analogous, triadic).
- No project context analysis for aesthetic matching -- doesn't consider the "mood" or "personality" of a project.
- No review/retrospective cycle -- generates and presents, no structured quality gates.
- No integration with the development methodology pipeline (no specs, tasks, sessions).
- No multi-step pipeline (spec -> plan -> decompose -> execute -> review -> retro).

## 2. Methodology Structure: How Skill Pipelines Are Organized

### Development Pipeline (Reference Pattern)

The existing development pipeline has this flow:

```
/new-user-spec -> /new-tech-spec -> /decompose-tech-spec -> /do-feature or /do-task -> /retrospective -> /done
```

Each step is a separate skill invocation. Each has:
- A **shim skill** (e.g., `new-user-spec/SKILL.md`) that redirects to the real skill (e.g., `user-spec-planning/SKILL.md`).
- A **procedural SKILL.md** with explicit phases, checkpoints, and self-verification.
- **Validators/agents** that run as subagents for quality gates.
- **Templates** in `shared/work-templates/` for output format.
- **Blocking gates** requiring user approval before proceeding.

### Skill Referencing Between Steps

Skills reference each other by:
1. **File path**: `$AGENTS_HOME/skills/{skill}/SKILL.md` -- used for shim -> real skill loading.
2. **Agent name**: `code-reviewer`, `security-auditor` -- used when spawning subagents.
3. **Skill name in frontmatter**: `skills: [code-writing]` -- used in task files to specify which skill the worker loads.
4. **Template paths**: `$AGENTS_HOME/shared/work-templates/{template}` -- for output formatting.

### Session Management (from feature-execution)

- `session-plan.md` groups waves into sessions by LOC budget (~1200 lines/session).
- At session boundary: HARD STOP, session report, documentation sync, next-session prompt generation.
- `checkpoint.yml` persists wave state for recovery.
- `next-session-prompt.md` generated from `shared/work-templates/session-prompt.md.template`.
- User must explicitly start next session -- no auto-transition.

### Wave Structure

- Wave 1, 2, ...: implementation waves (parallel tasks within wave, sequential waves).
- Audit Wave: 3 auditors (code, security, test) in parallel.
- Final Wave: QA + deploy.

## 3. Interview Patterns

### user-spec-planning Interview Pattern

- **Propose-first approach**: agent proposes concrete answers based on Project Knowledge and code, user confirms/adjusts.
- **3-4 items per batch**, as many batches as needed.
- **Scoring**: 0-100% per item, threshold 85% for required items.
- **3 interview cycles**: general understanding -> code-informed refinement -> review & finalize.
- **Completeness checker**: `interview-completeness-checker` agent validates coverage.
- State persisted in `work/{feature}/logs/userspec/interview.yml`.

### Interview Templates

| Template | Location | Purpose |
|----------|----------|---------|
| `shared/interview-templates/feature.yml` | Feature planning | 3 phases: feature overview (8 items), user experience (10 items), integration (5 items). Scoring, conversation history, decision rules. |
| `shared/interview-templates/skill.yml` | Skill creation | 3 phases: skill overview (4 items), usage scenarios (3 items), output & resources (3 items). Threshold 70%. |

### Work Templates

| Template | Purpose |
|----------|---------|
| `shared/work-templates/user-spec.md.template` | User spec output format: sections for what/why/how/acceptance criteria/constraints/risks/testing/verification. |
| `shared/work-templates/tech-spec.md.template` | Tech spec with solution, architecture, decisions, data models, dependencies, testing, implementation tasks. |
| `shared/work-templates/tasks/task.md.template` | Individual task file format. |
| `shared/work-templates/session-plan.md.template` | Session grouping plan with LOC budgets. |
| `shared/work-templates/session-prompt.md.template` | Handoff prompt between sessions. |
| `shared/work-templates/execution-plan.md.template` | Execution plan for feature-execution. |
| `shared/work-templates/checkpoint.yml.template` | Wave state persistence. |
| `shared/work-templates/decisions.md.template` | Decision log format. |

## 4. Skill Structure Conventions

### Standard SKILL.md Format (from skill-master)

```yaml
---
name: kebab-case-name  # required, <=64 chars
description: |         # required, <=1024 chars
  What the skill does.
  Use when: "trigger phrase 1", "trigger phrase 2"
---
```

Body: core workflow + links to references. Under 500 lines.

### Directory Structure

```
skill-name/
  SKILL.md              # required
  references/           # optional: conditional content loaded as needed
  scripts/              # optional: executable code
  assets/               # optional: files used in output (templates, images)
```

No README, CHANGELOG, or other docs.

### Reference Linking Patterns

- **Pattern A (strong)**: action-embedded -- "Write tests following patterns from [testing-guide.md](references/testing-guide.md)".
- **Pattern B (basic)**: conditional -- "For tracked changes, see [REDLINING.md] -- revision marks, accept/reject."
- Anti-pattern: passive resource catalog at end of file.

### Procedural Skills Must Have

- Explicit phases with numbered steps.
- Checkpoints after each phase.
- Self-verification section at end.

### Shim Skills

Shim skills (e.g., `do-feature`, `decompose-tech-spec`, `new-user-spec`) are thin wrappers that say "Read and follow `$AGENTS_HOME/skills/{real-skill}/SKILL.md`". They provide the `/command-name` entry point while the real logic lives in the canonical skill.

## 5. Existing Validators and Agents

### Validators Reusable by Design Pipeline

| Agent | File | Relevance |
|-------|------|-----------|
| `skill-checker` | `agents/skill-checker.md` | Validates skills against skill-master standards. Use after creating each design-* skill. |
| `interview-completeness-checker` | `agents/interview-completeness-checker.md` | Validates interview coverage. Could be adapted or reused for design interview completeness. |
| `completeness-validator` | `agents/completeness-validator.md` | Bidirectional requirements traceability. Reusable for design-spec validation. |
| `task-validator` | `agents/task-validator.md` | Template compliance for task files. Reusable as-is if design pipeline uses same task format. |
| `reality-checker` | `agents/reality-checker.md` | Validates tasks against real codebase. Reusable as-is. |
| `task-creator` | `agents/task-creator.md` | Generates task files from tech-spec. Reusable as-is. |
| `code-researcher` | `agents/code-researcher.md` | Codebase research. Reusable as-is for design context gathering. |

### Reviewers Potentially Relevant

| Agent | Relevance |
|-------|-----------|
| `code-reviewer` | Review generated CSS/HTML code quality. |
| `prompt-reviewer` | If design pipeline uses prompts for LLM-based generation. |
| `documentation-reviewer` | Review generated design system documentation. |

### Agents That Would Need Design-Specific Counterparts

- **design-reviewer** -- new agent needed for aesthetic review (color harmony, typography pairing, spacing consistency, visual hierarchy).
- **design-spec-validator** -- new agent to validate design spec completeness (color palette coverage, typography scale, component list adequacy).

## 6. Session Management Details

### feature-execution Session Flow

1. **Phase 0: Pre-flight** -- verify tech-spec approved, session-plan exists and approved, task files exist, codebase ready.
2. **Phase 1: Initialization** -- read specs, build execution plan, present session scope for confirmation (blocking gate).
3. **Phase 2: Execute Wave** -- spawn workers, run reviewers (max 3 rounds), commit per wave.
4. **Phase 3: Wave Transition** -- update statuses, session boundary check.
5. **Session End Protocol** -- session report, documentation sync (decisions.md, task statuses, tech-spec checkboxes, checkpoint.yml), next-session prompt generation, HARD STOP.

### Key Session Conventions

- LOC budget: ~1200 lines per session (+-300).
- Audit Wave + Final Wave always in last session.
- `git add -f` required for `work/{feature}/logs/` files (gitignored by global `logs/` rule).
- Session prompt must be self-contained (zero context from previous session).

### AGENTS.md Global Rules That Apply

- All communication in Russian, code/commands in English.
- Session End Protocol block required after every pipeline step.
- Intermediate summaries after each phase checkpoint.
- `decisions.md` updated after every significant decision.
- Pipeline navigation: always tell user the next step.

## 7. Architectural Observations for Design Pipeline

### Design Pipeline Maps to Development Pipeline

| Dev Pipeline Step | Design Pipeline Analog | Notes |
|-------------------|----------------------|-------|
| `/new-user-spec` (user-spec-planning) | `design-spec` | Interview for design requirements, aesthetic preferences, project mood. |
| `/new-tech-spec` (tech-spec-planning) | `design-plan` | Technical design plan: token structure, component list, layout decisions. |
| `/decompose-tech-spec` (task-decomposition) | `design-decompose` | Break design plan into tasks (create tokens, build components, generate pages). |
| `/do-feature` / `/do-task` | `design-execute` | Execute design tasks. |
| N/A (new) | `design-generate` | Generate designs from descriptions (from like-figma Phase 3). |
| N/A (new) | `design-review` | Aesthetic review of generated designs. |
| `/retrospective` | `design-retrospective` | Extract lessons from design process. |
| `/done` | `design-done` | Archive, update PK. |
| N/A (new) | `design-system-init` | Initialize design system (from like-figma Phase 1-2). |

### Key Resources from like-figma to Distribute

- `design-tokens.md` -> shared by `design-system-init` and `design-generate`.
- `component-patterns.md` -> used by `design-generate`.
- `generation-guide.md` -> used by `design-generate`.
- `preview-template.html` -> used by `design-generate`.
- Interview pattern (Phase 1 of like-figma) -> integrated into `design-spec`.

### Missing Pieces to Create

1. **Color theory reference** -- palette generation, harmony rules, contrast checking (WCAG AA/AAA), color psychology.
2. **Typography pairing reference** -- rules for heading + body font combinations, scale ratios.
3. **Aesthetic evaluation criteria** -- what makes a design "beautiful" (whitespace, alignment, visual hierarchy, consistency).
4. **Project mood detection** -- how to infer aesthetic direction from existing codebase (dark/light, minimal/rich, corporate/playful).
5. **Design interview template** -- new `shared/interview-templates/design.yml` with design-specific items (brand colors, mood, reference sites, target audience aesthetics).
6. **Design-specific agents** -- `design-reviewer` for aesthetic quality, `design-spec-validator` for design spec completeness.

---

## Updated: 2026-03-25 — Implementation-Level Deep Dive

---

## 8. like-figma SKILL.md: Complete Flow Analysis

**File:** `skills/like-figma/SKILL.md` (233 lines, procedural skill)

### Frontmatter

```yaml
name: like-figma
description: |
  Generates visual designs (HTML/CSS, SVG) from text descriptions using a project's
  design system. Creates and maintains design systems through interview.
  Use when: "сгенерируй дизайн", "сделай макет", "покажи как выглядит",
  "создай дизайн-систему", "generate design", "create mockup", "design system"
```

### Mode Detection Logic

The skill has a mode router at the top:
- **Mode A: Init** -- DS does not exist or user asks to create. Goes to Phase 1.
- **Mode B: Generate** -- DS exists, user describes page/component. Goes to Phase 3.
- **Mode C: Code-to-Design** -- user points at existing code, wants visual preview. Goes to Phase 4.
- **Mode D: Evolve** -- user wants to add/change tokens or components in existing DS. Goes to Phase 2 (skip interview).

**Migration note:** Mode A maps to `design-system-init`. Mode B maps to `design-generate`. Mode C is post-MVP. Mode D is a sub-mode of `design-system-init` (with `--evolve` flag or similar detection).

### Phase 0: Project Readiness

Checks if skill is accessible from current project (`.agents/skills/like-figma/` or reference in `AGENTS.md`). Offers Copy or Reference approach.

**Migration note:** Not needed for design-pipeline skills -- they will be in the global agents repo, not project-local.

### Phase 1: Design System Discovery (Interview)

Two sub-steps:
1. **1.1 Project Scan** -- reads CSS/SCSS/Tailwind config for existing colors, fonts, spacing. Reads component files for UI patterns. Checks for existing CSS custom properties and theme files. Notes tech stack.
2. **1.2 Interview** -- propose-first approach. Topics in order: Brand & Colors, Typography, Spacing & Layout, Components. One topic at a time, summarize after every 3-4 topics.

**Key interview items:**
- Primary, secondary, accent, neutral, semantic colors
- Font families (heading, body, mono), size scale, weight scale, line heights
- Spacing scale, border radii, shadows, breakpoints
- Component list selection from scan results

**Checkpoint:** Color palette defined, typography scale decided, spacing scale decided, component list approved.

**Migration note:** This interview pattern transfers directly to `design-system-init`. The new skill must ADD: (a) project context/mood interview from color-psychology.md, (b) font pairing guidance from font-pairing.md (to be created), (c) golden ratio / Fibonacci spacing options from grid-techniques.md.

### Phase 2: Build Design System

Creates `tokens.json` from `design-tokens.md` schema. Creates component HTML files per `component-patterns.md`. Generates README.md. Verification checkpoint.

**Migration note:** Transfers to `design-system-init` Phase 2 (build). The token schema and component patterns references are reusable as-is.

### Phase 3: Generate Design

Sub-steps: understand request, generate HTML + SVG output, present result, checkpoint. Uses `generation-guide.md` for layout selection and `preview-template.html` as base.

**Migration note:** Transfers directly to `design-generate`. Needs additions: (a) color-principles.md integration for palette validation during generation, (b) grid-techniques.md for layout selection beyond the 5 basic patterns.

### Phase 4: Code-to-Design

Reads source code, maps to DS components, generates preview. Post-MVP for design pipeline.

### Final Check

Universal verification: all files use tokens, HTML opens in browser, SVG renders, structure consistent, README current.

## 9. like-figma References: Detailed Content Map

### references/design-tokens.md (114 lines)

Complete `tokens.json` schema with example values:
- `colors`: primary (50-900 scale), secondary, neutral (0-1000), semantic (success/warning/error/info), background (default/surface/elevated), text (primary/secondary/disabled/inverse)
- `typography`: families (heading/body/mono), sizes (xs through 4xl), weights (regular/medium/semibold/bold), lineHeights (tight/normal/relaxed)
- `spacing`: numeric scale (1=4px through 16=64px)
- `radii`: sm/md/lg/full
- `shadows`: sm/md/lg
- `breakpoints`: sm/md/lg/xl

CSS custom property generation pattern: `--{category}-{path}` with hyphens.

Extraction rules for existing projects: Tailwind config > CSS custom properties > SCSS variables > hardcoded values (priority order).

Optional dark mode: `colorsDark` section, `[data-theme="dark"]` selector.

**Reuse plan:** Copy to `skills/design-system-init/references/design-tokens.md`. No modifications needed -- schema is complete.

### references/component-patterns.md (144 lines)

Self-contained HTML showcase structure for each component. 8 core components with variant definitions:
- Button: 5 variants (primary/secondary/outline/ghost/destructive), 3 sizes, 3 states
- Input: 3 variants, 3 sizes, extras (label, helper, icon)
- Card: 3 variants, 4 slots (header/body/footer/media)
- Badge: 5 variants, 2 sizes
- Alert: 4 variants, 4 slots
- Avatar: 3 variants, 4 sizes, 2 shapes
- Navigation: 3 variants, items with icon/active/disabled
- Table: features (header/rows/striped/hoverable), 2 slots

BEM-like naming: `.component`, `.component--variant`, `.component--size`, `.component__child`.

Slot pattern: semantic sections that can be omitted.

New component creation protocol: closest pattern, define variants, sizes, states, use tokens, showcase all.

**Reuse plan:** Copy to `skills/design-generate/references/component-patterns.md`. No modifications needed.

### references/generation-guide.md (178 lines)

6-step page assembly: parse request, select layout, place components, add content, generate HTML, generate SVG.

5 layout patterns with CSS:
1. Single Column (articles, forms) -- `max-width: 768px; margin: 0 auto`
2. Sidebar + Content (dashboards) -- `grid-template-columns: 240px 1fr`
3. Grid (catalogs) -- `repeat(auto-fill, minmax(300px, 1fr))`
4. Hero + Sections (landings) -- stacked full-width sections
5. Split Screen (auth) -- `grid-template-columns: 1fr 1fr; min-height: 100vh`

Responsive: mobile-first flex, desktop grid. Sidebar collapses to top nav.

Placeholder content rules: diverse realistic names, short meaningful sentences (no lorem ipsum), colored rectangles for images, realistic numbers/dates.

SVG generation: viewBox 1440x900 (desktop) or 390x844 (mobile). Absolute-positioned rects/text. Embedded colors (no CSS vars). Device frames optional.

Iteration vocabulary: "bigger/smaller" (spacing/fonts), "more contrast" (darken text), "simpler" (reduce components), "more detail" (add secondary info).

**Reuse plan:** Copy to `skills/design-generate/references/generation-guide.md`. Enrich with grid-techniques.md patterns (add 10 advanced layouts alongside 5 basic ones).

### assets/preview-template.html (99 lines)

Complete HTML template with:
- CSS reset (box-sizing, margin, font-smoothing, img/input resets)
- Full token set as `:root` CSS custom properties (all categories)
- Base body styles using tokens
- Placeholders: `{{PAGE_TITLE}}`, `{{PAGE_STYLES}}`, `{{PAGE_CONTENT}}`

**Reuse plan:** Copy to `skills/design-generate/assets/preview-template.html`. No modifications needed -- token values get replaced from actual `tokens.json`.

## 10. do-task SKILL.md: Design-Review Integration Point

**File:** `skills/do-task/SKILL.md` (147 lines, procedural skill)

### Flow Structure

4 main steps:
1. **Step 0: Pre-flight Checks** -- verify session-plan, task belongs to current session, codebase readiness, scope confirmation (blocking gate).
2. **Step 1: Read Task** -- read task file, verify status, update to `in_progress`, read context files.
3. **Step 2: Execute** -- load skills, follow workflow, git commit, run reviewers (max 3 rounds).
4. **Step 3: Verify** -- check acceptance criteria, smoke tests, user verification.
5. **Step 4: Complete** -- decisions.md entry, status update, tech-spec checkbox, session boundary check.

### Exact Integration Point for design-review

The integration should go between Step 2 (Execute) and Step 3 (Verify). Specifically, after the code commit in Step 2.3 and before running reviewers in Step 2.4.

**Current Step 2 flow:**
```
2.1 Load skills
2.2 Follow skill workflow
2.3 Git commit implementation
2.4 For each reviewer from task's "Reviewers" section → spawn subagent
```

**Proposed addition** -- insert between 2.3 and 2.4:
```
2.3.1 Design consistency check (if applicable):
  - Check if `.design-system/tokens.json` exists in the project
  - Check if task changed UI files (.tsx/.vue/.html/.css/.scss)
  - If both true → use design-review subagent:
    pass tokens.json path + list of changed UI files
  - Apply recommendations → git commit: "style: apply design system tokens for task {N}"
```

This is minimal (~5 lines of additions to do-task SKILL.md). The design-review skill handles all logic internally.

### Session Boundary Protocol

Step 4 has a comprehensive session end protocol with HARD STOP, session report, documentation sync, next-session prompt generation. Design-retrospective should hook into this -- after the last task of a design-related feature, suggest running `/design-retrospective`.

## 11. skill-master SKILL.md: Skill Creation Conventions

**File:** `skills/skill-master/SKILL.md` (420 lines, informational skill)

### Key Conventions for Creating design-* Skills

**Frontmatter rules:**
- `name`: kebab-case, <=64 chars, unique
- `description`: third person, <=1024 chars, include "Use when:" with 5-10 trigger phrases (Russian + English)
- No other frontmatter fields required (optional: `argument-hint`, `disable-model-invocation`, `allowed-tools`, model override -- see `references/frontmatter-options.md`)

**Body rules:**
- Core workflow + links to references. Under 500 lines.
- Concise: "Does Codex really need this explanation?"
- Positive instructions over negative ("Write in prose" not "Don't use bullets")
- Add motivation for rules (WHY it matters)
- Maximum one emphasis word per skill (ideally zero)

**Skill types applicable:**
- `design-system-init`: procedural (strict phase sequence: scan -> interview -> build -> verify)
- `design-generate`: procedural (parse -> layout -> generate -> present -> iterate)
- `design-review`: informational (read tokens, scan files, apply criteria -- no strict sequence)
- `design-retrospective`: procedural (collect evidence -> identify patterns -> write lessons -> report)

**Reference linking -- required patterns:**
- Pattern A (strong, action-embedded): "Create tokens following schema from [design-tokens.md](references/design-tokens.md)"
- Pattern B (conditional): "**Working with color palette?** Read [color-psychology.md](references/color-psychology.md) -- emotional map, context matching"
- Anti-pattern to avoid: passive resource catalog at end of file

**Subagent delegation rules:**
- Reviews, research, validation should use subagents for isolated context
- Skill holds methodology (WHAT/HOW), Agent adds isolation + output format
- Keep detailed agent prompts OUT of SKILL.md

**Validation after creation:**
- Run `skill-checker` subagent on each new skill
- Self-check list: frontmatter format, body <500 lines, all refs exist, no extra docs, Pattern A/B links, positive instructions, no emphasis words
- Procedural: phases, checkpoints, self-verification section
- Informational: sections by logic, decision frameworks (YES if / NO if)

## 12. retrospective SKILL.md: Pattern for design-retrospective

**File:** `skills/retrospective/SKILL.md` (122 lines, procedural skill)

### Structure to Mirror

4 phases:
1. **Collect Evidence** -- reads `decisions.md` + git log, counts fix rounds, identifies completed stage
2. **Identify Problems** -- analyzes signals (multiple validation rounds, review fix rounds, scope changes, blocked by missing info, wrong technical choice, repeated patterns). Extracts: what happened, root cause, resolution, target skill.
3. **Write Lessons** -- creates/appends entries in `skills/{skill}/references/lessons-learned.md`. Format: date, feature, Problem/Cause/Solution/Rule. Links lessons-learned.md from target skill's SKILL.md. Propagates to shim entry-points.
4. **Report** -- summary table (Skill | Shim | Problem | Rule), next step block.

### Key Differences for design-retrospective

| Aspect | retrospective | design-retrospective |
|--------|--------------|---------------------|
| Input | decisions.md + git log | design session feedback + generated files + user corrections |
| Signal types | Code problems (fix rounds, validation, scope changes) | Aesthetic problems (color corrections, typography changes, layout reworks, repeated style feedback) |
| Output location | `skills/{skill}/references/lessons-learned.md` | `.design-system/lessons-learned.md` in project |
| Promotion rule | N/A | Lesson repeated 3+ times -> promote to `.design-system/design-principles.md` |
| Target skill mapping | tech-spec-planning, code-writing, etc. | design-system-init, design-generate, design-review |

### Structural Elements to Reuse

- 4-phase structure (Collect -> Identify -> Write -> Report)
- Lesson entry format (Problem/Cause/Solution/Rule)
- Self-verification checklist
- Checkpoint after each phase
- Summary table at end

### What design-retrospective Must Add

- Aesthetic signal detection (not just code problems): "user asked to change color 3 times", "spacing was adjusted from scan", "font pair was rejected"
- Promotion mechanism: count occurrences of similar lessons, auto-promote to design-principles.md when count >= 3
- Session prompt generation for next design session (from `shared/work-templates/session-prompt.md.template` or custom)

## 13. Prepared Design References Analysis

### color-psychology.md (324 lines)

**Location:** `work/design-pipeline/references/color-psychology.md`

Two parts:
1. **Base Color Profiles** (10 colors): white, black, red, orange, yellow, green, blue, purple, brown, gray. Each has: psychological nature, what it conveys, nuance (specific hex examples showing warm/cool/muted variants), contexts.
2. **Emotional Combination Map** (10 combinations): black+gold (power), white+one accent (confidence), navy+warm white+copper (trust+character), red+black (tension), dusty rose+beige+terracotta (care), natural green+cream+ochre (organic), neon+black (rebellion), dusty blue+faded red+milk white (history), ultraviolet+silver+white (future), ochre+soot+raw white (craft).

Final map table: emotion -> combination (10 entries).

**Deployment target:** `skills/design-system-init/references/color-psychology.md` (used during interview for palette suggestion).

### color-principles.md (187 lines)

**Location:** `work/design-pipeline/references/color-principles.md`

10 professional color combination principles, each rated by complexity (2-3 stars):
1. Temperature contrast (warm/cool, not just light/dark)
2. 60-30-10 with dominance inversion (dark mode done right)
3. Split complementary (tension without aggression)
4. Tonal monochrome with one "traitor" (single accent on monochrome)
5. Optical blending through proximity (pointillism patterns, CSS example)
6. Grege principle ("dirty" colors for materiality)
7. Light source as palette principle (tinting all colors with one source)
8. Asymmetric saturation (varying saturation for focus)
9. Historical colorism (era-specific palettes: Bauhaus, midcentury, brutalism, Memphis)
10. Simultaneous contrast (same color looks different on different backgrounds)

Each has: concept, how to implement (with hex values and CSS), nuance, effect.

**Deployment target:** shared reference for `design-system-init` (palette construction) and `design-review` (palette validation). Could go to `skills/design-system-init/references/color-principles.md` with design-review loading it separately when needed.

### grid-techniques.md (228 lines)

**Location:** `work/design-pipeline/references/grid-techniques.md`

10 advanced grid/layout techniques, rated by complexity:
1. Golden ratio column system (phi-based division, recursive)
2. Fibonacci sequence grid (3-5-8-13-21-34 as spacing multipliers)
3. Broken grid (intentional rule-breaking, CSS negative margins)
4. Van de Graaf (medieval book margins, 1:1.5:2:3 ratio, content = 4/9 page)
5. Ratio grid (aspect-ratio based: 16:9, 4:3, 1:1, 3:4)
6. Optical margin alignment (visual alignment vs mathematical)
7. Diagonal layout (clip-path polygon for angled sections)
8. Typographic grid (all dimensions derived from line-height)
9. Bento grid with broken symmetry (Apple-style, asymmetric)
10. Swiss grid with color fields (12-column strict, full-bleed backgrounds)

Each has: concept, CSS implementation, where it works, effect.

**Deployment target:** `skills/design-generate/references/grid-techniques.md` (layout selection beyond 5 basic patterns). Also useful for `design-system-init` when choosing composition grid during interview.

## 14. Interview Template Format for Design

### Existing Templates Structure

**feature.yml** (300 lines): 4 phases, 23 scored items across 3 interview phases + completion. Conversation history with timestamps. Decision rules with thresholds (85% required, 50% optional). Question strategy: 3-4 per batch, propose solutions, challenge with substance.

**skill.yml** (135 lines): 3 phases, 10 items. Lower threshold (70%). Simpler structure.

### Design Interview Template Needs

For `design-system-init`, the interview covers design-specific topics not present in either existing template. A new `shared/interview-templates/design.yml` is NOT strictly needed -- the interview logic can be embedded directly in the `design-system-init` SKILL.md (like like-figma does). Interview templates are used by `user-spec-planning` which follows a generic interview loop; `design-system-init` has a domain-specific interview flow.

**Decision:** No interview template file needed. The interview structure lives in `design-system-init/SKILL.md` Phase 1, enriched with references to color-psychology.md and font-pairing.md for guidance during specific topics.

## 15. Shim Skill Pattern

### new-user-spec/SKILL.md (10 lines)

```yaml
---
description: Create user specification through adaptive interview
---
```

Body: "Read and follow `$AGENTS_HOME/skills/user-spec-planning/SKILL.md`" + warning not to look for helper scripts.

### do-feature/SKILL.md (20 lines)

```yaml
---
description: |
  Execute feature by orchestrating worker/reviewer waves.
  Use when: "выполни фичу", "do feature", "execute feature", "запусти фичу"
---
```

Body: "Read and follow `$AGENTS_HOME/skills/feature-execution/SKILL.md`" + pre-flight verification list (tech-spec approved, tasks exist, session-plan approved).

### Pattern Summary

Shim = frontmatter (name optional, description required) + redirect instruction + optional pre-flight checks. Total: 10-20 lines. The real skill has the full procedural content.

**Application to design-pipeline:** Since the MVP has 4 standalone skills (not shims wrapping deeper skills), shim pattern is NOT needed for MVP. Each design-* skill is its own canonical skill. Shims would only be needed if we later create `/design` as umbrella command routing to sub-skills.

## 16. AGENTS.md Global Rules Affecting Implementation

**File:** `AGENTS.md` (116 lines)

Key rules that constrain design-pipeline skill authoring:

1. **Instruction tone:** Every SHOULD is a MUST. "Consider" means "do it". "If needed" means "always". Skills must be written accordingly.
2. **No scripts:** Skills are SKILL.md files only. No dispatch scripts, no init scripts.
3. **Integrity check:** Before executing any SKILL.md, verify no git conflict markers.
4. **rg fallback:** If ripgrep blocked, fall back to PowerShell/findstr. Design skills doing project scans must handle this.
5. **npm install timeout:** Use `timeout_ms: 300000` for heavy packages. Not directly relevant to design skills.
6. **git add -f for logs/:** `work/{feature}/logs/` files need force-add due to gitignore.
7. **Session End Protocol:** After completing ANY pipeline step -- mandatory ending block with completion summary, next step, and self-contained prompt for next session.
8. **Intermediate summaries:** After each phase checkpoint within a skill, show progress summary.
9. **decisions.md updates:** After every significant decision during any step.
10. **Pipeline navigation:** Always tell user the next step after completing any step.
11. **Framework Update Protocol:** Minimal scope, context anchor, re-read only AGENTS.md.

**Impact on design skills:**
- Each design-* skill must end with Session End Protocol block
- Each phase checkpoint must show intermediate summary
- design-system-init interview must show summary after every 3-4 topics (matching like-figma pattern AND AGENTS.md requirement)
- design-review when called from do-task does NOT need Session End Protocol (it's a sub-step, not a pipeline step)

## 17. decisions.md Status

**File:** `work/design-pipeline/decisions.md`

Currently empty (template only, no task entries yet). Template format documented: Task N title, Status, Commit, Agent, Summary, Deviations, Reviews (rounds with file links), Verification. Entries added by agents as tasks complete.

## 18. Reference Deployment Plan

Summary of where each reference file ends up in the final skill structure:

| Source | Target Skill | Target Path | Modification |
|--------|-------------|-------------|-------------|
| `like-figma/references/design-tokens.md` | design-system-init | `references/design-tokens.md` | None (copy as-is) |
| `like-figma/references/component-patterns.md` | design-generate | `references/component-patterns.md` | None (copy as-is) |
| `like-figma/references/generation-guide.md` | design-generate | `references/generation-guide.md` | Enrich with grid-techniques patterns |
| `like-figma/assets/preview-template.html` | design-generate | `assets/preview-template.html` | None (copy as-is) |
| `work/design-pipeline/references/color-psychology.md` | design-system-init | `references/color-psychology.md` | None (move as-is) |
| `work/design-pipeline/references/color-principles.md` | design-system-init | `references/color-principles.md` | None (move as-is) |
| `work/design-pipeline/references/grid-techniques.md` | design-generate | `references/grid-techniques.md` | None (move as-is) |
| TODO: create font-pairing.md | design-system-init | `references/font-pairing.md` | New file to create |

### Cross-Skill Reference Access

design-review needs access to `color-principles.md` and `design-tokens.md` for validation, but these live in other skills. Options:
1. **Duplicate** -- copy to design-review/references/ (violates DRY but simple)
2. **Cross-reference** -- design-review loads from `$AGENTS_HOME/skills/design-system-init/references/color-principles.md` (fragile path)
3. **Shared references directory** -- create `shared/design-references/` alongside `shared/work-templates/` (clean, follows existing pattern)

**Recommendation for tech-spec:** Option 3 (shared directory) for references used by multiple design skills. Skill-specific references stay in the skill. Shared references: design-tokens.md, color-principles.md, color-psychology.md. Skill-specific: component-patterns.md (design-generate only), grid-techniques.md (design-generate only), font-pairing.md (design-system-init only).

## 19. Final Skill File Structure (Projected)

```
skills/
├── design-system-init/
│   ├── SKILL.md                          # ~300 lines, procedural
│   └── references/
│       └── font-pairing.md               # NEW: to create
│
├── design-generate/
│   ├── SKILL.md                          # ~250 lines, procedural
│   ├── references/
│   │   ├── component-patterns.md         # from like-figma
│   │   ├── generation-guide.md           # from like-figma, enriched
│   │   └── grid-techniques.md            # from work/design-pipeline/references/
│   └── assets/
│       └── preview-template.html         # from like-figma
│
├── design-review/
│   └── SKILL.md                          # ~150 lines, informational
│
├── design-retrospective/
│   └── SKILL.md                          # ~200 lines, procedural
│
shared/
└── design-references/                    # NEW directory
    ├── design-tokens.md                  # from like-figma (shared by init + generate + review)
    ├── color-psychology.md               # from work/design-pipeline/references/
    └── color-principles.md               # from work/design-pipeline/references/

# like-figma/ -- to be DELETED after migration complete
```

**Estimated total new lines:** ~900 lines of SKILL.md content + ~0 lines of references (all copied/moved from existing sources) + ~150 lines for font-pairing.md = ~1050 lines total new content.
