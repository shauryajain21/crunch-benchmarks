# Answer-contract v6/v7

**Name:** Answer-contract v6/v7  
**Date:** 2026-08-21  
**ID:** `prompt-v6-v7`  
**n:** 100  
**Status:** adopted  
**Source:** Deep-eval:STATUS.md

Question: Does the v7 answer contract reduce omissions versus v3/v6?

Paired checklist scoring and behavior inspection.

## Arms

- v3
- v6
- v7

## Headline

| Metric | Value |
|---|---|
| `v3_to_v7` | 0.012 |
| `ci95` | 0.041 |

**`missing_subset`:** `{"v3":0.258,"v6":0.192,"v7":0.167}`

## Decision

Adopt v7 for mechanism and reduced omissions, not as a proven pass-rate win.

## Limitations

- Small effects sit inside judge variance.
