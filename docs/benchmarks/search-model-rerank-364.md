# Model-ranked searchResults

**Name:** Model-ranked searchResults  
**Date:** 2026-09-01  
**ID:** `search-model-rerank-364`  
**n:** 364  
**Status:** adopted  
**Source:** crunch-public:evals/RESULTS.md

Question: Does the model choosing across sub-query lists beat score-max versus Deep?

Typed pairwise judge on same 850 pilot.

## Arms

- Model reranking
- Frozen Deep

## Headline

| Metric | Value |
|---|---|
| `wins` | 277 |
| `losses` | 87 |
| `ties` | 0 |
| `decided_win_rate` | 0.761 |

## Decision

Use the model to choose across sub-query result lists.

## Limitations

- Pilot slice; canonical score comes from n=857.

Supersedes: `search-rank-rff-50`, `search-rank-score-100`, `search-max-score-364`.
