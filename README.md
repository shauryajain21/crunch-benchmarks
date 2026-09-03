# Crunch benchmarks

Minimal record of the benchmarks run on Crunch. Scores are paired against the
same questions whenever possible; ties and judge instability are kept visible.

## Headline results

| Benchmark | n | Score | Comparator | Result |
|---|---:|---:|---|---|
| Production traffic — sourced answer | 687 | **84.3%** | Linkup Deep | Crunch preferred |
| Production traffic — search results | 857 | **72.8%** | Linkup Deep | Crunch preferred |
| Production traffic — structured | 319 | **69.8%** | Linkup Deep | Projector re-score |
| Filtered production traffic | 200 | **67.7% of decided** | Linkup Deep | 67–32–101 |
| Enterprise traffic, Vespa-only | 1,000 | **82.1% of rows; 85.1% decided** | Linkup Deep | 821–144–35 |
| DeepSearchQA | 896 | **36.2% all-correct** | Deep: 8.4% | +27.8 ± 3.4 points |
| Hard questions | 20 | **0.824–0.863** | Exa: 0.860–0.882 | Tie vs Exa |
| Production checklist | 100 | **0.940** | Exa: 0.894; Deep: 0.610 | Internal harness |

The complete result set, including ablations and older runs, is in
[`benchmarks.json`](benchmarks.json).

## How the benchmarks were run

### Pairwise production evaluations

1. Pin a deterministic sample of real production requests.
2. Run Crunch and the comparator on the same request.
3. Present both outputs to a blind, output-type-aware LLM judge.
4. Randomize answer order and allow ties.
5. Repeat a sample with the order swapped to measure position sensitivity.
6. Report wins, losses, ties, success rate, latency, and judge caveats.

The production-2,000 judge used `gpt-5.6-luna`, a 5,000-character answer
limit, complete result lists, and 60 swapped repeats. Enterprise evaluations
used `claude-sonnet-4-5` without answer truncation.

### Ground-truth and checklist evaluations

- **DeepSearchQA:** Google DeepMind's official autorater contract. The headline
  `all_correct` score requires every expected answer and no excessive answer.
- **Hard questions:** atomic success criteria scored independently by Gemini
  and Fable judges.
- **Production checklist:** question-only checks are generated, claims are
  extracted from each answer, then source URL validity is enforced in code.
- **Filter compliance:** returned URLs are checked directly against requested
  include/exclude domains in addition to pairwise quality judging.

## Score definitions

- **Decided win rate:** Crunch wins divided by Crunch wins plus comparator
  wins. A score labeled “of rows” uses wins divided by the full sample. Ties
  are always shown separately.
- **All-correct:** every expected answer part is present and there are no
  excessive answers.
- **Checklist score:** fraction of atomic requirements passed.
- **± interval:** paired 95% uncertainty interval where available.
- A margin smaller than the observed swap-instability is treated as a tie.

## Important limits

- The 69.8% structured score is a 319-row projector re-score, not a fresh
  456-row retrieval run.
- Customer and production sets are sampled traffic, not public leaderboards.
- DeepSearchQA used a replacement autorater model because the originally
  specified model was unavailable to new API keys; use it for paired comparison,
  not direct leaderboard comparison.
- Early 1,800-character judging understated Crunch by 23 points. Current prose
  results use 5,000 characters or no truncation.
- Raw production answers and customer queries are intentionally excluded.

## Provenance

Results were consolidated on 3 September 2026 from:

- `crunch-public` at commit `091db6c`
- `crunch-public/evals/`
- the internal `Deep-eval` benchmark harness

`benchmarks.json` records the method, configuration, score, caveats, and source
artifact paths for each run.
