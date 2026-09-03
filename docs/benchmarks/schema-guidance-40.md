# Schema-guided gathering pilot

**Name:** Schema-guided gathering pilot  
**Date:** 2026-09-03  
**ID:** `schema-guidance-40`  
**n:** 40  
**Status:** rejected  
**Source:** crunch-public@c1e405d:evals/schema-guidance-pilot-20260903/RESULTS.md

Live interleaved arms; blind two-order Gemini judge.

## Arms

- Current write path
- Broad schema-guided write path

## Headline

| Metric | Value |
|---|---|
| `nonempty_control` | 0.268 |
| `nonempty_guided` | 0.329 |
| `stable_guided_wins` | 9 |
| `stable_control_wins` | 8 |
| `latency_control_s` | 22.6 |
| `latency_guided_s` | 29.9 |

## Decision

Do not ship broad guidance alone: completion rose but correctness tied and latency increased.

## Limitations

- Prior Deep-win sample; three schema families; n=40.
