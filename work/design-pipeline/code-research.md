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
