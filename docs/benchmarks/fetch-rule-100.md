# Prompt-and-tool fetch encouragement

**Name:** Prompt-and-tool fetch encouragement  
**Date:** 2026-08-21  
**ID:** `fetch-rule-100`  
**n:** production_n=100, hard_n=20  
**Status:** diagnostic  
**Source:** Deep-eval:STATUS.md

Changed both system prompt and tool description; paired scoring.

## Arms

- tease
- read

## Headline

| Metric | Value |
|---|---|
| `production_delta` | -0.008 |
| `production_ci95` | 0.024 |
| `hard_delta` | 0.043 |
| `hard_ci95` | 0.084 |

## Decision

Keep conservative default for production; use read behavior only for analytical work.

## Limitations

- Fetch frequency remained low; hard set was small.
