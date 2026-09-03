# Production traffic paired benchmark

**Name:** Production traffic paired benchmark  
**Date:** 2026-08-24  
**ID:** `production-1000-legacy`  
**n:** 1000 (comparable_n=615)  
**Status:** superseded  
**Source:** crunch-public:BENCHMARKS.md

Blind pairwise with 60 order swaps.

## Arms

- Crunch
- Production Deep

## Headline

| Metric | Value |
|---|---|
| `wins` | 512 |
| `losses` | 96 |
| `decided_win_rate` | 0.842 |
| `crunch_p50_s` | 9.3 |
| `deep_p50_s` | 10.1 |

## Decision

Historical production result; use the stratified 2,000-row run for current claims.

## Limitations

- 385 filtered rows excluded; older implementation.
