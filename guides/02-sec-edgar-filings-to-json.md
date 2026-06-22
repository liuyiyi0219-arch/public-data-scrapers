---
title: How to Search SEC EDGAR Filings Programmatically (and Export to JSON)
tags: [api, finance, data, webscraping]
canonical_target: dev.to or Medium
---

# How to Search SEC EDGAR Filings Programmatically (and Export to JSON)

Every public US company files with the SEC, and it's all on
[EDGAR](https://www.sec.gov/edgar) — 10-Ks, 10-Qs, 8-Ks, S-1s, and more. The data is
free and public. The trick is searching it *full-text* and getting structured results
you can actually work with.

## The full-text search API

EDGAR has a full-text search endpoint that most people never find:

```
GET https://efts.sec.gov/LATEST/search-index?q="artificial intelligence"&forms=10-K
```

It searches the text of filings (not just company names) and returns JSON hits. Two
things to know:

1. **SEC requires a descriptive `User-Agent`** with a contact email, e.g.
   `My App (you@example.com)`. Requests without it get blocked.
2. Each hit is compact — you get `display_names` (company + ticker + CIK), `form`,
   `file_date`, and `adsh` (the accession number). To turn that into a clickable
   filing you build the archive URL yourself.

## Building the filing link

The accession number plus CIK gets you the filing index page:

```python
import requests

UA = {"User-Agent": "My App (you@example.com)"}

def search(q, forms="10-K", limit=100):
    url = "https://efts.sec.gov/LATEST/search-index"
    r = requests.get(url, params={"q": q, "forms": forms, "from": 0}, headers=UA).json()
    out = []
    for h in r.get("hits", {}).get("hits", []):
        s = h["_source"]
        adsh = s["adsh"]; cik = str(int(s["ciks"][0]))
        out.append({
            "companies": s.get("display_names"),
            "form": s.get("form"),
            "fileDate": s.get("file_date"),
            "accessionNumber": adsh,
            "indexUrl": f"https://www.sec.gov/Archives/edgar/data/{cik}/{adsh.replace('-','')}/{adsh}-index.htm",
        })
    return out
```

Add pagination via the `from` offset (100 per page, up to 10,000), plus retries, and
you've got a working EDGAR search.

## The no-code version

If you'd rather not maintain that, I wrapped it into a tool that handles the
User-Agent, pagination, and link-building and returns clean records:
**[SEC EDGAR Filing Monitor](https://apify.com/lxraa/edgar-filing-monitor)**.

Example output:

```json
{
  "companies": ["RadNet, Inc.  (RDNT)  (CIK 0000790526)"],
  "form": "8-K",
  "fileDate": "2020-03-16",
  "accessionNumber": "0001683168-20-000837",
  "indexUrl": "https://www.sec.gov/Archives/edgar/data/790526/000168316820000837/0001683168-20-000837-index.htm",
  "states": ["CA"]
}
```

Filter by keyword, form type, and date range; export to JSON/CSV; or schedule it to
watch for new filings mentioning a company, product, or topic.

## Use cases

- **Finance & research** — track filings by theme (e.g. every 8-K mentioning "data
  breach" or "going concern").
- **Competitive intelligence** — monitor a competitor's filings as they post.
- **Datasets** — build a structured corpus of filings for analysis or LLMs.

Public data, official source — just easier to use.
