# Crunch runtime design

Crunch turns a question into a grounded, typed API response by running one fast model inside a small, controlled search loop.

`question → one fast model → search/fetch loop → answer → coverage/citation cleanup → API output`

## Overall design

The request enters a runtime harness that reads the question, requested output shape, and caller filters.
The harness starts one model and requires it to search before answering.
The model can rewrite the query, inspect snippets, fetch selected pages, and repeat when evidence is weak.
It stops when it has enough support, then writes the answer from the evidence it gathered.
The harness checks coverage, repairs citation numbering and placement, rejects unsupported URLs, and shapes the final API response.

The same flow supports prose answers, ranked search results, and structured output.
The model and retrieval providers sit behind stable interfaces, so either can change without changing the response contract.

## What the harness owns

- Request parsing, caller filters, output mode, and schema.
- The model/tool loop, tool limits, evidence IDs, context limits, and stopping rules.
- The rule that evidence must be gathered before an answer is accepted.
- Coverage checks, citation cleanup, URL safety, schema validation, and typed success or failure.
- Final response shaping for each API output mode.

## What retrieval owns

- Searching indexes or the web for candidate documents.
- Returning snippets, document metadata, and stable source URLs.
- Fetching page text when the model selects a result to read.
- Retrieval ranking and provider-specific behavior behind the search/fetch interface.

Retrieval supplies evidence; it does not decide when the whole task is complete or define the final API response.

## Why it is built this way

- **Own the loop.** Crunch controls tool use, budgets, failures, and stopping instead of delegating those choices to a nested agent framework.
- **Force grounding before answering.** A required first search makes retrieved evidence the starting point, not an afterthought.
- **Stop adaptively.** Easy questions can finish quickly; harder questions can rewrite, search again, or fetch a source.
- **Use snippets first and fetch selectively.** Snippets are often enough to choose or reject a result, while full-page reads are reserved for pages that matter.
- **Let the same model gather and write.** It keeps the question, query choices, evidence, and uncertainty in one context.
- **Post-process citations and output shapes.** Deterministic code enforces contracts that prompts alone cannot guarantee.

## Anchor measurements

- On production `sourcedAnswer` traffic, Crunch won **84.3% of decided comparisons** against frozen Deep, supporting the end-to-end loop design.
- A separate Pro writer scored **0.750 versus 0.746** for the loop writer but used **4.8× the model cost**, supporting one model for gathering and writing.
- Automatically scraping top pages increased median latency from **9.5s to 30.5s with no quality gain**, supporting snippets-first, model-selected fetch.

## Current boundaries and open questions

- General web retrieval and bounded enterprise retrieval need different validation; one backend is not assumed to fit all traffic.
- Structured output works through the same evidence loop, but projection quality across new schema families still needs testing.
- Filtered search and empty-result behavior must improve without weakening caller domain and date constraints.
- The best trigger for eager fetching on analytical or multi-part questions is still open.
- Cheaper models are candidates only if they preserve first-search compliance, answer quality, and latency.

Detailed component decisions will be added only after this overall design is reviewed.

## Design choices shaped by evaluation

### 1. Own the agent loop

- The harness controls model turns, tool calls, budgets, failures, and stopping.
- On production `sourcedAnswer` traffic, n=687 produced 578 Crunch wins, 108 losses, and 1 tie against frozen Deep.
- This supports the whole Crunch system, but does not isolate loop ownership. It is end-to-end support for keeping the loop in the harness.

### 2. Stop adaptively

- The model can stop after enough evidence or continue with another search or fetch.
- Forced one-hop scored 62.5% and forced two-hop scored 65.0%, each on n=40 against Deep.
- The roughly 76% figure comes from the separate n=200 split-finish `searchResults` arm against Deep, not a directly paired three-arm test. We therefore avoid fixed hop counts.

### 3. Use snippets first

- Search returns snippets first; the model fetches full pages only when needed.
- On n=100, automatic scraping scored 0.930 at 30.5s p50. No automatic scraping scored 0.940 at 9.5s.
- Scraping added large latency without quality gain, so automatic top-page scraping was disabled.

### 4. Return ten results per search

- Each search returns at most ten results, reducing model context and retrieval cost.
- On n=25, ten versus twenty results changed quality by +0.015 ± 0.086.
- The quality difference was a null, so the lower-cost, smaller-context limit stayed at ten.

### 5. Use the v7 answer contract

- The v7 prompt makes the answer requirements explicit before final writing.
- On 24 difficult questions, missing requirements fell from 25.8% to 16.7%. The full n=100 gain was only +0.012 ± 0.041.
- We adopted v7 for fewer omissions and its clearer mechanism, not as a large aggregate quality win.

### 6. Run a coverage pass

- After writing, the harness checks whether requested parts are missing and fills supported gaps.
- On n=100, score rose from 0.911 to 0.930; the benefit was concentrated in agentic tasks.
- We kept the pass as an omission safeguard, not as a broad lookup-quality claim.

### 7. Repair and renumber citations

- Deterministic cleanup maps citations to the final emitted source list and renumbers them.
- Regrading the same stored answers raised score from 0.571 to 0.748; unsourced claims fell from 23.8% to 6.1%.
- Because the answers did not change, this showed a plumbing failure. Citation repair became part of finalization.

### 8. Model-rank `searchResults`

- The model chooses across result lists from rewritten subqueries instead of comparing raw cross-query scores.
- On the same n=364 benchmark slice, score-max reached 45.2% against Deep and model reranking reached 76.1% against Deep.
- These are separate comparisons to Deep, not necessarily a direct reranker-versus-score judge. The score-scale failure still justified model ranking.

### 9. End `searchResults` with `done`

- Once ranking is complete, the model calls `done` instead of writing prose that the API discards.
- On n=200, quality was unchanged by the paired sign test (p=0.67), while latency fell from 44.1s to 8.4s.
- The large latency gain with no measured quality loss made `done` the finish contract.

### 10. Keep filters outside the model

- The harness applies caller domain and date filters as immutable constraints outside model-written queries.
- On n=200 filtered requests, Crunch returned 0 domain-violating URLs versus 33 for Deep.
- Crunch `searchResults` recall and reliability were worse, so the filter design stayed but the release was not approved.
