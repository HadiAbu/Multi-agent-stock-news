# Eval metrics, snapshots and CI gating

Type: wayfinder:grilling
Blocked by: Analysis schema and engine interface, Data sources and terms
Status: CLOSED 2026-11-23

## Question
There is no ground truth for a "right" stance, so what is measured? Decide the frozen news-snapshot set (sources, size, how it is stored without licence or privacy problems), the metrics (citation validity, unsupported-claim rate, abstention on thin news, injection pass rate, cost and latency per run), how an LLM judge is used and checked, which model runs in CI, pass/fail thresholds, and how results appear in the README.

## Resolution
**Ground truth by design.** Real news has no right answer, so the evals use synthetic cases with a designed answer: each case is a snapshot with a designed lean or an expected abstention, a list of planted facts, and a tag for the failure mode it tests. Most metrics can then be checked by code.

**Metrics.**
Deterministic, checked by code (the bulk):
- Lean agreement on designed cases.
- Abstention correctness: thin and one-sided snapshots must return `unclear`, and well-supported ones must not.
- Fact recall in Triage: how many planted facts it extracted.
- Dropped-claim rate: how often agents propose claims that code must drop (invalid quotes or facts). It measures agent quality, because the final output is clean by construction.
- Advice-lint hits in final outputs: must be zero.
- Injection pass rate from the paired cases, per attack type.
- Confidence calibration: lean agreement by confidence level; `high` should beat `medium`, and `medium` should beat `low`.
- Cost, model calls and latency per run.

One LLM judge, only for whether the summary text is faithful to the surviving claims. The judge model is pinned, validated against a few hand-labelled examples, and reported together with its agreement rate. It runs only in live mode.

**The eval set (about 56 snapshots from a pool of roughly 60 short synthetic sources).**

| Case type | Count |
|---|---|
| Clearly positive or negative, several publishers | 12 |
| Mixed or conflicting facts | 6 |
| Thin: too few sources, a single publisher, or stale | 8 |
| One-sided | 4 |
| EDGAR-based, real filing excerpts with a planted-fact list | 6 |
| Clean subtotal | 36 |
| Poisoned variants for the paired injection tests, each paired with a clean case | 20 |

An LLM drafts the sources from designed briefs and the author reviews and edits each one for realism and label correctness. Everything is tagged synthetic or EDGAR, with source and retrieval date for real items. The README states the caveat plainly: model-written articles are easier for a model to read than real ones, so scores will look better than real-world performance, and the real EDGAR cases help keep it grounded.

**CI.**
- Every push and PR, free and deterministic: unit tests with a fake LLM and fake news client; the eval suite in replay mode (recorded model responses, no API key needed, so forked PRs can run it); the privacy scan (gitleaks plus the path grep). Replay mode catches regressions in the code around the models: grounding, quote checks, the lint, confidence rules, routing and abstention.
- Live runs (real models, costs money): manual or nightly on `master`, fast tier for cost. They re-record the cache, run the judge, and write a results file (metrics per category, cost, latency, judge agreement) committed under `docs/`. The README results table is updated from that file.

**Hard gates in replay mode (a failure blocks the merge).**
- Any advice-lint hit in a final output.
- Any invalid quote or unknown citation in a final output.
- Injection pass rate below the committed baseline.
- Lean agreement or abstention correctness more than a set tolerance below the committed baseline.
- Run cost or calls above the caps.
Live-run metrics are reported, not gating, because model variance would make builds flaky. The baseline is a committed file that the author updates on purpose when behavior improves.

**Left to the build plan.** Exact tolerances, how briefs for synthetic sources are written, and the judge's validation set.
