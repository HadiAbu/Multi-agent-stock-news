# Tracing stack

Type: wayfinder:research (AFK)
Blocked by: none
Status: CLOSED 2026-11-23

## Question
Which tracing and observability tool fits: LangSmith, Langfuse, or OpenTelemetry-based (for example OpenLIT, OpenInference with Phoenix)? Compare free-tier limits, self-hosting, LangGraph and CrewAI support, and how well each shows up in a public README (screenshots, shareable traces). Recommend one.

## Resolution
Research is in `07-research.md` (checked 2026-11-23). The author chose **Langfuse**.

- Free cloud tier of 50k units per month with 30-day access, free self-hosting, a LangGraph callback handler, CrewAI through the OpenInference instrumentor, and public trace links.
- License verified separately: MIT outside the `ee/` directories, which carry a separate enterprise license. Anthropic support is documented for the Python and JS SDKs.
- Both the LangGraph and CrewAI runs go to the same backend, so the framework comparison can be viewed side by side.
- No second tool now. Langfuse accepts OpenTelemetry spans, so Phoenix could be added later.
- Cautions: 50k units are not 50k traces, so estimate real usage (eval cases times agent runs) before relying on the free tier; the record/replay cache in the cost ticket reduces live runs. Public trace links expose full prompts and outputs, so share only synthetic runs, never anything with real data or keys.
- Wording note: the research note mentions "code diffs" and "PRs" in its privacy line, left over from the earlier code-review idea; for this project it means prompts and outputs.

**Still unverified, not blocking:** Anthropic support on LangSmith, Phoenix cloud free tier and sharing, and the Langfuse public-sharing docs page (404 when fetched; details came from search summaries).
