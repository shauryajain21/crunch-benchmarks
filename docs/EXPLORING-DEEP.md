# Exploring Deep

What we learned about production Deep before deciding to replace its harness. Three questions, each answered with live traffic and paired evals: where Deep V1 fails customers, why those failures come from the harness rather than the model, and whether the new harness holds up on our own index.

Naming: **Deep V1** is the current production Deep (LanggraphDeep + DistilLabs formatter). **Deep V2** is Crunch.

## Contents

1. [Why is Deep V1 not optimal for customers?](#why-is-deep-v1-not-optimal-for-customers) — refusal rate, recency, filter leaks, judged quality, latency and cost.
2. [What are the failure modes in the Deep V1 harness?](#what-are-the-failure-modes-in-the-deep-v1-harness) — one-hop stopping, generic planner queries, split answer/source assembly, prompt defaults. Tested by swapping the planner and the writer.
3. [Vespa-only Deep V2 vs Deep V2](#vespa-only-deep-v2-vs-deep-v2-does-the-harness-hold-up-on-our-own-index) — same harness on Toolbox vs Vespa-only; the harness still wins on either index.

Evidence IDs (e.g. `prod100-corrected`) resolve to [`benchmarks/`](benchmarks/) and [`../benchmarks.json`](../benchmarks.json).

## Why is Deep V1 not optimal for customers?

- Claim: on live traffic, Deep V1 underperforms on refusal, recency, filter compliance, judged quality, latency, and cost.
- Scope: customer-visible outcomes only. Hop count, query shape, and source-list assembly stay in the next question.

### 1. Refusal rate

- Deep V1 abstains from the whole question when the first retrieved pile does not name the entity. Last 7 days, Deep V1 `sourcedAnswer` (n=209,697): **40%** start with *“The provided information does not contain…”*; **1.8%** also say *“I cannot answer…”*.
- Two high-volume orgs: ~40–43% of answers use that opening line.
- Example: “The provided information does not contain any details about [company]… I cannot answer the question based on the available sources.”
- The Distil Labs formatter treats the retrieved pile as the evidence set and abstains instead of issuing another search.
- Evidence: `gold.search_logs` (7d).

### 2. Recency / fact adjudication

- Deep V1 does not pick the later source when two retrieved pages describe the same event at different stages. Announced and completed sit in the same pile with equal weight.
- Example: a live M&A query returned “expected to close” for two deals that had already closed (one by five months). Both the announcement and the completion notice were in the retrieved set; the writer used the older page.
- Evidence: production call `fbb4032e-9e2b-493e-927f-ceec8c1ef9bf`.

### 3. Domain-filter violation rate

- Live Deep V1: **33** domain-violating URLs on 200 filtered rows. Deep V2: **0**.
- Evidence: `filtered-production-200`.

### 4. Judged quality vs alternatives

- Vendor replay checklist pass: Deep V1 **0.35–0.45** vs Exa **0.70–0.82**.
- Production Deep V2 vs Deep V1 decided win rate: sourcedAnswer **84.3%**, searchResults **72.8%**. DeepSearchQA all-correct: Deep V2 **36.2%** vs Deep V1 **8.4%**.
- Unsourced-claim rate: Deep V1 **11.5%** vs Deep V2 **2.8%**.
- Evidence: `vendor-replay-400`; `production-2000`; `deepsearchqa-896`; `prod100-corrected`.

### 5. Latency, availability, and cost

- Deep V1 is **20%** of search traffic and **60%** of serving time.
- Live: p50 **~15s**, p95 **~1 min**, HTTP **429 = 17.8%**.
- Cost: ~**$0.055/call** vs Exa deep ~**$0.012**.
- Evidence: Deep v2 RFC / Datadog.

## What are the failure modes in the Deep V1 harness?

- Claim: Deep V1 fails because of how the harness is wired, not because the planner model is weak.
- Test: we replaced the first-turn planner (gpt-4.1-mini → Luna → DeepSeek), and separately tried a stronger writer on the same retrieved evidence.
- Result: the behaviors below did not change.
- Scope: we did not run Gemini through the full live Deep V1 graph. Replacing the planner and the writer was enough to show the harness, not the model, is the constraint.

### 1. Deep V1 rarely hopped

- Config allows 10 search turns and 3 follow-ups. Those are maxima, not what actually runs.
- Live (n=400): **77%** of calls did exactly one round (306/400). Mean rounds **1.25**, vs Standard **1.00**. No call hit 10.
- Artisan/Sapiom slice: ~**85%** stopped after one iteration. Almost all page fetches happen on round 2 or later, so most calls never read a page.
- Why it stops:
  - First turn only requires one tool call. That counts as “must search.”
  - The planner can then output `Answer` and stop searching.
  - A **separate** follow-up model (always gpt-4.1-mini, not the model we swapped) then asks: does this draft already answer the question?
  - If yes, the run ends. The follow-up prompt also forbids asking for more detail. If that call errors, the run ends anyway.
- After the planner swap: mini still **69%** one call, Luna **68%**. Same pattern. We did not replace the follow-up model.
- Evidence: 400 production traces; `planner-diverse-100`; Brain `LanggraphDeep` + `FollowUpNodeFactory`.

### 2. Planner queries were generic, not new questions

- The planner does not sit in the same loop as the final answer. It writes search strings once and does not rewrite them after seeing results.
- Live (n=400): **71%** of first queries were just a shorter copy of the user’s question.
- Extra queries split the same keywords (company vs product). They do not ask a new question about the same target — status, date, or “did it close” never appears.
- Example: `Dahlia Biosciences STARVUE U-VUE spatial biology` became the full string / “Dahlia Biosciences” / “STARVUE U-VUE spatial biology”.
- The searcher prompt also says: do not invent unrelated sub-asks. A different planner model still gets that instruction.
- Deep V2: one reasoning model both plans and answers. It writes the next query after it has seen results. First turn must run at least 3 different searches.
- The same first-turn bench also caught two smaller prompt/default bugs (sections 4 and 5).
- Evidence: 400-call query parse; `planner-diverse-100`.

### 3. Answer and sources are assembled in two places

- The planner gathers evidence. DistilLabs writes the customer-facing answer on **98.5%** of Deep V1 calls and throws away the planner’s draft.
- After that answer exists, the API builds the source list. It keeps pages that were used **or** merely “relevant.”
- Observed: **30–50** URLs returned, about **7** actually cited. The list can include links the answer never used.
- Inline citations exist, but they default **off**.
- Writer swap on the same evidence (`synthesis-writers-24`): loop writer **0.746**, Gemini Pro **0.750**, at **4.8×** the cost. No quality gain.
- Deep V2: the loop model writes the answer. Code then drops URLs that were never retrieved and renumbers citations.
- Evidence: `gold.search_logs` formatter counts; source-selection prompt; `synthesis-writers-24`.

### 4. Non-English traffic defaults to US

- On non-English questions, first-turn search still set `countryCode=US` (or left it unset): mini **63%**, Luna **67%**, DeepSeek **55%**.
- That is a prompt default. Changing the model did not change it.
- Evidence: `planner-diverse-100`.

### 5. Malformed `site:` dorks

- The prompt says `site:` must be a hostname. Models still invent invalid ones: mini **22**, Luna **17** (e.g. `site:fr`, `site:nintendo.com/store`).
- Search engines ignore those, so the query returns nothing (~14% of mini’s first-turn searches, ~9% of Luna’s).
- DeepSeek invented **0** because it almost never used `site:`. The prompt itself was not fixed.
- Evidence: `planner-diverse-100`.

## Vespa-only Deep V2 vs Deep V2: does the harness hold up on our own index?

- Question: if we run Deep V2 on Vespa only (no Brave long tail), does the harness still work?
- Test: same Deep V2 agent, two retrieval stacks — Toolbox (Columbus + Brave) and Vespa-only hybrid (lexical + semantic + RRF). Everything else fixed.
- Result: yes. Vespa-only Deep V2 loses a few points to Toolbox Deep V2 on general traffic, ties on the enterprise cohort, and still beats Deep V1 by a wide margin. The harness works on either index.

### Same retrieval, different harness (Deep V1 → Deep V2)

- Both use Toolbox, so this isolates the harness change.
- Checklist prod-100: Deep V1 **0.610** → Deep V2 **0.940**. Evidence: `prod100-corrected`.
- DeepSearchQA all-correct: Deep V1 **8.4%** → Deep V2 **36.2%**. Evidence: `deepsearchqa-896`.
- Production sourcedAnswer, Deep V2 vs Deep V1: **84.3%** decided. Evidence: `production-2000`.

### Same harness, different retrieval (Toolbox → Vespa-only)

- Prod-100 checklist: **0.938 → 0.892**. Paired: 3 better / 15 worse / 82 tie. Evidence: `vespa-prod100`.
- DeepSearchQA: **36.1% → 29.3%**. Both correct 214, Toolbox-only 110, Vespa-only 49. Evidence: `vespa-dsqa-898`.
- Hard-20: **0.863 → 0.724**. Drop sits in matrix, numeric, and temporal items; chains and traps unchanged. Not a same-day pair. Evidence: `vespa-hard-20`.
- Enterprise cohort, direct pairwise: **42–35–23** with 53% swap flips → tie. Evidence: `enterprise-toolbox-v-vespa-100`.

### Vespa-only Deep V2 vs Deep V1

- Enterprise cohort, n=1,000: **821–144–35**, **85.1%** decided. Evidence: `enterprise-vespa-1000`.
- Filtered production, n=200: **67–32–101**, **67.7%** decided, **0** domain violations vs Deep V1’s **33**. Evidence: `filtered-production-200`.
- Read: the harness produced the same shape of win on Vespa alone. Retrieval quality moves the score by a few points; the loop, stopping rule, and citation plumbing move it by tens.
