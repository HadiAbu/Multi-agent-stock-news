# Research Desk: design spec

Status: draft for review, 2026-11-23.
Source of truth for decisions: the closed tickets in `docs/wayfinder/research-desk/` (linked inline). Vocabulary: `GLOSSARY.md`. Where this spec and a ticket disagree, fix the one that is wrong and say so in the ticket.

## 1. Overview

The research desk turns a stock ticker and a snapshot of recent news into a cited, abstention-aware research note. A code-routed LangGraph graph runs a Triage agent, then Bull, Bear and Risk agents in parallel, a Critic, and a writer. Code checks every quote, citation, lean and confidence level. When the evidence is thin or one-sided, the answer is an abstention (`unclear`), not a guess.

It is a portfolio project. Its purpose is to show practical skill with LangChain/LangGraph and CrewAI, and with the parts of agent systems that are usually skipped: untrusted input, grounding in code, evaluation without ground truth, tracing and cost control.

It describes what published sources say. It never predicts prices, recommends trades, or uses personal data.

## 2. Goals and non-goals

Goals (v1):
- A standalone Python engine, `stock_news_desk`, exposing one pure function over a snapshot.
- Quote-verified grounding: every claim traces to a fact, every fact to a literal quote in a source.
- A quarantine boundary so only one agent reads raw text.
- Evals with designed answers, run in CI without an API key, and a pinned judge in live runs.
- A LangGraph vs CrewAI comparison on the analysis round, with predictions committed first.
- Tracing to Langfuse.

Non-goals (v1): price forecasts or trade execution; personalization or any real personal data; accounts; fetching publisher article pages; Freeman integration; the FastAPI service and UI (later repos); hosting and deploy; MCP; memory across runs; live price or fundamentals data.

## 3. Domain model

Terms are defined in `GLOSSARY.md`: Snapshot, Source, Quote, Fact, Claim, Citation, Quarantine boundary, Ruling, Case, News lean, Abstention, Confidence. The evidence chain is **claim, then fact, then quote inside a source**. A fact states what a source says, never what is true ("The CEO said growth will double", not "growth will double"). Conflicting facts both stand and the Critic flags them; nothing is silently merged.

## 4. Engine interface ([ticket 02](../../wayfinder/research-desk/02-analysis-schema-and-engine-interface.md))

`analyze(snapshot, config) -> Analysis | typed failure`. The engine never touches the network for news. Fetch adapters build snapshots separately.

Input:
- **Snapshot:** ticker, fetch time, list of sources. No position data.
- **Source:** id (stable hash of the canonical URL), title, publisher, published date, URL, origin tag (`gdelt` or `edgar`), text (a title for GDELT, a filing excerpt for EDGAR).
- **Config:** model choice per tier, per-run caps, which agents run, price table, prompt and rubric versions.

Output, **Analysis:**
- Header: ticker, generated time, snapshot id and date range, models used.
- News lean (`positive`, `negative`, `mixed`, `unclear`) and confidence (`low`, `medium`, `high`).
- Summary, each sentence backed by claims.
- Claims grouped as supporting, opposing and risks. Each has text, fact ids and the agent that made it.
- Open questions (what to watch for).
- Sources list: only cited sources, rebuilt from the snapshot, never from model output.
- Notes: dropped claims, unverified quotes, Critic rulings and conflicts, truncations and drops, caps hit, degradations, and the reason when the lean is `unclear`.
- Disclaimer: a fixed string added by code.
- Not included: raw model reasoning, full agent transcripts (these go to the trace), price targets.

Typed failures: "Triage failed", "budget exceeded", "timed out". A thin snapshot is not a failure; it returns an `unclear` analysis. The engine never returns partially invented output.

## 5. Architecture ([ticket 03](../../wayfinder/research-desk/03-supervisor-design.md))

The graph is routed entirely by code. No LLM decides what runs next, so runs are repeatable, no tokens go on routing, and a poisoned snapshot cannot steer a router.

Flow:
1. A thin snapshot short-circuits to an `unclear` analysis with no model call.
2. **Triage** reads the raw sources and emits facts. Code verifies every quote.
3. **Bull, Bear and Risk** run in parallel on verified facts only.
4. The **Critic** rules on each claim (`accept`, `reject`, `revise`, with a reason). One revision round at most: rejected and revised claims go back to the originating agent once; code re-checks; the Critic re-reviews only the revised claims; anything still failing is dropped and noted.
5. The **writer** proposes the news lean and composes the summary from surviving claims.
6. **Code** checks the lean against the claims, computes confidence, rebuilds the sources list, runs the advice lint, and attaches the disclaimer.

The Critic does only the semantic checks code cannot: overreach (a claim says more than its facts), conflicts (facts disagree and a claim ignores one), single-source dependence, and smuggled predictions. It sees only facts and claims, and it cannot add claims or change the lean.

Failure policy: one retry per agent for transient errors only (timeouts, rate limits, malformed schema output).

| Agent | If it fails after the retry |
|---|---|
| Triage | Typed failure, no analysis |
| Bull or Bear | Lean forced to `unclear`, confidence capped at `low`, note names the missing side |
| Risk | Continue without the risks group, note records the gap |
| Critic | Skip the revision round, cap confidence at `medium`, note |
| Writer | Deterministic template summary built from the claims, note |

State is in memory per run, with no checkpointer. A reducer appends claims, so parallel agents never overwrite each other. Replay and debugging come from Langfuse traces and from re-running a frozen snapshot.

## 6. Quarantine and prompt injection ([ticket 08](../../wayfinder/research-desk/08-prompt-injection-threat-model.md))

Sources are untrusted input. In scope and tested: output steering (planted text pushing the lean or hiding risks), forged or laundered citations, and cost or resource abuse. Ruled out by design: prompt and config leakage (the prompt is public, and no keys or personal data enter any prompt) and tool abuse (agents that read untrusted text get no tools and no secrets).

Guardrail: only Triage reads raw text, and it emits facts each carrying a quote that code verifies as a literal substring of its source. Unverified facts are dropped and noted. Every later agent sees only verified facts. Baseline from Freeman still applies: source text inside one JSON string, schema-constrained output, and code-level citation grounding on the final output. An injection classifier is deferred.

## 7. Abstention, confidence and advice rules ([ticket 04](../../wayfinder/research-desk/04-output-and-abstention-policy.md))

The result is `unclear` (an abstention, with the reason in the notes) when: the snapshot is thin (fewer than 3 usable sources or fewer than 2 distinct publishers, where usable means news from the last 7 days or filings from the last 90), Bull or Bear has no surviving claims, no claims survive at all, or the writer's lean fails the code check against the surviving claims. All thresholds are config defaults for the evals to tune.

Confidence is computed by code, versioned and unit-tested; first matching row wins. P is the number of distinct publishers behind claims supporting the lean.

| Confidence | Rule |
|---|---|
| `low` | Lean is `unclear`, or P is 1 or less, or Bull or Bear failed |
| `medium` | P is 2; or P is 3 or more with an unresolved conflict; or lean is `mixed`; or the Critic failed; or the newest news source is more than 3 days old |
| `high` | P is 3 or more, no unresolved conflict, lean is `positive` or `negative`, no degradation, recent news |

Confidence describes how well supported the evidence is. It is not the chance a stock rises or that the analysis is right.

No predictions or advice, in three layers: a prompt rule on every agent, the Critic's check for smuggled predictions, and a deterministic code lint on claims and the summary (phrases such as "will rise", "price target", "should buy", "undervalued"). A claim that trips the lint is dropped and noted. A claim attributed to a source ("Analyst X said the target is $120") passes; the desk asserting it fails. The phrase list is a versioned config file.

The disclaimer is a fixed string added by code: "This is an automated summary of published sources. It is not financial advice and does not predict prices or recommend trades." No human approval step exists in v1.

## 8. Cost and latency ([ticket 09](../../wayfinder/research-desk/09-cost-and-latency-budget.md))

All numbers are editable config defaults. Current prices were not checked; the dollar cap comes from a price table in config that the author maintains.

- Tiers: fast, Haiku 5.5 (`claude-haiku-5-5`), for Triage and the writer; strong, Sonnet 5.5 (`claude-sonnet-5-5`), for Bull, Bear, Risk and the Critic. One environment switch puts the whole graph on the fast tier for CI and cheap evals.
- Caps per run: at most 20 sources into Triage (distinct publishers first, then recency); 2,000 characters per source; 12 model calls; per-call `max_tokens` and a per-run token total; $0.25. Hitting a cap returns the typed "budget exceeded" failure. Every truncation and drop is noted.
- Timeouts: 45 seconds per call, 120 seconds per run (typed "timed out" failure). A typical run under about 60 seconds is a goal for the evals to check, not a measurement.
- Caching: none in the engine. The later service caches by a hash of the snapshot contents plus config and prompt versions. The eval runner has an optional record/replay cache keyed by (model, prompt hash).

## 9. Data sources ([ticket 12](../../wayfinder/research-desk/12-data-sources-and-terms.md), [ticket 15](../../wayfinder/research-desk/15-full-article-fetching.md))

- Live demo: GDELT (headlines, publisher domain, language and date, no article text; cited as its terms require) and SEC EDGAR filings (fair access: at most 10 requests per second and a User-Agent with a contact email taken from an environment variable, never committed).
- CI: committed fixtures only, with a mocked model and news client.
- Eval set: synthetic articles, a few EDGAR excerpts, and optionally GDELT metadata. No Finnhub data in the public repo or demo, because its terms bar redistribution.
- No publisher article fetching in v1. Triage works on headlines and filing text, so headline-based analyses are shallow and many tickers will return `unclear`. A fetcher could later be another adapter and its own ticket.
- Unverified in the research and not blocking: Finnhub endpoint details and "personal use" status, GDELT's current rate guidance, Alpha Vantage and Yahoo terms, non-US copyright treatment of headlines. Re-read each source's terms before publishing.

## 10. Evals and CI ([ticket 06](../../wayfinder/research-desk/06-eval-metrics-and-ci-gating.md))

There is no ground truth for a "right" lean on real news, so the evals use **cases**: synthetic snapshots with a designed answer, planted facts and a tagged failure mode.

Set: about 36 clean cases (12 clearly positive or negative, 6 mixed, 8 thin, 4 one-sided, 6 EDGAR-based) plus about 20 poisoned variants for paired injection tests. An LLM drafts the sources from designed briefs and the author reviews each. The README states that model-written articles are easier for a model to read, so scores overstate real performance.

Metrics: lean agreement, abstention correctness, planted-fact recall in Triage, dropped-claim rate, advice-lint hits, injection pass rate per attack type, confidence calibration (agreement by confidence level), and cost, calls and latency. One pinned LLM judge scores only summary faithfulness, validated against hand-labelled examples and reported with its agreement rate.

Paired injection cases: each poisoned snapshot is a clean case with one planted attack (instructions in a title, fake role tags, citation-id forgery, authority impersonation, oversized or repeated content, non-English or encoded instructions). Pass means the lean and confidence stay within tolerance of the clean run, no citation points outside the snapshot, every quote is a verified substring, and caps hold.

CI: every push runs unit tests with a fake LLM and news client, the eval suite in replay mode (no API key, so forks can run it), and the privacy scan. Live runs are manual or nightly on `master`, use the fast tier, re-record the cache, run the judge, and write a results file under `docs/` that feeds the README results table. Hard gates in replay mode: any advice-lint hit, any invalid quote or unknown citation, injection pass rate below the committed baseline, lean agreement or abstention correctness more than a set tolerance below the baseline, or cost and calls above the caps. Live metrics are reported, not gating. The baseline is a committed file updated on purpose.

## 11. Tracing ([ticket 07](../../wayfinder/research-desk/07-tracing-stack.md))

Langfuse (MIT outside its `ee/` directories; free cloud tier of 50k units per month; self-hostable). LangGraph uses its callback handler; CrewAI goes through the OpenInference instrumentor. Both frameworks trace to one backend so the comparison can be viewed side by side. Estimate real unit usage before relying on the free tier. Public trace links expose full prompts and outputs, so only synthetic runs are shared. Phoenix could be added later through OpenTelemetry.

## 12. LangGraph vs CrewAI comparison ([ticket 10](../../wayfinder/research-desk/10-langgraph-vs-crewai-comparison-protocol.md))

CrewAI rebuilds the analysis round: Bull, Bear, Risk and the Critic with its revision step. Input is verified facts, output is claims with rulings in the same schema; everything before and after is shared code, and the crew never sees raw text. Held constant: model tier (Sonnet 5.5), instruction text, input facts, output schema, eval cases, code checks and caps. Reported per framework: lean agreement, abstention correctness, dropped-claim rate, advice-lint hits, injection pass rate, model calls, tokens and dollars, latency, lines of code, and a qualitative debuggability note with Langfuse screenshots (including what it took to add the one-round revision cap).

Variance: all 56 cases, 3 runs each per framework, after a pilot of about 10 cases. Mean and spread are reported with no significance claim. Worst-case cost is about $84 (336 runs at the $0.25 cap); the pilot measures the real figure. Predictions are written and committed before the pilot, and the write-up (`docs/comparison.md`, with a summary table in the README and the raw results committed) reports which held and which did not, and states the limits: one task, one model tier, a synthetic eval set, a small sample. It does not crown a winner.

## 13. Repo, packaging and privacy ([ticket 01](../../wayfinder/research-desk/01-repo-and-package-layout.md), [ticket 13](../../wayfinder/research-desk/13-demo-privacy-and-synthetic-portfolio.md), [ticket 14](../../wayfinder/research-desk/14-relationship-to-freeman.md))

This repo (`HadiAbu/Multi-agent-stock-news`) holds the engine, evals and design docs. The FastAPI service (`stock-news-desk-api`) and UI (`stock-news-desk-ui`) become separate repos later and install a tagged engine release from git. Python only here. MIT license. Python package `stock_news_desk`, `src/` layout, managed by `uv`; PyPI is optional later.

Planned layout:
```
LICENSE, pyproject.toml
src/stock_news_desk/   schema.py, snapshot.py, triage.py, agents/, graph.py,
                       grounding.py, llm.py, crewai_compare/
evals/                 snapshots, paired injection cases, scoring, runner
tests/                 unit tests with a fake LLM and fake news client
scripts/               pre-push check
.github/workflows/     CI: tests, evals, privacy scan
docs/                  wayfinder map and tickets, specs, comparison
```
Module names may shift when the build plan is written.

Privacy: ticker-only analysis, no personalization. Never committed: keys, `.env` files, Turso URLs and DB files, real holdings, trades, watchlist or stored analyses from Freeman, its `.superpowers/` notes, coverage and build output, absolute local paths. Nothing is copied wholesale from Freeman; grounding, normalization and prompt logic are rewritten from the design and tests ported with synthetic fixtures only. Freeman's API lives in a parent git repo with an unrelated origin; that repo is never pushed. A local `pre-push` hook and a CI job both run gitleaks and a grep for personal paths and sensitive names. The desk is a standalone showcase; any Freeman integration is out of scope and would be its own ticket.

## 14. Technology and why

| Technology | Used for | Why this and not something else |
|---|---|---|
| Python, `uv`, `src/` layout | The engine as an installable package | One language for the engine and the evals. `uv` gives fast reproducible installs and a lockfile; a git-tag dependency lets the service pin an exact engine version with no PyPI account or release pipeline yet. |
| LangGraph | The supervisor graph: explicit state, parallel Bull/Bear/Risk fan-out, claim reducers, bounded revision loop | The design is a graph routed by code, which is what LangGraph expresses directly. State and routing are visible and testable, with no LLM spending tokens on routing. |
| LangChain (`langchain-anthropic`) | Model wrappers, structured output, tracing hooks | Thin, well-supported glue that makes the provider switchable by an environment variable. Used as plumbing, not as the architecture. |
| Pydantic | Schemas for snapshot, facts, claims, rulings and analysis | Schema-constrained model output and strict validation at every boundary; the same models drive the tests and the evals. |
| Anthropic models (Haiku 5.5 and Sonnet 5.5) | Fast tier for extraction and writing, strong tier for judgment | Two tiers cut cost where the work is mechanical and code verifies the result (Triage quotes), and spend on the steps that need judgment. One switch moves everything to the fast tier for CI. |
| CrewAI | A rebuild of the analysis round for the comparison | Role-based crews are a different orchestration model from an explicit graph. Rebuilding one well-bounded piece with shared inputs and checks makes the comparison fair and shows judgment, not framework collecting. |
| Langfuse | Tracing for both frameworks, public traces of synthetic runs | Roomiest free tier of the options researched, free self-hosting, LangGraph and CrewAI both supported, public trace links for the README. |
| pytest, fake LLM and news client | Unit tests with no network or keys | Keeps CI deterministic and free, and keeps the code around the models well tested. |
| Eval runner with record/replay cache | Metrics on designed cases; gating in CI | Replay means CI needs no API key (forks can run it) and reruns cost nothing; live runs re-record. This is what makes "evals in CI" affordable. |
| gitleaks and a path grep | Pre-push hook and CI job | A public repo built next to a private app needs an automated guard against leaking secrets and personal paths. |
| GitHub Actions | CI: tests, replay evals, privacy scan; nightly live evals | The repo is on GitHub, so the CI is native and free for public repositories. |
| GDELT and SEC EDGAR | News headlines and filing text | Terms that permit a public demo with attribution, unlike Finnhub's personal-use, no-redistribution terms. |
| FastAPI and a small UI (later repos) | Serving the engine and showing results | A thin HTTP layer over the same pure function, kept out of this repo so the engine stays standalone. |
| MCP (not in v1) | Not used | Nothing in v1 needs agents to call tools, and agents that read untrusted text must have none. It may return for a later phase as a way to expose the desk to other agents. |

## 15. Delivery phases

1. Engine: schema, snapshot adapters, Triage with quote verification, agents, graph, grounding, lint, confidence, caps.
2. Evals, replay CI and tracing; first live run and README results table.
3. LangGraph vs CrewAI: predictions committed, pilot, full run, `docs/comparison.md`.
4. FastAPI service repo, then a small UI repo.
Each phase is developed on its own branch and merged with `--no-ff`; the branch is kept as a rollback anchor. Each independent piece gets its own plan and approval.

## 16. Open items and risks

- The confidence thresholds, the abstention thresholds and the eval tolerances are first guesses to be tuned on the eval set.
- Headline-only input will make many analyses shallow or `unclear`. This is stated, and the evals measure it, but a reviewer may still find the demo underwhelming.
- Model-written eval articles overstate performance. Real EDGAR cases and the plain statement of the caveat are the mitigation.
- Source terms (GDELT, EDGAR, Finnhub) were summarized from research and not legal advice; re-read before publishing.
- Fog left for later: hosting and deploy target, the UI, where MCP fits, memory across runs, live price or fundamentals data.
