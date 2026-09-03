# Explicit chain instruction A/B

**Name:** Explicit chain instruction A/B  
**Date:** 2026-08-25  
**ID:** `chain-prompt-10`  
**n:** 10  
**Status:** rejected  
**Source:** crunch-public:behaviours.md

Question: Does an explicit sequence-the-chain bullet beat guess-and-verify?

Alternating paired runs, blind fixed checks.

## Arms

- Stock prompt
- Sequential-chain bullet

## Headline

| Metric | Value |
|---|---|
| `stock_pass` | 10/10 |
| `chain_pass` | 10/10 |
| `better` | 0 |
| `worse` | 0 |
| `tie` | 10 |

## Decision

Preserve guess-and-verify; do not spend a turn sequencing guessable entities.

## Limitations

- Small, model-guessable set; unknown intermediates remain untested.
