# Supervisor design

Type: wayfinder:grilling
Blocked by: Analysis schema and engine interface
Status: CLOSED 2026-11-23

## Question
How does the LangGraph supervisor work with the News-triage, Bull, Bear, Risk and Critic agents and a writer? Decide the graph state, which steps run in parallel (Bull and Bear), what the Critic checks and whether it can send work back, how claims are merged, and how a failed or timed-out agent is handled.

## Resolution
**Routing: a graph routed entirely by code.** No LLM decides what runs next, so runs are repeatable, no tokens go on routing, and a poisoned snapshot cannot steer a router. An LLM-supervisor variant may appear as an experiment in the framework comparison, not in the main design.

**Flow.**
1. A thin snapshot short-circuits straight to an `unclear` analysis with no model call.
2. Triage reads the raw sources and emits facts. Code verifies every quote (the quarantine boundary).
3. Bull, Bear and Risk run in parallel, seeing only verified facts.
4. The Critic rules on each claim. At most one revision round (see below).
5. The writer proposes the news lean and composes the summary from surviving claims.
6. Code checks the lean against rules (for example `positive` needs surviving supporting claims, a missing side forces `unclear`), computes confidence, rebuilds the sources list, and attaches the disclaimer.

**Critic.** Does only the semantic checks code cannot: overreach (a claim says more than its facts), conflicts (facts disagree and a claim ignores one), single-source dependence, and smuggled predictions. It sees only facts and claims, never raw text. It returns a ruling per claim: `accept`, `reject` or `revise`, each with a reason code and short note. It cannot add claims or change the lean. Rejected and revised claims go back to the originating agent once with the reason; code re-checks the results, the Critic re-reviews only the revised claims, and anything still failing is dropped and noted. No second loop. Rulings appear in the analysis notes.

**Who sets the lean.** The writer proposes it from the surviving claims and code checks it, so one place is accountable for the headline. Confidence stays computed by code.

**Failure policy (graceful), one retry per agent for transient errors only (timeouts, rate limits, malformed schema output); no retry for content problems.**

| Agent | If it fails after the retry |
|---|---|
| Triage | Typed failure, no analysis |
| Bull or Bear | Lean forced to `unclear`, confidence capped at `low`, note names the missing side |
| Risk | Continue without the risks group, note records the gap |
| Critic | Skip the revision round, cap confidence at `medium`, note |
| Writer | Fall back to a deterministic template summary built from the claims, note |

Hitting the per-run token or call cap stops the graph with a typed "budget exceeded" failure, never partial output. Every degradation appears in the analysis notes and as a trace event.

**State.** In memory per run, with no checkpointer. The graph state holds the snapshot, verified facts, claims (each tagged with its agent), the Critic's rulings, notes and a budget counter. A reducer appends claims so parallel agents never overwrite each other. Debugging and replay come from Langfuse traces and re-running a frozen snapshot. A SQLite checkpointer could return later if a human-approval step is added.

**Left to other tickets.** Exact abstention thresholds (output and abstention policy), budget numbers (cost and latency), the agent prompts, and which agent is rebuilt in CrewAI (comparison protocol).
