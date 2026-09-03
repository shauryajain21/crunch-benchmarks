# Vendor replay on production traffic

**Name:** Vendor replay on production traffic  
**Date:** 2026-08-17  
**ID:** `vendor-replay-400`  
**n:** 400 (population=Two high-volume production cohorts; prompts excluded from archive)  
**Status:** diagnostic  
**Source:** Deep-eval:docs/experiment.md

Same anonymized request IDs; checklist judge on 400 Flash rows and 398 Sonnet rows.

## Arms

- Linkup Deep
- Linkup Standard
- Exa variants
- Parallel Core

## Headline

**`flash_pass`:** `{"exa_reasoning":0.82,"exa_agent":0.798,"exa_lite":0.792,"parallel_core":0.616,"deep":0.352}`

**`sonnet_pass`:** `{"exa_agent":0.726,"parallel_core":0.72,"exa_reasoning":0.705,"deep":0.452}`

## Decision

Motivated owning the search loop; vendor results are supporting evidence, not Crunch claims.

## Limitations

- Traffic mix was concentrated; this predates Crunch and the initial judge clipped answers.
