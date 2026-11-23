# Stock News Research Desk

A multi-agent system that turns a stock ticker and a snapshot of recent news into a cited, abstention-aware research note. It describes what published sources say. It never recommends trades or predicts prices.

## Language

### Inputs

**Snapshot**:
The set of sources fetched for one ticker at one point in time, each with a stable id.
_Avoid_: Feed, batch, dataset

**Source**:
One news article, headline or filing item inside a snapshot.
_Avoid_: Article, item, document

**Quote**:
An exact span of text copied from one source. It is valid only if it appears literally in that source.
_Avoid_: Excerpt, snippet

### Reasoning

**Fact**:
An attributed statement extracted from a source, with a quote and the id of that source. A fact states what a source says, never what is true.
_Avoid_: Finding, evidence, data point

**Claim**:
A point made by an analysis agent or the writer. Every claim must cite at least one fact.
_Avoid_: Point, argument, finding

**Citation**:
The link from a claim to the facts that support it, shown to the reader as the underlying source.
_Avoid_: Reference, link, footnote

### Output

**News lean**:
The headline verdict of an analysis: `positive`, `negative`, `mixed` or `unclear`, describing what the cited sources taken together suggest about the company's near-term outlook.
_Avoid_: Stance, recommendation, signal, rating, buy/hold/sell

**Confidence**:
`low`, `medium` or `high`, computed by code from the evidence behind the news lean. It is never reported by a model.
_Avoid_: Certainty, probability, score
