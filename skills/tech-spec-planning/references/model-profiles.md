# Codex Model Profiles

Stable model tiers for this methodology. Use tiers instead of legacy `opus/sonnet/haiku` labels.

## Tier Mapping

| Tier | Primary | Fallback | Typical use |
|------|---------|----------|-------------|
| `tier_opus` | `gpt-5.4` | `gpt-5.3-codex` | Complex architecture, hard debugging, security-critical decisions |
| `tier_sonnet` | `gpt-5.4-mini` | `gpt-5.3-codex` | Most reviewers/validators, medium complexity execution |
| `tier_haiku` | `gpt-5.4-mini` + `reasoning_effort: low` | `gpt-5.1-codex-max` | Cheap routine tasks, formatting, simple checks |

## Reasoning Defaults

- `tier_opus`: `high` (or `xhigh` for hardest tasks)
- `tier_sonnet`: `medium`
- `tier_haiku`: `low`

## Selection Rules

1. If task can break production or security boundaries -> `tier_opus`.
2. If task is a reviewer/validator or normal implementation helper -> `tier_sonnet`.
3. If task is deterministic and low-risk -> `tier_haiku`.
4. If primary model fails/unavailable -> retry once with fallback before escalating.
