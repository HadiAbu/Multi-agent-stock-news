# Data sources and terms

Type: wayfinder:research (AFK, subagent writes a cited markdown file)
Blocked by: none
Status: CLOSED 2026-11-23

## Question
Which news sources can a public demo use, and under what terms? Check Finnhub (free tier limits, redistribution and caching terms, whether stored samples may be published) and GDELT (rate limits, reuse terms), plus alternatives. Can frozen snapshots be committed to the repo, or must evals use synthetic articles? Recommend a data plan for live demo, CI and the eval set.

## Resolution
Research is in `12-research.md` (checked 2026-11-23; not legal advice; re-read each source's terms before publishing). The author accepted this data plan:

| Use | Source |
|---|---|
| Live demo | GDELT (cited, paced, cached server-side) and SEC EDGAR recent filings |
| CI | Committed fixtures only, with a mocked model and news client; no live calls |
| Eval set | About 20-30 hand-written synthetic articles, a few EDGAR excerpts, and optionally GDELT metadata (title, URL, date, domain) with the required citation |

- **No Finnhub data in the public repo or demo.** Its terms bar redistributing data or derived results without written approval, and its plans are personal-use only. A Finnhub adapter could exist as code, but no Finnhub output is committed or shown to others.
- Record the source and retrieval date for every real-data item. Synthetic items are tagged with the failure mode they test (conflicting claims, thin news, injection text, stale items).
- GDELT's reuse terms require a citation to the GDELT Project with a link, shown wherever its data appears.
- SEC EDGAR fair access: at most 10 requests per second and a User-Agent that declares a contact email. That email comes from an environment variable and is never committed.

**Consequences for other tickets.**
- GDELT returns titles, country, language and date, not article text. Triage working on GDELT sources sees headlines only, so quotes are headline fragments and many snapshots will be thin. EDGAR filings carry real text. This raises the importance of the full-article fetching ticket and of leaning on EDGAR for depth.
- The eval ticket builds on synthetic articles and EDGAR excerpts, with no real publisher snippets.

**Still unverified, not blocking:** Finnhub's free endpoint list and history range, whether a public demo counts as Finnhub "personal use", GDELT's current rate guidance, the Alpha Vantage and Yahoo terms, and non-US copyright treatment of headlines.
