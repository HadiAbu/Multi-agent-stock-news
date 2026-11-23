# Relationship to Freeman

Type: wayfinder:grilling
Blocked by: Repo and package layout
Status: CLOSED 2026-11-23

## Question
Is the research desk a standalone showcase that only shares the domain and design, or the engine that Freeman later calls instead of its single LLM call? If it later plugs in, what contract must hold (the analysis schema, API key auth, the daily cap, caching) and what stays out of scope for v1?

## Resolution
**A standalone showcase that shares only the domain.** Nothing in v1 is designed around Freeman. The two systems differ on purpose:
- Vocabulary: Freeman's app renders a buy / hold / sell stance; the desk outputs a news lean (positive, negative, mixed, unclear).
- Data and model: Freeman uses Finnhub and OpenAI and personalizes with a real position; the desk is ticker-only, defaults to Anthropic, and uses GDELT and SEC EDGAR in the public demo.
- Home: Freeman's API lives in a parent git repo with an unrelated origin that must never be pushed; the desk is a clean public repo.

**Out of scope for v1:** any integration with Freeman, including keeping the output schema compatible with it. If integration is ever wanted, it becomes its own ticket: a thin adapter would build snapshots from Freeman's own private data sources, call the desk service, and map the news lean to what the app can show. The app's screens, auth, cap and cache would all need their own design at that point.

Freeman's design stays reusable as reference only, under the extraction rules in the repo-layout and privacy tickets.
