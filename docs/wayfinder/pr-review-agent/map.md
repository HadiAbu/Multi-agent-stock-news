# Map: multi-agent-review-system

## Destination
A design spec for an open-source PR review agent. It reviews Python and TypeScript PRs with a LangGraph supervisor over Bugs, Security and Tests specialist agents, shipped as a standalone engine plus a GitHub Action. It includes evals, tracing, guardrails, and a LangGraph vs CrewAI comparison of the Tests reviewer. The spec also explains each technology and why it was chosen.

## Notes
- Goal: portfolio project that shows employers real capability with LangChain/LangGraph, CrewAI and production practices. Original code only; do not copy the mkabajah workshop.
- Default model: Anthropic via `langchain-anthropic`, switchable by env var; cheaper model for CI evals.
- Phasing: engine, then GitHub Action, then GitHub App service, then web dashboard.
- Standing request: later in the session, explain what each technology is used for and why (also goes into the final spec).
- Pushing to any remote needs per-instance approval from the user.

## Decisions so far
- Scope: GitHub PR reviewer first, web dashboard as a later phase.
- v1 specialists: Bugs, Security, Tests.
- Languages: Python and TypeScript only.
- Second framework: CrewAI, used to rebuild the Tests reviewer for a head-to-head comparison against LangGraph.
- Trigger: GitHub Action first, with the engine as a standalone package, then a GitHub App service wrapping the same engine.

## Not yet specified
The GitHub App service (webhooks, queueing, hosting), the dashboard, the deploy target, where MCP fits, memory across PRs.

## Out of scope
- Languages other than Python and TypeScript (v1 focus).
- GitLab and other hosts (GitHub only).
- Auto-fixing or committing code (review only, no writes to the PR branch).
- A Style reviewer (low signal, noisy; possible later addition).

## Open tickets
See the `NN-*.md` files in this folder; each lists its type and what blocks it. Frontier at charting time: 01, 05, 07, 08, 11.
