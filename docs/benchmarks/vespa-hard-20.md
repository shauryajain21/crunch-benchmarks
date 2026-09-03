# Vespa-only versus Toolbox on hard-20

**Name:** Vespa-only versus Toolbox on hard-20  
**Date:** 2026-08-31  
**ID:** `vespa-hard-20`  
**n:** 20  
**Status:** diagnostic  
**Source:** Deep-eval:SESSION.md

Question: Does Vespa-only hybrid match Toolbox Crunch on the hard set?

Same vespa-hybrid DAG. Gemini criterion judge. Toolbox 0.863 is the published 6-hop v1 Gemini number, not a same-day v7 regrade.

## Arms

- Vespa-only hybrid Crunch
- Published Toolbox Gemini v1

## Headline

| Metric | Value |
|---|---|
| `vespa` | 0.724 |
| `toolbox_published` | 0.863 |
| `delta` | -0.139 |
| `vespa_p50_s` | 21.1 |

**`by_kind`:** `{"chain":[0.81,0.81],"trap":[0.94,0.94],"matrix":[0.53,0.75],"numeric":[0.5,1.0],"temporal":[0.56,1.0]}`

## Decision

Hard-set drop is larger than prod-100 and concentrated in matrix, numeric, and temporal items.

## Limitations

- Toolbox comparator is an older published grade, not a same-day paired regrade.
