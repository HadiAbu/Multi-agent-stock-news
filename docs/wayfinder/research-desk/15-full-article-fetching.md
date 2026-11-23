# Full-article fetching

Type: wayfinder:grilling
Blocked by: Data sources and terms, Prompt-injection threat model

Input from the data-sources ticket: GDELT returns titles, country, language and date, never article text, so without fetching, Triage sees headlines only and many snapshots will be thin. SEC EDGAR filings do carry text. Publisher article text raises rights, robots and safety questions.

## Question
Freeman only sees snippets of about 300 characters. Should the desk read full articles? Decide whether it is in v1, legal and robots limits, how pages are fetched safely (SSRF, size and time limits, content-type checks, HTML-to-text), how longer text raises the injection risk and the cost, and what happens when a page cannot be read.
