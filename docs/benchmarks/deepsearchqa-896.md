# DeepSearchQA paired benchmark

**Name:** DeepSearchQA paired benchmark  
**Date:** 2026-08-24  
**ID:** `deepsearchqa-896`  
**n:** 896  
**Status:** canonical  
**Source:** crunch-public:BENCHMARKS.md

Official-style all-correct autorater on identical questions.

## Arms

- Crunch
- Production Deep

## Headline

| Metric | Value |
|---|---|
| `crunch` | 0.362 |
| `deep` | 0.084 |
| `paired_delta` | 0.278 |
| `ci95` | 0.034 |
| `better` | 277 |
| `worse` | 28 |
| `tie` | 591 |

## Decision

Use as evidence on exhaustive multi-step questions.

## Limitations

- Replacement Gemini autorater prevents public leaderboard comparison.
