# Supervisor design

Type: wayfinder:grilling
Blocked by: Analysis schema and engine interface

## Question
How does the LangGraph supervisor work with the News-triage, Bull, Bear, Risk and Critic agents and a writer? Decide the graph state, which steps run in parallel (Bull and Bear), what the Critic checks and whether it can send work back, how claims are merged, and how a failed or timed-out agent is handled.
