# Forced gather-hop count

**Name:** Forced gather-hop count  
**Date:** 2026-09-01  
**ID:** `forced-gather-hops-40`  
**n:** 40  
**Status:** rejected  
**Source:** crunch-public:evals/RESULTS.md

Question: Does forcing one or two later gather hops beat letting the model stop?

Pilot ablation against frozen Deep.

## Arms

- Forced one hop
- Forced two hops

## Headline

| Metric | Value |
|---|---|
| `one_hop_win_rate` | 0.625 |
| `two_hop_win_rate` | 0.65 |

## Decision

Let the model decide when to stop; more forced searches were not better.

## Limitations

- Directional small slice.
