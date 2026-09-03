# Enterprise Vespa-only benchmark

**Name:** Enterprise Vespa-only benchmark  
**Date:** 2026-09-02  
**ID:** `enterprise-vespa-1000`  
**n:** 1000 (mode=sourcedAnswer)  
**Status:** canonical  
**Source:** crunch-public:evals/sapiom-deep-1000-20260902/RESULTS.md

Blind no-truncation Sonnet judge; 200 swaps.

## Arms

- Vespa-only Crunch
- Stored Deep

## Headline

| Metric | Value |
|---|---|
| `wins` | 821 |
| `losses` | 144 |
| `ties` | 35 |
| `row_win_rate` | 0.821 |
| `decided_win_rate` | 0.8508 |
| `p50_s` | 12.3 |
| `mean_s` | 13.4 |
| `cost_per_query` | 0.018 |

## Decision

Canonical enterprise-cohort result; proves Vespa-only viability on this traffic.

## Limitations

- Single cohort; hash sample over-represents a repeated wrapper pattern; 26.5% swap flips.

Supersedes: `enterprise-vespa-100`.
