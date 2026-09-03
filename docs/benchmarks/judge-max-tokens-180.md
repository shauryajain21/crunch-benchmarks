# Judge max_tokens=180 clip

**Name:** Judge max_tokens=180 clip  
**Date:** 2026-09-02  
**ID:** `judge-max-tokens-180`  
**n:** campaign=enterprise-toolbox-100, regraded=True  
**Status:** invalid  
**Source:** crunch-public:evals/sapiom-deep-100-20260902/RESULTS.md

Question: Did Anthropic max_tokens=180 silently invent pairwise ties?

Caught mid-run on the Sapiom-100 pairwise judge: 180 completion tokens cut the judge JSON and produced fake ties. Raised to 1024 and those rows were re-graded.

## Arms

- Anthropic max_tokens=180
- max_tokens=1024

## Headline

No scoreable headline. See limitations.

## Decision

Do not use any mid-run ties from the 180-token clip. The published Sapiom-100 scores are the regrade.

## Limitations

- No separate scoreboard for the clipped pass; it was discarded.
- Primary numbers for the campaign live on enterprise-toolbox-100.
