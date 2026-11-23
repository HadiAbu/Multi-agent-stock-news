# Output and abstention policy

Type: wayfinder:grilling
Blocked by: Analysis schema and engine interface
Status: CLOSED 2026-11-23

## Question
What does the desk say and when does it refuse? Decide when the answer is "unclear" (thin news, one-sided support, critic rejection), how citations are required and enforced in code (as Freeman's grounding does), confidence rules, the not-financial-advice framing, the ban on price predictions, and whether any step needs human approval.

## Resolution
**Abstention (the result is `unclear`) when any of these holds:**
- The snapshot is thin: fewer than 3 usable sources, or fewer than 2 distinct publishers. Usable means news from the last 7 days or filings from the last 90 days. This returns `unclear` with the reason and makes no model call.
- Bull or Bear has no surviving claims (one-sided support).
- After the Critic and the code checks, no claims survive.
- The lean proposed by the writer fails the code check against the surviving claims.
All thresholds are config defaults for the evals to tune.

**Citations.** Enforced in code, as in the schema ticket: every claim must cite facts, every fact must carry a quote that appears literally in its source, and anything failing is dropped and noted.

**Confidence rubric (computed by code, versioned, unit-tested, and explained in the notes).** Inputs: P is the number of distinct publishers behind claims supporting the lean; an unresolved conflict is a Critic-flagged conflict that survived the revision round; a degraded run is any agent failure. Rules are checked in order, first match wins:

| Confidence | Rule |
|---|---|
| `low` | Lean is `unclear`, or P is 1 or less, or Bull or Bear failed |
| `medium` | P is 2; or P is 3 or more with an unresolved conflict; or lean is `mixed` (capped at medium); or the Critic failed (capped); or the newest news source is more than 3 days old |
| `high` | P is 3 or more, no unresolved conflict, lean is `positive` or `negative`, no degradation, recent news |

Confidence describes how well supported the evidence is (independent sources, conflicts, freshness). It is not the chance that a stock rises or that the analysis is right. `high` is meant to be rare so evals can check it holds up better than `medium`. Thresholds are a first guess to be tuned on the eval set.

**No predictions, no advice: three layers.**
1. Prompt rule on every agent: describe what sources say, never forecast or advise.
2. The Critic checks for smuggled predictions.
3. A deterministic code lint on claims and the summary catches forecasting and advice phrasing (such as "will rise", "price target", "should buy", "undervalued"). A claim that trips it is dropped and noted. The lint targets the desk's own voice: a claim attributed to a source ("Analyst X said the target is $120") passes, and an unattributed assertion ("the stock will reach $120") fails. The phrase list is a versioned config file that grows as evals find gaps.

**Disclaimer.** A fixed string added by code, never by a model: "This is an automated summary of published sources. It is not financial advice and does not predict prices or recommend trades."

**Human approval.** None in v1. Nothing is published automatically; a person reads each output. A checkpointer and approval step could return if publishing is ever added.
