# Planner first-hop diverse bench

**Name:** Planner first-hop diverse bench  
**Date:** 2026-08-18  
**ID:** `planner-diverse-100`  
**n:** 100 (comparable_n=98)  
**Status:** diagnostic  
**Source:** Deep-eval:docs/experiment.md

Offline first-hop tool-choice comparison without retrieval.

## Arms

- gpt-4.1-mini
- gpt-5.6-luna
- DeepSeek V4 Flash

## Headline

**`avg_tool_calls`:** `[1.78,1.92,1.7]`

**`malformed_site_dorks`:** `[22,17,0]`

## Decision

Prompt defaults, not only model choice, needed repair; superseded the narrow n=20 planner impression.

## Limitations

- Org-weighted robustness sample, not traffic weighted.
