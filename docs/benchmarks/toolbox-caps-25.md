# Toolbox result and page caps

**Name:** Toolbox result and page caps  
**Date:** 2026-08-21  
**ID:** `toolbox-caps-25`  
**n:** 25  
**Status:** adopted  
**Source:** Deep-eval:STATUS.md

Question: Do ten results and 12k page chars cost quality versus twenty uncapped hits?

Paired operational and checklist comparison.

## Arms

- 20 hits and uncapped pages
- 10 hits and 12k-character pages

## Headline

| Metric | Value |
|---|---|
| `answered_before` | 16/25 |
| `answered_after` | 24/24 |
| `quality_delta` | 0.015 |
| `ci95` | 0.086 |

## Decision

Cap results at 10 and fetched page text at 12k.

## Limitations

- Small sample; quality result is a null.
