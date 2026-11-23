# Cost and latency budget

Type: wayfinder:grilling
Blocked by: Supervisor design
Status: CLOSED 2026-11-23

## Question
What are the spend and time limits for one run? Decide model tiers per agent, per-run token and dollar caps, how oversized snapshots are handled (truncating, dropping, summarising), caching, and the target latency for a typical run. (Reworded from the original, which still described PR diffs.)

## Resolution
All numbers are editable config defaults, not claims about what a run costs. Current prices were not checked; the dollar cap is computed from token usage times a price table in config that the author maintains.

**Model tiers by job.**
- Fast tier, Haiku 5.5: Triage (fact extraction; code verifies every quote, so a weaker model can miss facts but cannot smuggle in unsupported ones, and the evals measure the misses) and the writer (composes from already verified claims).
- Strong tier, Sonnet 5.5: Bull, Bear, Risk and the Critic, the judgment steps.
- One environment switch puts the whole graph on the fast tier for CI and cheap eval runs. Opus is not used in v1; a cost-versus-quality chart across tiers could be a later eval experiment.

**Caps per run.**
- Sources into Triage: at most 20, chosen by distinct publishers first and recency second; extras dropped and noted.
- Text per source: at most 2,000 characters. Headlines pass through whole; long filing excerpts are cut at a sentence boundary with a note.
- Model calls per run: at most 12 (about 8 expected, leaving room for the revision round and retries).
- Output tokens: a hard `max_tokens` per agent. Input and output tokens: a total cap per run.
- Dollar cap per run: $0.25 by default, checked before each call, stopping if the next call would exceed it.
- Hitting any cap returns a typed "budget exceeded" failure, never partial output.
- No chunking or summarizing of huge inputs in v1. Truncating and dropping by rule is repeatable and cannot introduce summary errors that the quote check would miss.
- Every truncation, drop and cap hit appears in the analysis notes and the trace.

**Latency.** Bull, Bear and Risk run in parallel. Per-call timeout 45 seconds, whole-run timeout 120 seconds (typed "timed out" failure), typical-run target under about 60 seconds. The 60 seconds is a goal to check in the evals, not a measurement.

**Caching.**
- The engine has no cache; it stays a pure function.
- The later service caches by a hash of the snapshot contents plus the config and prompt versions. This fixes Freeman's weakness, where the cache key ignores the news content and can hide new articles behind a stale result.
- The eval runner has an optional record/replay cache keyed by (model, prompt hash), so CI reruns are free and repeatable, and it can be switched off to measure live behavior.
