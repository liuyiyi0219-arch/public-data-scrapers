# Sample Datasets

Free sample snapshots from the [public-data scrapers](../README.md). Each links to the actor that produces **fresh, custom** data on demand.

## ⭐ Flagship: public records mapped to stock tickers

The differentiator — raw public data **already joined to an investable stock ticker**.
See [the method & full hit-rate-by-source table](../guides/05-map-records-to-stock-tickers.md).

### US defense & aerospace contracts → stock tickers
- **200 records** · [`federal-contracts-to-tickers.json`](federal-contracts-to-tickers.json) · [`federal-contracts-to-tickers.csv`](federal-contracts-to-tickers.csv)
- Largest 2025–2026 US defense/aerospace federal contracts (USAspending.gov), each recipient mapped to its public-company ticker. **71% mapped** (rest are national labs / JVs / private firms).
- Columns: awardId, recipient, **ticker**, amount, awardingAgency, awardingSubAgency, startDate, description, url
- **Live / custom data:** [usaspending-monitor actor](https://apify.com/lxraa/usaspending-monitor)

### SEC 10-K filers mentioning AI → stock tickers
- **100 records** · [`sec-edgar-10k-to-tickers.json`](sec-edgar-10k-to-tickers.json) · [`sec-edgar-10k-to-tickers.csv`](sec-edgar-10k-to-tickers.csv)
- Companies discussing "artificial intelligence" in their 10-K, each mapped to its ticker via an **exact CIK join**. **85% mapped** (rest trade OTC / not in SEC's ticker list).
- Columns: company, cik, **ticker**, form, fileDate, accessionNumber
- **Live / custom data:** [edgar-filing-monitor actor](https://apify.com/lxraa/edgar-filing-monitor)

## Other samples

### Clinical Trials (recruiting, cancer)
- **50 records** · [`clinical-trials.json`](clinical-trials.json) · [`clinical-trials.csv`](clinical-trials.csv)
- Recruiting oncology trials from ClinicalTrials.gov.
- Columns: nctId, url, title, status, phase, studyType, enrollment, leadSponsor…
- **Live / custom data:** [clinical-trials-monitor actor](https://apify.com/lxraa/clinical-trials-monitor)

### SEC EDGAR 10-K filings mentioning AI
- **50 records** · [`sec-edgar-filings.json`](sec-edgar-filings.json) · [`sec-edgar-filings.csv`](sec-edgar-filings.csv)
- 10-K filings matching "artificial intelligence" via SEC EDGAR full-text search.
- Columns: companies, ciks, form, rootForms, fileDate, periodEnding, accessionNumber, fileNumbers…
- **Live / custom data:** [edgar-filing-monitor actor](https://apify.com/lxraa/edgar-filing-monitor)

### US federal AI contracts
- **50 records** · [`us-federal-contracts.json`](us-federal-contracts.json) · [`us-federal-contracts.csv`](us-federal-contracts.csv)
- Federal contracts matching "artificial intelligence" from USAspending.gov.
- Columns: awardId, recipient, amount, description, awardingAgency, awardingSubAgency, startDate, endDate…
- **Live / custom data:** [usaspending-monitor actor](https://apify.com/lxraa/usaspending-monitor)

### Open tech jobs (Stripe, Ramp, GitLab)
- **60 records** · [`company-jobs.json`](company-jobs.json) · [`company-jobs.csv`](company-jobs.csv)
- Open roles pulled from company ATS boards (Greenhouse/Ashby).
- Columns: company, ats, title, location, department, url, postedAt, scrapedAt
- **Live / custom data:** [company-jobs-aggregator actor](https://apify.com/lxraa/company-jobs-aggregator)

### FDA ongoing drug recalls
- **50 records** · [`fda-drug-recalls.json`](fda-drug-recalls.json) · [`fda-drug-recalls.csv`](fda-drug-recalls.csv)
- Ongoing FDA drug recalls from the openFDA API.
- Columns: recallNumber, status, classification, recallingFirm, reason, productDescription, productQuantity, distributionPattern…
- **Live / custom data:** [fda-recalls-monitor actor](https://apify.com/lxraa/fda-recalls-monitor)

*Snapshots for preview. Re-run the linked actor for current, filtered results.*
