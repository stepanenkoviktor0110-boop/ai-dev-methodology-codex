# Codex Model Profiles

Stable model tiers for this methodology. Tier names are model-agnostic — they map to capability levels, not specific vendors.

## Tier Mapping

| Tier | Primary | Fallback | Typical use |
|------|---------|----------|-------------|
| `tier_high` | `gpt-5.4` | `gpt-5.3-codex` | Complex architecture, hard debugging, security-critical decisions |
| `tier_medium` | `gpt-5.4-mini` | `gpt-5.3-codex` | Most reviewers/validators, medium complexity execution |
| `tier_low` | `gpt-5.4-mini` + `reasoning_effort: low` | `gpt-5.1-codex-max` | Cheap routine tasks, formatting, simple checks |

## Reasoning Defaults

- `tier_high`: `high` (or `xhigh` for hardest tasks)
- `tier_medium`: `medium`
- `tier_low`: `low`

## Selection Rules

1. If task can break production or security boundaries -> `tier_high`.
2. If task is a reviewer/validator or normal implementation helper -> `tier_medium`.
3. If task is deterministic and low-risk -> `tier_low`.
4. If primary model fails/unavailable -> retry once with fallback before escalating.
