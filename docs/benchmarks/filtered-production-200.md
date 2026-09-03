# Filtered production benchmark

**Name:** Filtered production benchmark  
**Date:** 2026-09-03  
**ID:** `filtered-production-200`  
**n:** 200 (population=405462)  
**Status:** diagnostic  
**Source:** crunch-public:evals/filtered-200-20260903/RESULTS.md

Question: Do immutable caller filters hold, and is filtered Crunch launch-ready?

Blind typed pairwise judge plus programmatic domain audit and 60 swaps.

## Arms

- Vespa-only Crunch v1
- Live Linkup Deep

## Headline

| Metric | Value |
|---|---|
| `wins` | 67 |
| `losses` | 32 |
| `ties` | 101 |
| `decided_win_rate` | 0.6768 |

**`search_results`:** `[6,20,49]`

**`domain_violations`:** `{"crunch":0,"deep":33}`

**`success`:** `{"crunch":192,"deep":200}`

## Decision

Retain immutable filters, but do not ship unchanged: filtered searchResults and empty-result handling regress.

## Limitations

- Date compliance could not be audited from response documents; result dominated by sourcedAnswer.

Supersedes: `filtered-production-200-v0`.
