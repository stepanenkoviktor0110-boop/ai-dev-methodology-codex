# Optional Frontmatter Fields

These fields are NOT required for most skills. Use only when needed.

## Field Reference

| Field | Default | Description |
|-------|---------|-------------|
| `argument-hint` | None | Autocomplete hint shown after skill name |
| `disable-model-invocation` | `false` | If `true`, skill only triggers manually via `/skill-name` |
| `user-invocable` | `true` | If `false`, skill hidden from `/` menu, only Codex can invoke |
| `model` | `inherit` | Override model: `gpt-5.4`, `gpt-5.4-mini`, `gpt-5.3-codex`, `gpt-5.1-codex-max`, `inherit` |

## When to Use Each Field

### argument-hint

Shows hint in autocomplete to guide user input.

```yaml
---
name: fix-issue
argument-hint: "[issue-number]"
---
```

User sees: `/fix-issue [issue-number]`

### disable-model-invocation

Prevents Codex from auto-triggering the skill. Only manual `/skill-name` works.

```yaml
---
name: dangerous-operation
disable-model-invocation: true
---
```

Use for: destructive operations, expensive API calls, operations requiring explicit user consent.

### user-invocable

Hides skill from `/` menu. Only Codex can invoke it programmatically.

```yaml
---
name: internal-helper
user-invocable: false
---
```

Use for: helper skills that shouldn't appear in user-facing menu, internal utilities.

### model

Overrides the model used for this skill.

```yaml
---
name: quick-lookup
model: gpt-5.1-codex-max
---
```

Options:
- `inherit` — use orchestrator's model (default, recommended)
- `gpt-5.4-mini` — fast, good for most tasks
- `gpt-5.4` — best quality, use for complex reasoning
- `gpt-5.3-codex` — strong code generation, fallback for opus
- `gpt-5.1-codex-max` — budget option for simple lookups

Recommended profile mapping:
- `tier_high`: `gpt-5.4` (fallback `gpt-5.3-codex`)
- `tier_medium`: `gpt-5.4-mini` (fallback `gpt-5.3-codex`)
- `tier_low`: `gpt-5.4-mini` with low reasoning (fallback `gpt-5.1-codex-max`)

Use sparingly. `inherit` is usually best.
