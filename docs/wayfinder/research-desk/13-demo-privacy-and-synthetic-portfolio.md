# Demo privacy and synthetic portfolio

Type: wayfinder:grilling
Blocked by: none
Status: CLOSED 2026-11-23

## Question
How does the public repo stay free of the author's real data? Decide the synthetic portfolio used in demos and evals, whether personalization ("use my position") is in v1 at all, which Freeman artifacts are never copied (keys, `.env`, Turso URLs and DB files, real holdings and stored analyses, `.superpowers/` notes), and a pre-push check (secret scan, path scan) that runs before any push.

## Resolution
- **No personalization in v1.** The desk analyzes a ticker only. No position data enters prompts, traces or evals. This removes the leak risk and the need for a synthetic portfolio. If personalization is added later, it uses a synthetic portfolio only and gets its own ticket.
- **Never copied from Freeman:** keys, `.env` and `.env.local`, Turso URLs and tokens, `*.db` files, real holdings, trades, watchlist and stored analyses, `.superpowers/` notes, coverage and build output, and absolute local paths. Reusable: pipeline design, prompt, schemas, grounding logic, and tests that use synthetic fixtures.
- **Pre-push check, in two places:**
  1. A local `pre-push` git hook.
  2. A CI job that must pass before merge.
  Both run a secret scanner (gitleaks) and a grep for personal paths and sensitive names (the author's home path, `.env`, Turso URLs, `.superpowers`). Either failing blocks the push or the merge.
- Implementation of the hook and CI job belongs to the build plan, not to this decision.
