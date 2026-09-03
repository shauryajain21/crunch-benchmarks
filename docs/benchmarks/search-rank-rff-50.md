# Reciprocal-rank fusion for searchResults

**Name:** Reciprocal-rank fusion for searchResults  
**Date:** 2026-08-24  
**ID:** `search-rank-rff-50`  
**n:** 50  
**Status:** superseded  
**Source:** Deep-eval:results/output_modes/README.md

Gold-answer recall at fixed cuts.

## Arms

- Arrival order
- Reciprocal-rank fusion

## Headline

| Metric | Value |
|---|---|
| `top10_arrival` | 0.355 |
| `top10_rrf` | 0.521 |
| `delta` | 0.166 |

## Decision

Replace concatenation with a real ranking; later superseded by model ranking.

## Limitations

- Only 50 DSQA rows; no exposed comparable cross-query score.
