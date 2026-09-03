# Timeline

## 17–18 August 2026 — establish the problem
- A 400-request vendor replay showed that the existing answer/judge contract rewarded competitors and exposed attribution gaps. It motivated an owned loop; it was not itself a Crunch result.
- The planner robustness bench found malformed domain dorks and a country default shared across models.

## 18–19 August — build and select the loop
- The first index agent used parallel `search`/`fetch`, stable evidence IDs, and a deadline governor.
- The n=20 bake-off selected Gemini 3.7 Flash without a deadline.
- A hard 20-question set showed parity with Exa and a matrix-shaped weakness.
- The initial production-100 run looked much worse than Exa, but its magnitudes were later invalidated.

## 20–21 August — repair the instrument, then simplify
- Citation renumbering and answer-clip audits recovered large false losses.
- Hard domain scope beat broad hints. Brave breadth, ranked auto-read, claim verification, larger writers, and separate synthesis did not pay.
- v6/v7 answer contracts and a coverage pass reduced omissions.
- Automatic scraping was deleted after a 3.2× latency reduction with no quality loss.
- The corrected production-100 board reached 0.940 checklist pass, with parity—not superiority—versus Exa on ordinary lookups.

## 24–25 August — public implementation and output contracts
- The public one-file agent gained sourced, structured, and search-results modes.
- Structured projection became question-aware and schema-validated.
- Search results moved from arrival order toward explicit ranking.
- An explicit chain prompt was a no-op; observed behavior was guess-and-verify.
- A 1,000-row production replay and DeepSearchQA established broader performance, while retaining filter and autorater caveats.

## 31 August — Vespa-only versus Toolbox
- Same Crunch agent on vespa-hybrid (no Brave): production-100 0.892 vs Toolbox 0.938; DeepSearchQA 29.3% vs 36.1%; hard-20 0.724 vs a published Toolbox 0.863.
- Sixteen hops did not close the DSQA gap. A 1,000-row vespa production run did not start.

## 1 September — typed production bench and ablations
- The 850-row pilot exposed a 1,800-character judge clip, cross-query score misuse, split-finish status errors, and a row-1,520 runner crash.
- Model ranking and a `done` finish fixed searchResults quality and latency.
- Forced hops, one literal query, Vespa-only general retrieval, mechanical structured abstention, and the first gather→schema path were rejected.
- The corrected stratified 2,000-row run produced canonical sourcedAnswer and searchResults scores. Shipping structured remained near a tie.

## 2 September — projector repair and enterprise retrieval
- Null/coercion/source repairs lifted a selected structured re-projection to 69.8% decided, but it was not a fresh full run.
- A 1,000-row enterprise cohort showed Vespa-only Crunch at 82.1% of rows versus stored Deep. Mid-run, `max_tokens=180` clipped the judge JSON and was regraded.
- Toolbox and Vespa were indistinguishable on the smaller direct comparison because swap instability exceeded the margin.

## 3 September — filters and direct structured projection
- Filtered v1 achieved zero observed domain violations, but filtered searchResults and empty-result reliability blocked shipment.
- Broad schema guidance improved completion but not judged correctness.
- A four-arm holdout isolated direct ranked-evidence projection as the likely lever.
- Commit `3f00dd1` confirmed guided-direct over current write→project on 300 rows (139–79–82); the gain was schema-family dependent and cost more input tokens.
- A 100-row first-hop A/B (`478a4ae`) isolated `min_first_searches=3` versus a free first turn: 41–24–35 for the force, but 40% swap flips, so the shipped rule was left unchanged.
