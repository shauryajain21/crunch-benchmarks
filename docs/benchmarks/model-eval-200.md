# Five-model generation bake-off on filtered-200

**Name:** Five-model generation bake-off on filtered-200  
**Date:** 2026-09-03  
**ID:** `model-eval-200`  
**n:** 200  
**Status:** incomplete  
**Source:** local:/Users/shaurya/tmp/crunch-model-eval-200-20260903

Question: Which loop models complete the filtered-200 sample?

Live Crunch generation through local Toolbox on the pinned filtered-200 sample. Used to release Toolbox before firsthop-ab-100. No checklist or pairwise judge file exists.

## Arms

- gemini-direct 200/200 ok
- glm-baseten 199/200 ok
- luna-safe 192/200 ok
- openrouter gemini/glm/qwen/deepseek incomplete or failed

## Headline

| Metric | Value |
|---|---|
| `gemini_direct_ok` | 200 |
| `glm_baseten_ok` | 199 |
| `luna_safe_ok` | 192 |
| `quality_judge` | — |

## Decision

Do not use as a quality result. Operational completion only.

## Limitations

- No quality scores. Several OpenRouter arms failed or stopped early.
