# Forced first-hop A/B

**Name:** Forced first-hop vs free first turn  
**Date:** 2026-09-03  
**ID:** `firsthop-ab-100`  
**n:** 100 production `sourcedAnswer`  
**Status:** diagnostic  
**Commit:** crunch-public `478a4ae`

Question: does forcing at least three first-turn searches (no answer on turn 1) beat allowing the model to answer without that force?

## Arms

Identical except the first-hop rule. Gemini 3.7 Flash via direct `GEMINI_API_KEY`, local Toolbox, v7 contract, coverage, citation repair, 8 hops, 150s cap, 10 results, 12k page chars.

`crunch.py` has no CLI flag. Isolation was `MIN_FIRST_SEARCHES` plus, on the free arm only, replacing the stock first-turn force bullet. That bullet still forbids a first-turn answer when `min_first=0`, so leaving it would not isolate the rule. The rest of the v7 contract, including “never answer from memory,” was unchanged.

| Arm | `min_first_searches` | First-turn `tool_choice` | First-turn answer |
|---|---:|---|---|
| shipped | 3 | required | forbidden |
| free | 0 | auto | allowed |

Sample: Deep-eval `deep_prod_100` (`prod_001`–`prod_100`). Same IDs on both arms. Raw rows stay in `/Users/shaurya/tmp/crunch-firsthop-ab-100-20260903`.

## Headline

| n | shipped | free | tie | shipped decided | 95% CI |
|---|---:|---:|---:|---:|---|
| 100 | **41** | 24 | 35 | **63.1%** | 50.9–73.8% |

Both arms completed 100/100. Failures included (none). Judge: Claude Sonnet 4.5, blind typed pairwise, randomized order, ties allowed, no answer clip, 40 swap checks.

## Swap

| Check | Result |
|---|---|
| Swap n | 40 |
| `winner_arm` flips | 16 (40.0%) |
| swap_report flips | 11 (27.5%) |
| Swap-stable | shipped 15, free 3, tie 6, flip 16 |
| First-position rate | 0.438 |

The 13-point decided margin over 50% does not beat 40% order instability. No shipped-config change.

## Mechanism

| | shipped | free |
|---|---:|---:|
| First-turn searches | 3.31 | 2.03 |
| First-turn hist | 3×69, 4×31 | 1×48, 2×13, 3×27, 4×12 |
| Answered on turn 1 | 0 | 0 |
| Zero-search answers | 0 | 0 |
| Mean searches / hops | 3.80 / 2.40 | 2.74 / 2.71 |
| p50 / p90 latency | 10.5s / 19.6s | 10.6s / 17.0s |
| Search RPCs | 380 | 274 |

The force did what it claims: shipped never issued fewer than three first-turn searches. Free usually issued one or two, then continued. “Never answer from memory” still blocked a memory-only first turn, so this is not an ungrounded-answer ablation.

Latency was a tie at the median. Free used fewer searches and a slightly faster p90.

## Safety

One arm at a time, 8 question workers, 24 in-flight `WebSearchTool` RPCs. No Searcher 429/503. Search transport errors 0. Rolling search p90 stayed ~1.4s. Fetch p90 was 14–28s and was not treated as a Vespa stop. No Vespa/config writes.

## Judge notes

- `--no-truncate`; answer clip 0. Old 1,800-character clip would have cut 65/100 rows.
- `shown_truncated` fired on 38/100 because some answers already contain an ellipsis character, not because the judge clipped.
- One row (`prod_018`) was unparseable both orders and recorded as a tie.
- Longer-answer wins 41/65 decided. That is confounded with the shipped arm writing more; the prompt forbids length as a win.

## Decision

Keep the shipped first-hop rule. The quality lean toward force is real in direction and in swap-stable counts, but it is not large enough to declare a new design result against 40% order flips. Free first turn is not a latency win at p50.

Related: `forced-gather-hops-40` rejected extra *later* hops. This test is the *first* hop only.
