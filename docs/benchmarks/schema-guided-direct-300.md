# Guided direct projection confirmation

**Name:** Guided direct projection confirmation  
**Date:** 2026-09-03  
**ID:** `schema-guided-direct-300`  
**n:** 300  
**Status:** adopted  
**Source:** crunch-public@3f00dd1:evals/schema-ab-300-20260903/RESULTS.md

Interleaved live A/B; blind exact-schema judge; 60 reversed-order checks.

## Arms

- Current gather-write-project
- Guided gather-ranked-evidence-project

## Headline

| Metric | Value |
|---|---|
| `nonempty_current` | 0.396 |
| `nonempty_guided` | 0.425 |
| `cost_per_1000_current` | 33.27 |
| `cost_per_1000_guided` | 39.02 |

**`primary`:** `{"guided":139,"current":79,"ties":82,"decided_win_rate":0.638,"ci95":[0.572,0.699],"p":5.8e-05}`

**`swap`:** `{"guided":25,"current":15,"ties":20}`

## Decision

Adopt direct ranked-evidence projection as the established lever; narrow guidance before production rollout.

## Limitations

- Gain is concentrated in one schema family; guidance versus unguided direct remains unproven; cost rose 17.3%.

Supersedes: `schema-factorial-40`, `structured-projector-rescore-319`.
