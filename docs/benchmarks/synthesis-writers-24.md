# Separate synthesis writers

**Name:** Separate synthesis writers  
**Date:** 2026-08-21  
**ID:** `synthesis-writers-24`  
**n:** 24  
**Status:** rejected  
**Source:** Deep-eval:STATUS.md

Question: Does a separate Pro or Sonnet writer beat the loop model?

Same evidence, separate writer model.

## Arms

- Loop writer
- Gemini Pro writer
- Claude Sonnet writer

## Headline

| Metric | Value |
|---|---|
| `loop_pass` | 0.746 |
| `gemini_pro_pass` | 0.75 |
| `sonnet_pass` | 0.654 |
| `gemini_cost_multiple` | 4.8 |
| `sonnet_cost_multiple` | 16 |

## Decision

Keep one model for loop and writing; improve the contract instead.

## Limitations

- Targeted subset; cost deltas are decisive despite sample size.
