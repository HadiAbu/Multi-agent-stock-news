# Ticket 12: News data sources, terms, and a data plan

Checked 2026-11-23. This is research, not legal advice. Re-read each source's terms before publishing.

## Recommendation

- **Do not commit Finnhub data to the public repo.** Its terms bar redistributing data or "derived results" without written approval.
- **Eval set:** use self-written synthetic articles, plus real SEC EDGAR filing excerpts (public information). Optionally add GDELT metadata (title, URL, date, domain) with the required citation. Avoid committing publisher snippets.
- **CI:** run only on committed fixtures with a mocked model/news layer. No live news calls.
- **Live demo:** fetch GDELT (and EDGAR) on request. Finnhub is a personal-use key, so use it only if the demo stays personal and is not a redistribution channel. Treat that as unresolved (see below).

## 1. Finnhub

- **Free tier:** the pricing page lists 60 API calls/minute for the free plan (via search snippet; my direct fetch of the page returned no limits). A "30 calls/second" figure appears only as a third-party client default, so I could not confirm it. Source: https://finnhub.io/pricing
- **Endpoints:** I could not retrieve the endpoint docs (company-news, market-news, profile) in this session. Which endpoints are free, and the history range, are **unverified**. Check https://finnhub.io/docs/api before relying on them.
- **Terms** (https://finnhub.io/terms-of-service):
  - "You hereby agree to not redistribute or share access to data or derived results" (continues, requiring written approval).
  - "All data must be deleted should your subscription to that data ends."
  - "All plan listed on Finnhub website is strictly for personal use unless explicitly stated otherwise." Personal plans exclude use under a business name.
  - Caching is not explicitly addressed. Publishing is not separately addressed, but the redistribution clause plausibly covers a public repo. Committing headlines or summaries looks incompatible with it; I am not certain how Finnhub would read it.
  - Whether a public portfolio demo counts as "personal use" is **unclear**. Ask Finnhub in writing if you want to use it.

## 2. GDELT

- **Terms:** use is "unlimited and unrestricted ... for any academic, commercial, or governmental use of any kind without fee." "You may redistribute, rehost, republish, and mirror any of the GDELT datasets in any form." Any use or redistribution "must include a citation to the GDELT Project" and a link to https://www.gdeltproject.org/. Source: https://www.gdeltproject.org/about.html
- **Rate limit:** GDELT says only that its APIs "are rate limited to protect the underlying ElasticSearch clusters" and gives no number (https://blog.gdeltproject.org/?p=15416). The "1 request / 5 s" figure comes from third-party tools, not from GDELT. Treat it as a pacing target.
- **What it returns:** DOC 2.0 article lists show title, source country, language and publication date, plus a social image where available. The default is 75 records, up to 250 with MAXRECORDS (https://blog.gdeltproject.org/gdelt-doc-2-0-api-debuts/). The URL field and JSON field names were not listed on that page, but the page does not describe full text, and I saw nothing suggesting it is returned.
- **Caveat:** GDELT's licence covers GDELT's dataset. The titles come from third-party publishers, so GDELT cannot grant rights in them. I found no statement either way; that is a gap.
- **Committing samples:** allowed by GDELT's terms with citation. The remaining risk is the publisher titles (see section 4).

## 3. Alternatives

- **SEC EDGAR filings** (8-K, 10-K/10-Q text, press-release exhibits):
  - sec.gov states "Information presented on www.sec.gov is considered public information and may be copied or further distributed." Do not use SEC logos or artwork. Source: https://www.sec.gov/privacy.htm#dissemination
  - Fair-access rules: max 10 requests/second and a declared User-Agent with a contact email. The SEC says it does not allow botnets or automated crawling tools outside its policy. Source: https://www.sec.gov/search-filings/edgar-search-assistance/accessing-edgar-data
  - Caveat: a filing can quote third-party material. The SEC page does not use the words "public domain", so I would not claim that.
  - Fit: strong for fixtures, weaker as "news" (less timely, more formal).
- **Yahoo Finance RSS:** Yahoo's terms reportedly bar adding advertising to RSS feeds and let Yahoo require users to stop. An older (Irish-region) version limited use to unmodified display with attribution. I could not read the current US terms, so this is **partly verified**. Not suitable for committed data. Source (search result): https://legal.yahoo.com/us/en/yahoo/terms/otos/index.html
- **Alpha Vantage news:** I could not read its terms (the fetch returned an unreadable PDF), so its storing/republishing rules are **unverified**. Do not assume it is better than Finnhub. https://www.alphavantage.co/terms_of_service/
- **Company IR press-release pages / publisher RSS:** terms vary per site and were not checked. Each would need its own review.

## 4. Can frozen snapshots be committed?

- **Facts vs. expression:** US copyright does not protect facts (Feist v. Rural, 1991). Commentators generally argue short headlines are weakly protected at best. Courts abroad have split (Australia: not protected; a UK court: protectable). Source (secondary summary, not a primary ruling): https://www.techdirt.com/2004/03/24/can-you-copyright-news-headlines/
- The US Copyright Office's publisher-protections study records that stakeholders still disagree on headline and snippet aggregation: https://copyright.gov/policy/publishersprotections/202206-Publishers-Protections-Study.pdf
- **My reading:** a bare headline plus URL is low-risk in the US. Multi-sentence snippets and summaries are higher risk. Full text is not acceptable. I am **not sure** about the position outside the US, and a contract (API terms) can restrict you even where copyright does not. Finnhub's terms are such a contract.
- **Consequence for evals:** frozen snapshots of *Finnhub* output should not be committed. Snapshots of *GDELT metadata* or *EDGAR text* are defensible. Synthetic articles carry no third-party risk and can be written to test specific behaviours (conflicting claims, thin news, injection text, stale items).

## 5. Data plan

| Use | Source | Notes |
|---|---|---|
| Live demo | GDELT ArtList (cited, cached server-side, paced about 1 req/5 s with backoff) plus EDGAR recent filings | Show GDELT citation and links to originals. Finnhub only if you confirm terms, and show data live, never stored for others. |
| CI | Committed fixtures only; mocked LLM and news client | Deterministic, no keys or network. |
| Eval set | About 20-30 synthetic articles written by you, plus a few EDGAR excerpts, plus optional GDELT title/URL/date/domain records | Record source and retrieval date per item. Keep real-data items to metadata or short filing excerpts. Synthetic items can be tagged with the failure mode they test. |

Open items: confirm Finnhub endpoint availability, and whether a public demo is "personal use"; confirm GDELT's current rate guidance; decide whether to include real GDELT titles at all.
