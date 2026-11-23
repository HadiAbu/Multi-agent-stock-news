# LangGraph vs CrewAI comparison protocol

Type: wayfinder:grilling
Blocked by: Analysis schema and engine interface, Eval metrics, snapshots and CI gating
Status: CLOSED 2026-11-23

## Question
Which agent (probably the Bull/Bear debate) is rebuilt in CrewAI, and how is it compared fairly? Decide that both use the same model, prompts and snapshots, which measures are reported (quality, cost, latency, lines of code, debuggability), how many runs to average out variance, and how the write-up is framed so it shows judgment rather than a winner-takes-all claim.

## Resolution
**What is rebuilt: the analysis round.** CrewAI reimplements Bull, Bear, Risk and the Critic with its revision step. The input is verified facts and the output is claims with rulings, in the same schema. Triage, the quote check, the writer and all code checks (grounding, lean check, confidence, lint) stay shared code, so differences come from the framework and how it orchestrates agents. The crew never sees raw source text, so the quarantine boundary holds. Rebuilding only Triage (a single agent) or the whole pipeline (unfair, since code routing suits LangGraph's explicit graph) was rejected. Expected friction: the bounded one-round revision loop is natural in LangGraph and awkward in CrewAI. That friction is a legitimate finding, not a setup flaw.

**Held constant on both sides.** Same model tier (Sonnet 5.5 for all four agents); same instruction text in each prompt (CrewAI's role, goal and backstory fields wrap the same instructions); same input facts and output schema; same eval cases; same code checks after the round; same run caps; both traced to Langfuse so traces compare side by side.

**Reported, per framework.**
- Quality: lean agreement, abstention correctness, dropped-claim rate, advice-lint hits, injection pass rate.
- Cost: model calls, tokens and dollars per run (from the config price table).
- Latency per run.
- Size: lines of code for the round, counted the same way on both sides.
- Debuggability, as a short qualitative note with Langfuse screenshots: how readable a failed run is, and what it took to add the one-round revision cap.

**Variance.** All 56 cases, 3 runs each per framework, reporting mean and spread. No significance claim, because the sample is small. A pilot of about 10 cases runs first to catch setup bugs and to measure real cost. Worst-case bound: 56 cases times 2 frameworks times 3 runs is 336 runs; at the $0.25 per-run cap that is about $84. Real cost should be far lower; the pilot gives the actual number.

**Framing.**
- Predictions are written and committed before the pilot, for example which framework will be cheaper, easier to debug, shorter, and whether quality will differ.
- The write-up reports which predictions held and which did not, including results that go against the design. It does not crown a winner. It says which kind of system suits which framework (explicit code-routed graphs versus role-based crews) and states the limits: one task, one model tier, a synthetic eval set, a small sample.
- It lives in `docs/comparison.md`, with a short summary table in the README and the raw per-run results committed so anyone can recheck the numbers.
- An LLM-supervisor variant, rejected for the main design, could be a further experiment but is not part of this protocol.
