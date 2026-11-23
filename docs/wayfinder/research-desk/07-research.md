# 07 Research: Tracing tool (LangSmith vs Langfuse vs OTel/Phoenix)

Checked 2026-11-23. Pricing and limits change often; re-verify before committing.

## Recommendation

**Primary: Langfuse.** Its free cloud tier is the roomiest for CI evals (50k units/month), it can be self-hosted for free, it has a one-call LangChain/LangGraph callback handler, and it covers CrewAI through the OpenInference instrumentor. It also supports public trace links, which a README needs.

**Second tool: not worth adding now.** Because Langfuse ingests OpenTelemetry spans, the CrewAI path already uses the OTel standard. That leaves you free to point the same spans at Phoenix later, so no second tool is needed. LangSmith is the strongest fit for LangGraph but its free tier is the tightest. Revisit it only if you want its LangGraph-specific UI.

## Comparison

| | LangSmith | Langfuse | OTel + Phoenix (OpenInference) |
|---|---|---|---|
| Free tier | Developer: 5,000 base traces/mo, 14-day retention, 1 seat, pay-as-you-go beyond that [1] | Hobby: 50k units/mo, 30 days data access, 2 users [2] | Self-hosted Phoenix: no usage limits or feature gates [6]. Phoenix cloud free tier: not verified |
| Self-hosting | Not on free tier; Enterprise only [1] | Yes, free [2]. License not confirmed on the pricing page | Yes: Docker, Compose, Kubernetes, Helm, Railway, Cloud Run and others [6] |
| LangGraph | Native: env vars only, LangChain not required [3] | `CallbackHandler` works for LangGraph agents [4] | Phoenix lists LangChain auto-instrumentation [5]. LangGraph is not named on the page I fetched, so unverified |
| CrewAI | Via OpenTelemetry instrumentors, not a native callback [7] | Via OpenInference `CrewAIInstrumentor`, exported as OTel spans [8] | OpenInference/OTel route. CrewAI is not listed on the Phoenix page I fetched, so unverified [5] |
| Anthropic | Not documented on the LangGraph page; the page says "any LLM provider" [3] | Not verified on the pages fetched | Listed in Phoenix's provider auto-instrumentation [5] |
| Public trace link | Yes: Share, then copy link; no login needed, no plan restriction mentioned [9] | Yes: a trace can be made public. Via SDK it is one-way, and the whole trace becomes public [10] | Not covered by the Phoenix pages I fetched. Unverified |
| Setup effort | Lowest for LangGraph (env vars). CrewAI needs OTel wiring | Low: install, keys, one handler. CrewAI needs a few lines [8] | Highest: you run and host the backend and wire instrumentors |

## Notes for this project

- **CI evals:** 5,000 traces/mo with 14-day retention (LangSmith) is easy to burn through, because every eval case times every agent run adds traces. Langfuse's 50k units is roughly 10x larger on paper. Units and traces are not the same measure, and one trace can use several units. Estimate real usage before relying on it.
- **Fairness of the framework comparison:** the LangGraph and CrewAI runs should go to the same backend. Langfuse supports both, so a side-by-side comparison in one UI is possible.
- **README screenshots:** public links need a cloud-hosted instance, or a self-hosted one that is reachable from the internet. LangSmith notes that on a self-hosted VPC deployment only authenticated VPC members can open shared links [9].
- **Privacy:** public links expose full prompts and code diffs. Use only synthetic or open-source PRs for shared traces.

## Gaps (could not verify)

- Langfuse license (the pricing page does not name it).
- Anthropic support for Langfuse and LangSmith on the pages fetched.
- Phoenix cloud free tier, public sharing, and named LangGraph/CrewAI support.
- Whether the Langfuse public-sharing docs page URL still resolves (the one I tried returned 404; details came from a search summary [10]).

## Sources

1. https://www.langchain.com/pricing
2. https://langfuse.com/pricing
3. https://docs.langchain.com/langsmith/trace-with-langgraph
4. https://langfuse.com/integrations/frameworks/langchain
5. https://arize.com/docs/phoenix
6. https://arize.com/docs/phoenix/self-hosting
7. https://docs.langchain.com/langsmith/trace-with-crewai (via search summary; page not fully read)
8. https://langfuse.com/integrations/frameworks/crewai
9. https://docs.langchain.com/langsmith/share-trace
10. https://langfuse.com/docs/observability/features/url (via search summary; page not fully read)
