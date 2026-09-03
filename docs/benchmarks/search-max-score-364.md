# Cross-query max-score search ranking

**Name:** Cross-query max-score search ranking  
**Date:** 2026-09-01  
**ID:** `search-max-score-364`  
**n:** 364  
**Status:** invalid  
**Source:** crunch-public:evals/RESULTS.md

Question: Can raw cross-encoder maxima rank across rewritten sub-queries?

Production 850 pilot.

## Arms

- Max raw cross-encoder score

## Headline

| Metric | Value |
|---|---|
| `reported_win_rate` | 0.452 |

## Decision

Do not compare raw reranker scores across rewritten queries.

## Limitations

- Narrow rewrites dominated due to score-scale mismatch.
