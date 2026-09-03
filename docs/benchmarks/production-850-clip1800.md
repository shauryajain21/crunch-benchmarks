# 850-row prose judge with 1,800-character clip

**Name:** 850-row prose judge with 1,800-character clip  
**Date:** 2026-09-01  
**ID:** `production-850-clip1800`  
**n:** 292 (mode=sourcedAnswer)  
**Status:** invalid  
**Source:** crunch-public:evals/RESULTS.md

Blind pairwise with asymmetric clipping.

## Arms

- Crunch
- Frozen Deep

## Headline

| Metric | Value |
|---|---|
| `reported_win_rate` | 0.613 |
| `corrected_win_rate` | 0.8419 |

## Decision

Invalidate the 61% number; raise clip to 5,000 or do not truncate.

## Limitations

- 75% of Crunch versus 20% of Deep was clipped.
