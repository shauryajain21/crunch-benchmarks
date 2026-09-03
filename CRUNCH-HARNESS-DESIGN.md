# Crunch runtime design

Crunch turns a question into a grounded, typed API response by running one fast model inside a small, controlled search loop.

`question → one fast reasoning model → search/fetch loop → answer → coverage/citation cleanup → API output`

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
