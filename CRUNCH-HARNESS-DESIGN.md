# Crunch runtime design

`question → one fast reasoning model → search/fetch loop → answer → coverage/citation cleanup → API output`

## Overall design

- The request enters a runtime harness that reads the question, requested output shape, and caller filters.
- The harness starts one model and requires it to search before answering. The model can:
  - rewrite the query
  - inspect snippets
  - fetch selected pages
  - repeat when evidence is weak
- It stops when it has enough support, then writes the answer from the evidence it gathered.
- The harness checks coverage, repairs citation numbering and placement, rejects unsupported URLs, and shapes the final API response.

The same flow supports prose answers, ranked search results, and structured output.

The model and retrieval providers sit behind stable interfaces, so either can change without changing the response contract.

## Crunch design choices

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
