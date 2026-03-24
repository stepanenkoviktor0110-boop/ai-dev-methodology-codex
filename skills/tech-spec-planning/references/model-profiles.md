# Codex Model Profiles

Stable model tiers for this methodology. Tier names map to capability levels. Tiers are optimized for **coding workflows** based on SWE-Bench Pro benchmarks.

**IMPORTANT:** Only models available in Codex CLI are used. Check `/model` in Codex to see your available models.

## Tier Mapping

| Tier | Primary | Fallback | Typical use | Benchmark / Rationale |
|------|---------|----------|-------------|----------------------|
| `tier_high` | `gpt-5.4` | `gpt-5.3-codex` | Workers, complex architecture, hard debugging, security-critical decisions | SWE-Bench Pro 57.7% — flagship, best multi-file reasoning. Comparable to Anthropic Opus |
| `tier_medium` | `gpt-5.3-codex` | `gpt-5.4-mini` | Reviewers, validators, medium complexity execution | SWE-Bench Pro ~56% — purpose-built for agentic coding, supports reasoning levels (low/medium/high/xhigh). Comparable to Anthropic Sonnet |
| `tier_low` | `gpt-5.4-mini` | `gpt-5.1-codex-mini` | Simple checks, formatting, deterministic routine tasks | SWE-Bench Pro 54.4% — fast, low cost. Use with `reasoning_effort: low` for cheapest option. Comparable to Anthropic Haiku |

**Why `gpt-5.3-codex` for tier_medium instead of `gpt-5.4-mini`:**
- SWE-Bench Pro: ~56% vs 54.4% — codex wins on coding tasks
- Purpose-built for agentic coding (the exact use case of this framework)
- Supports reasoning effort levels for fine-tuning cost/quality tradeoff
- `gpt-5.4-mini` is the fallback and also serves as tier_low primary

**Note:** `gpt-5.4-nano` exists in the API but is NOT available in Codex CLI. Do not use it.

## Reasoning Defaults

- `tier_high`: `high` (or `xhigh` for hardest tasks)
- `tier_medium`: `medium`
- `tier_low`: `low`

## Selection Rules

1. If task can break production or security boundaries → `tier_high`.
2. If task is a reviewer/validator or normal implementation helper → `tier_medium`.
3. If task is deterministic and low-risk → `tier_low`.
4. If primary model fails/unavailable → retry once with fallback before escalating.

## Verification

To confirm models actually switch when spawning subagents:
- Check OpenAI Usage Dashboard (dashboard.openai.com → Usage) after a wave — you MUST see multiple model names in the API calls.
- Or run: `CODEX_LOG_LEVEL=debug codex ...` and grep for model names in the log.
- If only one model appears — tier switching is NOT working. Fix config before proceeding.
