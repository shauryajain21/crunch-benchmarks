# Hard domain filter versus broad hint

**Name:** Hard domain filter versus broad hint  
**Date:** 2026-08-20  
**ID:** `domain-scope-108`  
**n:** asker_dork_n=52, metadata_domain_n=56  
**Status:** adopted  
**Source:** Deep-eval:STATUS.md

Paired runs, repeated checklist grades.

## Arms

- Hard includeDomains
- Broad host hint

## Headline

| Metric | Value |
|---|---|
| `asker_dork_delta` | 0.132 |
| `metadata_delta` | 0.033 |

**`ci95`:** `[0.04,0.227]`

## Decision

Keep explicit user domain constraints hard.

## Limitations

- Initial subset parser falsely matched the word 'Website:' and was corrected.
