# Full-article fetching

Type: wayfinder:grilling
Blocked by: Data sources and terms, Prompt-injection threat model
Status: CLOSED 2026-11-23

## Question
Freeman only sees snippets of about 300 characters. Should the desk read full articles? Decide whether it is in v1, legal and robots limits, how pages are fetched safely (SSRF, size and time limits, content-type checks, HTML-to-text), how longer text raises the injection risk and the cost, and what happens when a page cannot be read.

Input from the data-sources ticket: GDELT returns titles, country, language and date, never article text, so without fetching, Triage sees headlines only and many snapshots will be thin. SEC EDGAR filings do carry text. Publisher article text raises rights, robots and safety questions.

## Resolution
**Not in v1.** Triage reads GDELT headlines and SEC EDGAR filing text; the desk never fetches publisher article pages.

- Headlines are attributed statements, so the quote check works on them, and the thin-snapshot rules (3 usable sources, 2 publishers) can be met with headlines alone.
- The README states the limitation plainly: analyses built from headlines are shallow, and many tickers will return `unclear`. That is the desk working as designed, not a failure.
- `Source.text` is already general (a title for GDELT, a filing excerpt for EDGAR), so a fetcher could be added later as another adapter without changing the engine.
- v1 avoids a whole class of risk: scraping and robots rules, copyright of full text, server-side request forgery, paywalled and script-only pages, and a much larger prompt-injection surface and token cost.

**Possible later ticket (not planned).** If a richer demo matters after the engine and evals exist, fetching becomes its own ticket, scoped as demo-time only and never stored. It would need a safe fetcher (block internal addresses, size, time and content-type limits, robots.txt) and a rights review, and the evals would measure whether full text actually improves results. An allowlist of friendly sources (investor-relations pages, press-release wires) is the likelier shape than the open web.
