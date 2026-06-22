# Public Data Scrapers — clean JSON from official APIs

No-code [Apify](https://apify.com) actors that turn **official public data APIs**
into clean, structured JSON/CSV — plus free guides showing how to do it yourself.

All three pull from **official, public sources** (no personal data, within terms),
handle pagination and messy nested responses for you, and can be **scheduled** to
monitor new records.

## The actors

| Tool | Source | Get |
|---|---|---|
| [Clinical Trials Monitor](https://apify.com/lxraa/clinical-trials-monitor) | ClinicalTrials.gov | Trials by condition / sponsor / status → flat JSON |
| [SEC EDGAR Filing Monitor](https://apify.com/lxraa/edgar-filing-monitor) | SEC EDGAR full-text search | 10-K/10-Q/8-K/S-1 by keyword / form / date → JSON + links |
| [US Federal Spending Monitor](https://apify.com/lxraa/usaspending-monitor) | USAspending.gov | Federal contracts & grants by keyword / agency → JSON |
| [Company Jobs Aggregator](https://apify.com/lxraa/company-jobs-aggregator) | Greenhouse / Lever / Ashby | Open jobs across companies via official ATS APIs → JSON |

## Free how-to guides

If you'd rather build it yourself, each guide explains the official API, the gotchas,
and a working Python snippet:

- [Get ClinicalTrials.gov data as clean JSON](guides/01-clinicaltrials-data-as-json.md)
- [Search SEC EDGAR filings programmatically](guides/02-sec-edgar-filings-to-json.md)
- [Pull US federal contract data for sales intelligence](guides/03-us-federal-contracts-data.md)
- [Track company job openings via ATS APIs](guides/04-company-jobs-from-ats-apis.md)

## Who these are for

- **Healthcare / pharma / CRO** — clinical trial tracking.
- **Finance / research** — SEC filing search and monitoring.
- **B2B sales / GovCon / market intelligence** — federal award data.

Code it yourself with the guides, or use the no-code actors. Both beat copy-pasting
from a website.

---

*Built on official public APIs. Contributions and issues welcome.*
