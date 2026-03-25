# Design Pipeline

Разработчику без дизайнера нужен способ делать визуально качественные интерфейсы. Design Pipeline — это 4 скилла для Claude Code, которые превращают агента в стилиста: он подбирает палитры по настроению проекта, сочетает шрифты, выстраивает визуальную согласованность между экранами. Всё без Figma — дизайн-система хранится как код прямо в проекте.

## Как это работает

Агент проводит интервью о проекте: какое настроение, какая аудитория, какие компоненты нужны. На основе ответов генерирует `tokens.json` — единый источник правды для цветов, типографики, отступов и теней. Дальше по запросу собирает HTML-макеты из 15 паттернов лейаутов, ревьюит UI-код на соответствие токенам, а после сессии проводит ретроспективу и накапливает "вкус" проекта.

## Что умеет

- **Создать дизайн-систему через интервью** — агент сканирует проект (Tailwind, CSS vars, SCSS), задаёт 6 тем (настроение, палитра, типографика, spacing, компоненты), генерирует tokens.json + готовые HTML-компоненты с CSS custom properties
- **Сгенерировать макет страницы по описанию** — 15 паттернов лейаутов (от простого sidebar до Swiss Grid и Golden Ratio), итерации по скриншотам, диагональный коллаж "до/после" при утверждении финального варианта
- **Проверить UI-код на соответствие дизайн-системе** — находит hardcoded цвета и spacing, предлагает конкретные токены на замену, встроен в пайплайн выполнения задач
- **Провести дизайн-ретроспективу** — анализирует фидбек за сессию, пишет уроки в проект, промоутит повторяющиеся паттерны в принципы дизайна

## Фишка

Палитра подбирается не по цветовому кругу, а через психологию цвета: агент знает, что синий транслирует доверие для финтеха, а оранжевый — энергию для фитнес-приложений. Справочник на 323 строки покрывает эмоциональные ассоциации, культурные контексты и отраслевые конвенции. Второй справочник — 10 проверенных пар Google Fonts с готовыми весами и размерами для tokens.json.

## Чем интересен

Это не генератор шаблонов. Агент выстраивает дизайн-систему под конкретный проект, а потом сам себя проверяет — design-review автоматически включается при каждой задаче с UI-файлами. Ретроспектива накапливает "вкус": что сработало, что нет, какие принципы закрепить. С каждой сессией агент лучше понимает эстетику проекта.

## Масштаб

- Кодовая база: ~2300 строк (4 скилла + 9 справочников)
- 13 файлов, 12 задач в 5 волнах, 3 сессии разработки
- Стек: Markdown skills, HTML/CSS, SVG, CSS custom properties

## Что дальше

**v1 — углубление обучения:**
- Профиль вкуса проекта — агент накапливает предпочтения из референсов и фидбека
- Автоматический промоушен уроков в принципы дизайна (урок повторился 3+ раз → правило)
- Интеграция с code-writing — при генерации UI-кода агент сразу применяет токены из дизайн-системы

**v2 — полный пайплайн:**
- 5 новых скиллов: design-spec → design-plan → design-decompose → design-execute → design-done
- Полный цикл как для кода: от спецификации до финализации
- Автоматическая генерация макетов при создании новых экранов
- Поддержка не-веб проектов

---

# Design Pipeline (EN)

A developer without a designer needs a way to build visually polished interfaces. Design Pipeline is a set of 4 skills for Claude Code that turn the agent into a stylist: it selects palettes based on project mood, pairs fonts, and maintains visual consistency across screens. No Figma needed — the design system lives as code inside the project.

## How it works

The agent interviews about the project: what mood, what audience, what components are needed. Based on answers, it generates `tokens.json` — a single source of truth for colors, typography, spacing, and shadows. Then on demand it assembles HTML mockups from 15 layout patterns, reviews UI code against tokens, and after each session runs a retrospective to accumulate the project's "taste."

## What it does

- **Create a design system through interview** — the agent scans the project (Tailwind, CSS vars, SCSS), covers 6 topics (mood, palette, typography, spacing, components), generates tokens.json + ready HTML components with CSS custom properties
- **Generate page mockups from text descriptions** — 15 layout patterns (from simple sidebar to Swiss Grid and Golden Ratio), screenshot-based iteration, diagonal before/after collage when the final variant is approved
- **Review UI code against the design system** — finds hardcoded colors and spacing, suggests specific token replacements, integrated into the task execution pipeline
- **Run design retrospectives** — analyzes session feedback, writes lessons to the project, promotes recurring patterns into design principles

## The thing

Palette selection is driven by color psychology, not the color wheel: the agent knows blue signals trust for fintech and orange conveys energy for fitness apps. A 323-line reference covers emotional associations, cultural contexts, and industry conventions. A second reference provides 10 vetted Google Fonts pairs with ready weights and sizes for tokens.json.

## What makes it interesting

This isn't a template generator. The agent builds a design system tailored to the specific project, then checks its own work — design-review activates automatically on every task with UI files. Retrospectives accumulate "taste": what worked, what didn't, which principles to keep. With each session the agent better understands the project's aesthetics.

## Scale

- Codebase: ~2,300 lines (4 skills + 9 references)
- 13 files, 12 tasks in 5 waves, 3 development sessions
- Stack: Markdown skills, HTML/CSS, SVG, CSS custom properties

## What's next

**v1 — deeper learning:**
- Project taste profile — the agent accumulates preferences from user references and feedback
- Auto-promotion of lessons to design principles (lesson repeats 3+ times → becomes a rule)
- Integration with code-writing — when generating UI code, the agent applies design system tokens automatically

**v2 — full pipeline:**
- 5 new skills: design-spec → design-plan → design-decompose → design-execute → design-done
- Full cycle mirroring the dev methodology: from specification to finalization
- Automatic mockup generation when creating new screens
- Non-web project support

---

Design Pipeline — 4 скилла для Claude Code, которые превращают агента в дизайн-стилиста.

Агент проводит интервью о проекте и создаёт дизайн-систему (tokens.json + HTML-компоненты). Потом генерирует макеты из 15 паттернов лейаутов, ревьюит UI-код на соответствие токенам и проводит ретроспективы, накапливая "вкус" проекта.

Фишка: палитра подбирается через психологию цвета — агент знает, какой цвет транслирует нужную эмоцию для конкретной отрасли. А при утверждении макета автоматически собирается диагональный коллаж "до/после".

Дальше: профиль вкуса, авто-промоушен уроков в принципы, а в v2 — полный дизайн-пайплайн из 9 скиллов по аналогии с dev-методологией.

~2300 строк | Стек: Markdown skills, HTML/CSS, SVG, CSS custom properties

---

Design Pipeline — 4 skills for Claude Code that turn the agent into a design stylist.

The agent interviews about the project and creates a design system (tokens.json + HTML components). Then generates mockups from 15 layout patterns, reviews UI code against tokens, and runs retrospectives, accumulating the project's "taste."

The thing: palette selection is driven by color psychology — the agent knows which color conveys the right emotion for a specific industry. And when a mockup is approved, a diagonal before/after collage is generated automatically.

What's next: taste profiles, auto-promotion of lessons to principles, and in v2 — a full design pipeline of 9 skills mirroring the dev methodology.

~2,300 lines | Stack: Markdown skills, HTML/CSS, SVG, CSS custom properties

---
Design Pipeline MVP
