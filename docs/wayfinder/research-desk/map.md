# Map: multi-agent-research-desk

## Destination
A design spec for an open-source "research desk". Given a ticker and a news snapshot, a LangGraph supervisor runs News-triage, Bull, Bear, Risk and Critic agents, then a writer produces a cited, abstention-aware, not-financial-advice analysis. It comes with evals, tracing, prompt-injection guardrails, a LangGraph vs CrewAI comparison, and a FastAPI service with a small UI later. No price predictions. The spec also explains each technology and why it was chosen.

## Notes
- Goal: portfolio project that shows employers real capability with LangChain/LangGraph, CrewAI and production practices. Original code only; do not copy the mkabajah workshop.
- Domain comes from the author's Freeman tracker, which already has a single-call news analysis (Finnhub + GDELT news, one LLM call, schema-constrained output, citation grounding in code, thin-news rule, cache, daily cap, disclaimer). The research desk extends that design with separate agents and real evals. Safe to reuse: pipeline design, prompt, schemas, grounding logic, tests with synthetic fixtures.
- Default model: Anthropic via `langchain-anthropic`, switchable by env var (Freeman itself uses OpenAI); cheaper model for CI evals.
- Phasing: engine, then FastAPI service, then small UI/dashboard.
- Standing request: later in the session, explain what each technology is used for and why (also goes into the final spec).
- Pushing to any remote needs per-instance approval from the user.
- PRIVACY: never commit keys, `.env` files, Turso URLs or DB files, real holdings, trades, watchlist, or stored analyses from Freeman, nor its `.superpowers/` notes, coverage or build output. The demo uses a synthetic portfolio. Freeman's API lives in a parent git repo whose origin is an unrelated project (Scalable-Scrapper); never push that repo. Extract design and tests into this repo instead.

## Decisions so far
- Domain: Freeman-style stock news research desk (switched from a PR code-review agent on 2026-11-23).
- v1 agents: News-triage, Bull, Bear, Risk, Critic, plus a writer.
- Pipeline in LangGraph; CrewAI rebuilds one agent (likely the Bull/Bear debate) for a head-to-head comparison.
- Default model Anthropic, env-switchable.
- Delivery: engine as a standalone package, then a FastAPI service, then a small UI.
- Tracing candidate: Langfuse (MIT outside `ee/`, free tier, LangGraph and CrewAI support); see `07-research.md`. Not yet closed.
- [Demo privacy](13-demo-privacy-and-synthetic-portfolio.md) (closed): no personalization in v1, ticker-only analysis; fixed never-copy list; pre-push check is a local hook plus a CI job running gitleaks and a personal-path grep.
- [Prompt-injection threat model](08-prompt-injection-threat-model.md) (closed): steering, forged citations and cost abuse are tested; leakage and tool abuse are ruled out by design (no tools or secrets for agents reading untrusted text). Only News-triage reads raw text and emits quote-verified facts; other agents see only those. Evals use about 20 paired clean/poisoned cases with code-checked pass criteria.
- Data plan candidate, from `12-research.md` (not yet closed, awaiting the author's read): do not commit Finnhub data (its terms bar redistributing data or derived results); use GDELT with citation and SEC EDGAR for the live demo; evals use synthetic or self-written articles, EDGAR excerpts and optionally GDELT title/URL/date metadata; CI uses fixtures and mocks, no live calls. Several points unverified in the note.

## Not yet specified
Hosting and deploy target, the UI, where MCP fits, memory across runs, live price or fundamentals data.

## Out of scope
- Price forecasts or trade execution.
- Using real holdings or personal data in the public demo.
- Accounts and multi-user support.
- Personalization ("use my position") in v1; may return later with a synthetic portfolio.
- The earlier PR code-review idea (archived in `archive/`).

## Open tickets
See the `NN-*.md` files in this folder; each lists its type and what blocks it. Frontier: 01, 07, 12 (research done, awaiting review). Closed: 08, 13.

## Archive
`archive/` holds the retired PR-review tickets 05 (eval datasets) and 11 (GitHub Action constraints) with their research notes. Their licence findings were verified on 2026-11-23: CVEfixes source MIT and data CC BY 4.0, BugsJS, SWE-bench and AgentDojo MIT, BugsInPy has no licence.
