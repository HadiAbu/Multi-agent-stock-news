# Repo and package layout

Type: wayfinder:grilling
Blocked by: none
Status: CLOSED 2026-11-23

## Question
How is the repo laid out so the engine, the FastAPI service and the later UI stay separate? Decide the license, the package structure (engine as an installable package versus app code), the Python/TypeScript tooling split, and how design and tests are extracted from Freeman's API without pulling in its parent git repo, its secrets or its real data.

## Resolution
- **Repos:** the engine, evals and design docs live in this repo (`HadiAbu/Multi-agent-stock-news`). The FastAPI service and the UI become separate repos later (`stock-news-desk-api`, `stock-news-desk-ui`). Chosen over a monorepo; the cost is more repos to keep in sync.
- **Packaging:** a Python package with a `src/` layout, managed by `uv`. The service installs a tagged release from git (`engine @ git+https://github.com/HadiAbu/Multi-agent-stock-news@vX.Y.Z`). PyPI is an optional later step.
- **Tooling split:** this repo is Python only. TypeScript lives in the UI repo.
- **License:** MIT for the code. Committed eval data keeps its own terms (see the data sources ticket).
- **Python package name:** `stock_news_desk`.
- **Extraction from Freeman:** nothing is copied wholesale. Rewrite the grounding, normalization and prompt logic from the design; port tests using synthetic fixtures only; run the privacy scan over everything before it is committed.

### Planned layout
```
Multi-agent-stock-news/
  LICENSE                 MIT
  pyproject.toml          uv-managed, src layout
  src/stock_news_desk/
    schema.py             analysis schema
    snapshot.py           news snapshot types and normalization
    triage.py             quote-verified fact extraction
    agents/               bull, bear, risk, critic, writer
    graph.py              LangGraph supervisor
    grounding.py          code-level citation checks
    llm.py                provider switch (Anthropic default)
    crewai_compare/       the CrewAI rebuild for the comparison
  evals/                  snapshots, paired injection cases, scoring, runner
  tests/                  unit tests with a fake LLM and fake news client
  scripts/                pre-push check (gitleaks plus path grep)
  .github/workflows/      CI: tests, evals, privacy scan
  docs/                   wayfinder map, tickets, the spec
```
The layout is a plan. Module names may shift when the schema and supervisor tickets close.
