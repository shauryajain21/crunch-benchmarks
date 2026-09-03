# Checklist judge answer-clip audit

**Name:** Checklist judge answer-clip audit  
**Date:** 2026-08-21  
**ID:** `judge-clip-1400`  
**n:** 100  
**Status:** invalid  
**Source:** Deep-eval:STATUS.md

Regraded identical answers.

## Arms

- 1,400-character clip
- 16,000-character clip

## Headline

| Metric | Value |
|---|---|
| `crunch_v7_delta` | 0.094 |
| `exa_delta` | 0.096 |
| `deep_delta` | 0 |

## Decision

Invalidate pre-21-Aug cross-system magnitudes and remove asymmetric clipping.

## Limitations

- Rankings mostly survived, but deltas did not.
