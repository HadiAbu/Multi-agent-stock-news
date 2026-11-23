# Eval metrics, snapshots and CI gating

Type: wayfinder:grilling
Blocked by: Analysis schema and engine interface, Data sources and terms

## Question
There is no ground truth for a "right" stance, so what is measured? Decide the frozen news-snapshot set (sources, size, how it is stored without licence or privacy problems), the metrics (citation validity, unsupported-claim rate, abstention on thin news, injection pass rate, cost and latency per run), how an LLM judge is used and checked, which model runs in CI, pass/fail thresholds, and how results appear in the README.
