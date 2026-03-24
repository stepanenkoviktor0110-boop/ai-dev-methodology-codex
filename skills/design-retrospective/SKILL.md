---
name: design-retrospective
description: |
  Analyze design session feedback, extract aesthetic lessons into
  .design-system/lessons-learned.md, promote recurring patterns to
  design-principles.md, generate context-free next-session prompt.

  Use when: "дизайн ретроспектива", "design retrospective",
  "уроки дизайна", "что улучшить в дизайне", "design lessons learned",
  "ретроспектива дизайн-сессии", "design feedback analysis"
---

# Design Retrospective

Evaluate the completed design session, extract aesthetic lessons from user feedback and corrections, embed them into the project's design system knowledge.

**Input:** session history (user feedback, corrections), `.design-system/` files
**Output:** entries in `.design-system/lessons-learned.md`, promotions to `.design-system/design-principles.md`, next-session prompt
**Language:** lessons and communication in Russian

## Phase 1: Collect Evidence

1. Check that `.design-system/` directory exists in the project.
   - If missing — stop and suggest: "Директория `.design-system/` не найдена. Запустите `/design-system-init` для создания дизайн-системы."
2. Ask user which design session to analyze (if not obvious from conversation context).
3. Read `.design-system/` structure: `tokens.json`, `components/`, `pages/` — note what was created or modified during the session.
4. Collect user feedback from the session:
   - Color corrections requested ("слишком яркий", "сделай теплее")
   - Font/typography changes ("шрифт не подходит", "заголовок крупнее")
   - Spacing adjustments ("слишком тесно", "больше воздуха")
   - Layout reworks ("переделай структуру", "поменяй расположение")
   - Component modifications ("кнопка не та", "карточка слишком широкая")
   - Elements rejected entirely ("убери это", "не нужен этот блок")
5. List all files changed during the session (components, pages, tokens).

**Checkpoint:** `.design-system/` confirmed, session feedback collected, changed files listed.

## Phase 2: Identify Patterns

1. Categorize collected corrections into types:

| Category | Examples |
|----------|----------|
| Color corrections | Palette too cold/warm, contrast issues, accent color wrong |
| Typography changes | Wrong font pair, sizes off, weight mismatch |
| Spacing adjustments | Too tight/loose, inconsistent gaps, padding issues |
| Layout reworks | Structure changed, grid reorganized, section order swapped |
| Component modifications | Shape/size/style of specific components adjusted |

2. Look for repetitions within the session — same type of correction appearing 2+ times signals a pattern.
3. Look for repetitions across sessions — compare with existing `.design-system/lessons-learned.md` entries (if file exists).
4. If no corrections were made and no patterns detected — report to user: "Сессия прошла без коррекций, паттернов не выявлено." Skip to Phase 5.

**Checkpoint:** corrections categorized, patterns identified (or confirmed absent).

## Phase 3: Write Lessons

1. Check if `.design-system/lessons-learned.md` exists.
   - If no — create with header:
     ```markdown
     # Design Lessons Learned

     Accumulated lessons from design sessions. Each entry records a problem,
     its cause, how it was resolved, and the rule to follow in the future.
     ```
   - If yes — read existing content.

2. For each identified pattern, check for duplicates:
   - Read existing entries in `lessons-learned.md`.
   - If the same problem is already recorded (matching root cause) — skip, do not duplicate.

3. Append new entries in this format:

```markdown
### {YYYY-MM-DD}: {pattern title}

**Problem:** {what went wrong — 1 sentence}
**Cause:** {why it happened — 1 sentence}
**Solution:** {how it was fixed — 1 sentence}
**Rule:** {what to do in the future — actionable instruction}
```

Example entry:
```markdown
### 2026-03-15: Слишком холодная палитра для уютного бренда

**Problem:** Сгенерированная палитра воспринималась холодной и отстранённой для сайта спа-салона.
**Cause:** Выбраны чистые синие оттенки без тёплого подтона — не соответствует настроению "уют, расслабление".
**Solution:** Заменены на приглушённые тёплые оттенки (sage, warm beige, muted terracotta).
**Rule:** Для "уютных" и "расслабляющих" проектов использовать приглушённые тёплые тона, избегать чистых холодных оттенков.
```

**Checkpoint:** lessons written to `.design-system/lessons-learned.md`, duplicates skipped.

## Phase 4: Promote Principles

1. Read all entries in `.design-system/lessons-learned.md`.
2. Group entries by similar rules — same root cause type or same correction category.
3. Count occurrences in each group. If a rule appears 3+ times across all entries (not per session) — it qualifies for promotion.
4. Check if `.design-system/design-principles.md` exists.
   - If no — create with header:
     ```markdown
     # Design Principles

     Project-specific aesthetic rules derived from accumulated feedback.
     Each principle emerged from 3+ repeated lessons and reflects the
     project's taste profile.
     ```
   - If yes — read existing content.

5. For each qualifying rule:
   - Check if already present in `design-principles.md` — skip if exists.
   - Add new principle:
     ```markdown
     ## {principle title}

     **Rule:** {actionable instruction}
     **Rationale:** {why — derived from N lessons, dates: ...}
     **Category:** {color | typography | spacing | layout | component}
     ```

6. If no rules qualify for promotion — report: "Нет правил для промоушена (ни одно не появилось 3+ раз)."

**Checkpoint:** qualifying rules promoted to `design-principles.md`, or reported that none qualify.

## Phase 5: Generate Next-Session Prompt

Generate a self-contained prompt for continuing design work in a new session. The prompt requires zero prior conversation context to be useful.

Prompt structure:
```
## Контекст проекта
{Project name, purpose, target audience — from tokens.json metadata or session context}

## Текущее состояние дизайн-системы
{List of tokens.json sections: palette mood, font pair, spacing scale}
{List of existing components and pages}

## Накопленные принципы
{List rules from design-principles.md, or "Принципов пока нет" if empty/missing}

## Что было сделано в последней сессии
{Summary of session work: what was created/modified, key decisions}

## Что делать дальше
{Suggested next steps: new pages, component refinements, areas needing attention}

## Как запустить
{Concrete command: /design-generate for new pages, /design-system-init for token updates}
```

Present the prompt to user in a code block for easy copy-paste.

**Checkpoint:** next-session prompt generated and shown to user.

## Self-Verification

Before finishing, verify:
- [ ] `.design-system/` existence checked (refused if missing)
- [ ] Session feedback collected and categorized
- [ ] Each lesson has all 4 fields (Problem, Cause, Solution, Rule)
- [ ] No duplicate lessons added
- [ ] Rules are actionable (not vague advice like "делать лучше")
- [ ] Promotion count is across all entries, not per session
- [ ] Already-promoted rules not duplicated in `design-principles.md`
- [ ] Next-session prompt is self-contained (no references to "this conversation")
- [ ] All file writes target `.design-system/` in the project (not skill directory)
