# Analysis schema and engine interface

Type: wayfinder:grilling (use domain-modeling for vocabulary)
Blocked by: Repo and package layout

## Question
What is the engine's contract? Define the input (ticker, a news snapshot of normalized items with stable ids, optional position info) and the output. Freeman's output is stance (buy/hold/sell/unclear), confidence, summary, cited case for and against, and a watch-for list. What changes with multiple agents (per-agent findings, critic verdicts, dropped-claim notes, trace ids), and how are claims and citations identified so grounding and evals can check them?
