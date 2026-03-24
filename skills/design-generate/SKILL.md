---
name: design-generate
description: |
  Generates HTML/CSS pages and static SVG mockups from text descriptions using
  a project's design system. Selects from 15 layout patterns (5 basic + 10 advanced
  grids), assembles components, applies tokens, supports iteration via screenshots.

  Use when: "сделай макет", "generate design", "сгенерируй страницу", "create mockup",
  "макет страницы", "generate a page", "собери страницу", "покажи страницу",
  "design page", "сверстай макет"
---

# Design Generate

Generate HTML+SVG page mockups from text descriptions using the project's design system.

```
User description → Parse → Select Layout → Assemble Components → HTML + SVG
                                                                      ↓
                                                              User feedback (screenshot)
                                                                      ↓
                                                                  Iterate
```

## Phase 0: Readiness Check

1. Check that `.design-system/tokens.json` exists in the project root
   - If missing → stop and tell the user: "Design system not found. Run `/design-system-init` to create one."
   - If the file exists, read it and validate that it is valid JSON
   - If JSON is malformed → stop and tell the user: "tokens.json is corrupted. Run `/design-system-init` to recreate the design system."

2. Parse tokens into CSS custom properties for use in generated pages (following the mapping pattern from [design-tokens.md](../../shared/design-references/design-tokens.md))

**Checkpoint:** tokens.json exists, parses as valid JSON, contains colors/typography/spacing sections.

## Phase 1: Parse Request

1. Identify from the user's description:
   - **Page type** — what kind of page (login, dashboard, landing, catalog, settings, etc.)
   - **Components needed** — buttons, cards, inputs, tables, navigation, etc.
   - **Layout structure** — sidebar? grid? hero section? split view?
   - **Content** — real content provided, or use realistic placeholders?

2. If the description is ambiguous, propose a specific interpretation:
   - "I'll create a login page with Split Screen layout: brand image on the left, login form (email + password + submit button) on the right. Sound good?"
   - Wait for confirmation before proceeding

3. List components that are in `.design-system/components/`. For missing ones:
   - Simple components (badge, alert, divider) → create on the fly following patterns from [component-patterns.md](references/component-patterns.md)
   - Complex components (carousel, data-table, calendar) → inform the user this needs a separate session via `/design-system-init` to add the component properly

**Checkpoint:** page type identified, component list ready, layout direction chosen, ambiguities resolved.

## Phase 2: Select Layout

1. Choose a layout from 15 available patterns using the selection table in [generation-guide.md](references/generation-guide.md) (layout patterns, selection criteria, CSS implementation)

2. For **basic layouts** (Single Column, Sidebar+Content, Grid, Hero+Sections, Split Screen):
   - Use the built-in CSS from generation-guide.md directly

3. For **advanced layouts** (Golden Ratio, Fibonacci, Broken Grid, Van de Graaf, Ratio Grid, Optical Margin, Diagonal, Typographic Grid, Bento, Swiss Grid):
   - Apply the CSS technique from [grid-techniques.md](references/grid-techniques.md) for the chosen pattern
   - Advanced layouts are appropriate when the user asks for visual impact, editorial feel, or premium aesthetics

4. Present the chosen layout to the user:
   - "For your login page, I'll use Split Screen layout — brand visual on the left half, form on the right."
   - If the user disagrees → select a different pattern

**Checkpoint:** layout pattern selected and confirmed, CSS approach determined.

## Phase 3: Assemble & Generate

### 3.1 Assemble page

1. Build the page structure using the selected layout CSS
2. Place components into layout regions following patterns from [component-patterns.md](references/component-patterns.md) (BEM naming, variant system, slot pattern)
3. Inject design tokens as CSS custom properties in `:root`
4. Add content — real if provided, realistic placeholders otherwise (per generation-guide.md placeholder rules: diverse names, meaningful text, colored rectangles for images, realistic numbers)

### 3.2 Generate HTML

1. Use [preview-template.html](assets/preview-template.html) as the base template
2. Replace `{{PAGE_TITLE}}` with the page name
3. Replace `{{PAGE_STYLES}}` with component styles + layout CSS
4. Replace `{{PAGE_CONTENT}}` with assembled page HTML
5. Replace placeholder tokens in `:root` with actual values from tokens.json

### 3.3 Generate SVG

1. Convert the same layout to static SVG (per generation-guide.md SVG rules)
2. Desktop viewport: `viewBox="0 0 1440 900"`
3. Mobile viewport (if requested): `viewBox="0 0 390 844"`
4. Convert layouts to absolute-positioned shapes, embed colors directly (no CSS variables in SVG)

### 3.4 Save files

1. Validate file names: only lowercase letters, digits, and hyphens allowed (`/^[a-z0-9-]+$/`). Reason: prevents path traversal when writing to `.design-system/pages/`
2. Save HTML to `.design-system/pages/{name}.html`
3. Save SVG to `.design-system/pages/{name}.svg`
4. If device variants requested: `{name}-mobile.html`, `{name}-desktop.html`

**Checkpoint:** HTML file uses template + tokens (no hardcoded values), SVG renders the same layout, file names validated, files saved to `.design-system/pages/`.

## Phase 4: Present & Iterate

### 4.1 Present result

1. Tell the user where files were saved
2. Suggest opening the HTML file in a browser: "Open `.design-system/pages/{name}.html` in your browser to see the interactive preview."
3. Ask: "Send a screenshot if you'd like adjustments, or describe what to change."

### 4.2 Iterate on feedback

When the user provides feedback (screenshot or text):
- "Bigger" / "smaller" → adjust spacing and font sizes
- "More contrast" → darken text, increase color differences
- "Simpler" → reduce components, increase whitespace
- "More detail" → add secondary info, icons, metadata
- Specific feedback → apply the requested change

Regenerate both HTML and SVG after each iteration.

### 4.3 Handle context exhaustion

If the conversation context is running low and further iterations are needed:
1. Save the current state of all generated files
2. Generate a continuation prompt the user can paste into a new session:

```
Continue design iteration for `.design-system/pages/{name}.html`.
Design system: `.design-system/tokens.json`.
Layout: {layout pattern used}.
Last feedback: "{user's last feedback}".
Open the HTML file, apply the feedback, regenerate HTML+SVG.
```

3. Tell the user: "Context is getting long. Start a new session with `/design-generate` and paste this prompt: {prompt above}"

**Checkpoint:** result presented, user feedback addressed (or continuation prompt generated).

## Final Check

Before finishing, verify:
- [ ] All generated HTML files use design tokens from tokens.json (no hardcoded color/spacing values)
- [ ] HTML files use preview-template.html as base (includes CSP meta tag)
- [ ] SVG files render the same layout as HTML with embedded colors
- [ ] File names match `/^[a-z0-9-]+$/` pattern
- [ ] `.design-system/pages/` directory contains the generated files
- [ ] User has been shown where to find the files
