# Hard multi-hop benchmark

**Name:** Hard multi-hop benchmark  
**Date:** 2026-08-19  
**ID:** `hard-bench-20`  
**n:** 20 (completed_n=17)  
**Status:** canonical  
**Source:** crunch-public:BENCHMARKS.md

Fixed atomic criteria; independent Gemini and Fable judges.

## Arms

- Crunch
- Exa Deep Reasoning
- Linkup Deep

## Headline

| Metric | Value |
|---|---|
| `crunch_vs_exa_delta` | -0.028 |
| `ci95` | 0.151 |

**`gemini`:** `{"crunch":0.863,"exa":0.86,"deep":0.653}`

**`fable`:** `{"crunch":0.824,"exa":0.882,"deep":0.686}`

## Decision

Treat Crunch and Exa as tied; matrix questions remain Crunch's clearest gap.

## Limitations

- Three Crunch rows had network failures in the later fixed-write comparison; n is small.
