# Prompt-injection threat model

Type: wayfinder:grilling
Blocked by: none

## Question
News articles, snippets and full-article text are untrusted input an attacker can plant. What can go wrong (steering the stance, hiding risks, leaking the system prompt, the position or keys, forged citations), which guardrails apply (data isolation in one JSON string as Freeman does, schema-constrained output, citation checking, no tools for agents that read articles, least privilege), and how are these attacks included in the evals?
