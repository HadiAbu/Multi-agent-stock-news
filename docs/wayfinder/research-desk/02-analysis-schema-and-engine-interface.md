# Analysis schema and engine interface

Type: wayfinder:grilling (use domain-modeling for vocabulary)
Blocked by: Repo and package layout
Status: CLOSED 2026-11-23

## Question
What is the engine's contract? Define the input (ticker, a news snapshot of normalized items with stable ids, optional position info) and the output. Freeman's output is stance (buy/hold/sell/unclear), confidence, summary, cited case for and against, and a watch-for list. What changes with multiple agents (per-agent findings, critic verdicts, dropped-claim notes, trace ids), and how are claims and citations identified so grounding and evals can check them?

## Resolution
Vocabulary is fixed in `GLOSSARY.md` (Snapshot, Source, Quote, Fact, Claim, Citation, News lean, Confidence). Field names below follow it.

**Headline verdict.** A *news lean* of `positive`, `negative`, `mixed` or `unclear`, describing what the cited sources together suggest about the near-term outlook. Freeman's buy/hold/sell is dropped, because it contradicts the not-financial-advice promise. Terms like recommendation, rating and signal are banned from the schema and the docs.

**Chain of evidence.** claim, then fact, then quote inside a source. Code checks each link: fact ids exist and each quote is a literal substring of its source. A claim without a valid chain is dropped and noted. A fact states what a source says, never what is true (attribution is part of the fact). Conflicting facts both stand and are flagged by the Critic; nothing is silently merged.

**Input contract (pure function).** `analyze(snapshot, config) -> Analysis | typed failure`.
- Snapshot: ticker, fetch time, list of sources. No position data (no personalization in v1).
- Source: id (stable hash of the canonical URL), title, publisher, published date, URL, origin tag (such as `gdelt` or `edgar`), text (a title for GDELT, a filing excerpt for EDGAR).
- Config: model choice, per-run token and call cap, which agents run.
- The engine never touches the network for news. Fetching is a separate adapter layer that builds snapshots. Evals and CI feed frozen snapshots. Typed failures include "snapshot too thin" and "budget exceeded"; the engine never returns partially invented output.

**Output contract.**
- Header: ticker, generated time, snapshot id and date range, model(s) used.
- News lean and confidence.
- Summary, with each sentence backed by claims.
- Claims grouped as supporting, opposing and risks. Each claim has text, fact ids and the agent that made it (per-agent attribution is exposed).
- Open questions (what to watch for).
- Sources list: only cited sources, rebuilt from the snapshot, never taken from model output.
- Notes: dropped claims, unverified quotes, conflicts flagged by the Critic, and the reason when the lean is `unclear`.
- Disclaimer: a fixed string added by code.
- Not in the output: raw model reasoning, full agent transcripts (these go to the trace), price targets.

**Confidence.** `low`, `medium` or `high`, computed by code, never reported by a model. Inputs: the number of independent publishers behind the lean (several facts from one outlet count once), whether the Critic flagged unsupported or conflicting claims, how thin or stale the snapshot is, and whether Bull and Bear were both well supported. Rules are written down, versioned and tested; `unclear` falls out of the same rules. Thresholds start simple and are tuned against the eval set. The evals should check that high confidence is right more often than low.

**Left to other tickets.** Exact abstention thresholds (output and abstention policy), the agent graph (supervisor design), metric definitions (eval metrics), budget numbers (cost and latency).
