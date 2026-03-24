---
name: like-figma
description: |
  Generates visual designs (HTML/CSS, SVG) from text descriptions using a project's
  design system. Creates and maintains design systems through interview. Converts existing
  code into visual previews. Works without Figma — everything stays as code.

  Use when: "сгенерируй дизайн", "сделай макет", "покажи как выглядит",
  "создай дизайн-систему", "generate design", "create mockup", "design system",
  "визуализируй компонент", "code to design", "превью страницы"
---

# LikeFigma

Generate designs from descriptions and code. Design system as source of truth. Output: HTML/CSS + SVG. No Figma API needed.

## How It Works

```
Design System (tokens + components)
        │
        ├── Text description ──→ HTML/CSS page / SVG mockup
        ├── Existing code     ──→ Visual preview
        └── Component name    ──→ Component showcase
```

## Design System Location

```
.design-system/
├── tokens.json              — colors, typography, spacing, shadows, radii
├── components/              — component definitions (HTML/CSS templates)
│   ├── button.html
│   ├── card.html
│   ├── input.html
│   └── ...
├── pages/                   — generated page designs
│   ├── *.html               — interactive HTML/CSS previews
│   └── *.svg                — static SVG exports
└── README.md                — design system overview (auto-generated)
```

## Phase 0: Project Readiness

Before any work, verify the skill is accessible from the current project:

1. Check if the project has `.agents/skills/like-figma/` or a reference to it in `AGENTS.md`
2. If not — ask the user which approach they prefer:
   - **Copy**: copy the skill into the project's `.agents/skills/like-figma/`
   - **Reference**: add a path reference in the project's `AGENTS.md`
3. Apply the chosen approach, then proceed

Skip this step if the skill is already accessible (e.g., invoked via global skills directory).

## Mode Detection

Determine the mode from user's request:

**Mode A: Init** — design system does not exist yet, or user asks to create/recreate it.
→ Go to Phase 1.

**Mode B: Generate** — design system exists, user describes a page/screen/component to create.
→ Go to Phase 3.

**Mode C: Code-to-Design** — user points at existing code and wants a visual preview.
→ Go to Phase 4.

**Mode D: Evolve** — user wants to add/change tokens or components in existing design system.
→ Go to Phase 2 (targeted update, skip interview).

## Phase 1: Design System Discovery (Interview)

Run only when `.design-system/` does not exist or user explicitly asks to create one.

### 1.1 Project Scan

Before asking anything, scan the project:
- Read CSS/SCSS/Tailwind config for existing colors, fonts, spacing
- Read component files for UI patterns already in use
- Check for existing design tokens (CSS custom properties, theme files)
- Note the tech stack (React, Vue, plain HTML, etc.)

Present findings: "I found these design patterns in your project: {list}. I'll use them as a starting point."

### 1.2 Interview

Propose-first approach — suggest concrete values based on scan, user confirms or adjusts.

**Brand & Colors:**
- Primary, secondary, accent colors (propose from scan or sensible defaults)
- Neutral palette (grays)
- Semantic colors: success, warning, error, info
- Background and surface colors

**Typography:**
- Font families (headings, body, mono)
- Size scale (propose: 12/14/16/18/20/24/32/48)
- Weight scale (regular 400, medium 500, semibold 600, bold 700)
- Line heights

**Spacing & Layout:**
- Spacing scale (propose: 4/8/12/16/24/32/48/64)
- Border radii (propose: 4/8/12/full)
- Shadows (propose: sm/md/lg)
- Breakpoints (if responsive)

**Components:**
- Which components does the project need? (propose based on scan)
- Component variants (sizes, states)
- Show a list, user picks what to include in v1

One topic at a time. After every 3-4 topics, summarize what was decided.

### 1.3 Checkpoint

Move to Phase 2 when:
- Color palette defined (primary, secondary, neutrals, semantics)
- Typography scale decided
- Spacing scale decided
- Component list approved

## Phase 2: Build Design System

### 2.1 Create Tokens

Write `tokens.json` following structure from [design-tokens.md](references/design-tokens.md) (token schema, naming conventions, CSS custom property generation).

### 2.2 Create Components

For each approved component, create an HTML file in `.design-system/components/` following patterns from [component-patterns.md](references/component-patterns.md) (component template structure, variant system, slot patterns).

Each component file:
- Self-contained HTML + inline CSS using design tokens as CSS custom properties
- Shows all variants (sizes, states, colors)
- Works standalone — open in browser to see the component

### 2.3 Generate README

Auto-generate `.design-system/README.md` with:
- Token overview (color swatches, type scale, spacing)
- Component list with descriptions
- Usage instructions

### 2.4 Checkpoint

Verify:
- `tokens.json` is valid JSON with all decided values
- Each component renders correctly with tokens
- README reflects actual contents

## Phase 3: Generate Design

User describes what they want → assemble from design system.

### 3.1 Understand Request

Parse the user's description:
- What page/screen/section is this?
- Which components are needed?
- Layout structure (sidebar? grid? stack?)
- Content (real or placeholder?)

If ambiguous — propose a specific interpretation and confirm.

### 3.2 Generate Output

Apply generation approach from [generation-guide.md](references/generation-guide.md) (layout algorithms, responsive patterns, content placement).

Generate both formats:

**HTML/CSS (interactive preview):**
- Use [preview-template.html](assets/preview-template.html) as base
- Inject design tokens as CSS custom properties
- Assemble page from components
- Add responsive behavior if relevant
- Save to `.design-system/pages/{name}.html`

**SVG (static mockup):**
- Render the same layout as SVG
- Embed fonts, flatten styles
- Save to `.design-system/pages/{name}.svg`

### 3.3 Present Result

Tell user where to find the files. Suggest opening HTML in browser for interactive preview.

Ask: "Adjust something or generate another page?"

### 3.4 Checkpoint

Verify generated files:
- HTML opens in browser without errors
- SVG renders correctly
- Both use design system tokens (no hardcoded values)
- Layout matches user's description

## Phase 4: Code-to-Design

User points at existing code → generate visual representation.

### 4.1 Read Source

Read the code files user specified. Identify:
- UI components used
- Layout structure
- Data/content displayed
- Conditional rendering branches (generate the default/primary state)

### 4.2 Map to Design System

- Match code components to design system components
- If a component doesn't exist in design system — create a close approximation
- Extract content/data for realistic preview

### 4.3 Generate Preview

Same output as Phase 3 (HTML + SVG), but based on actual code structure.

Label output clearly: "Generated from {file paths}. Shows the default render state."

### 4.4 Checkpoint

Verify the preview reasonably represents the actual code output.

## Final Check

Before finishing any mode, verify:
- [ ] All generated files use design tokens (no magic numbers)
- [ ] HTML files open in browser without errors
- [ ] SVG files render correctly
- [ ] `.design-system/` structure is consistent
- [ ] README.md reflects current state of design system
