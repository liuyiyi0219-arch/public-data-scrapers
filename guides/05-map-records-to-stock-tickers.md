---
title: How to Map SEC Filings and Federal Contracts to Stock Tickers (Free, Exact Method)
tags: [api, finance, alternative-data, webscraping, quant]
canonical_target: dev.to or Medium
---

# How to Map SEC Filings and Federal Contracts to Stock Tickers (Free, Exact Method)

Raw public data — SEC filings, federal contracts, clinical trials, FDA recalls — is
free. The part people actually pay for is the **processing layer**: turning a company
*name* in a record into an **investable entity** — a stock ticker and a stable
identifier you can join on. A 10-K is interesting; a 10-K you can instantly attach to
`NVDA` is a signal.

This guide shows the cleanest free way to do that mapping, the one trick that makes it
*exact* instead of fuzzy, and where the method breaks down (so you don't trust a bad
match).

## The one official file you need

The SEC publishes a free, authoritative list of every public company with a ticker:

```
https://www.sec.gov/files/company_tickers.json
```

It maps **CIK ↔ ticker ↔ company name**:

```json
{"0":{"cik_str":320193,"ticker":"AAPL","title":"Apple Inc."}, ...}
```

The **CIK** (Central Index Key) is the SEC's permanent ID for a filer. This is the
hinge of the whole technique.

## The exact method: join on CIK, never on name

If your data source already carries a CIK — and **every SEC EDGAR record does** —
don't match on the company name at all. Join on the CIK and you get a near-perfect,
unambiguous ticker:

```python
import requests

UA = {"User-Agent": "My App (you@example.com)"}  # SEC requires a contact UA

# 1) Build a CIK -> ticker index once
idx = {}
for row in requests.get("https://www.sec.gov/files/company_tickers.json",
                        headers=UA).json().values():
    if row.get("ticker"):
        idx[int(row["cik_str"])] = row["ticker"]

# 2) Each EDGAR full-text hit has a CIK already — just look it up
def ticker_for_cik(cik):
    return idx.get(int(cik))   # None if the filer isn't publicly traded
```

EDGAR's full-text search (`https://efts.sec.gov/LATEST/search-index?q=...`) returns a
`ciks` array on every hit, so this join needs **no fuzzy matching and almost never
mismatches.** In testing across a real 10-K query, this maps **~96% of hits** to a
ticker (the misses are OTC/delisted names not on the official list).

## The fuzzy method: when you only have a name

Federal contracts (USAspending.gov), clinical trials (ClinicalTrials.gov), and FDA
recalls (openFDA) give you a recipient/sponsor **name**, not a CIK. Now you have to
normalize and match — carefully, because **a wrong ticker is worse than no ticker.**

```python
import re

SUFFIXES = re.compile(r"\b(THE|CORPORATION|CORP|INCORPORATED|INC|COMPANY|CO|LLC|LP|"
                      r"LLP|LTD|LIMITED|HOLDINGS|GROUP|PLC|SA|SE|NV|AG|AND|"
                      r"PHARMACEUTICALS?|PHARMA|THERAPEUTICS|LABORATORIES|LABS|"
                      r"SCIENCES)\b")

def norm(name: str) -> str:
    s = (name or "").upper()
    s = re.sub(r"/[A-Z]{2}/", " ", s)          # strip SEC state markers like /DE/
    s = re.sub(r"[.,&'\"()/\-]", " ", s)
    s = SUFFIXES.sub(" ", s)
    return re.sub(r"\s+", " ", s).strip()

# Build a normalized-title -> ticker index from the same SEC file, then:
def match(name, by_title):
    n = norm(name)
    if len(n) < 4:
        return None
    if n in by_title:                          # 1) exact normalized match
        return by_title[n]
    for title, tk in by_title.items():         # 2) safe prefix (subsidiaries)
        if len(title) >= 8 and " " in title and n.startswith(title + " "):
            return tk                          #    "LOCKHEED MARTIN AERONAUTICS" -> LMT
    return None
```

Two refinements that matter in practice:

- **Strip `/DE/`-style state-of-incorporation markers** the SEC appends to titles, or
  prefixes like "Northrop Grumman Systems" silently fail to match "Northrop Grumman".
- **Keep a tiny alias table** for renames and foreign ADRs the file won't resolve:
  `RAYTHEON → RTX`, `FACEBOOK → META`, `GOOGLE/ALPHABET → GOOGL`, `JANSSEN → JNJ`,
  `GENENTECH → RHHBY`.

## Hit rate depends entirely on "public-company density"

This is the part nobody tells you. The same technique is excellent on one dataset and
nearly useless on another — because of **who's in the data**, not the code. Measured on
real queries:

| Source | Join | Realistic hit rate | Why |
|---|---|---|---|
| **SEC EDGAR** | CIK (exact) | **~96%** | Filers *are* SEC registrants |
| **US federal contracts** | name | **~87%** | Defense/IT primes are nearly all US-listed |
| Clinical trials (industry) | name | ~10% | Sponsors skew foreign / private / micro-biotech |
| FDA recalls | name | ~5% | Recallers are mostly private or foreign generics |

**Takeaway:** EDGAR and federal contracts are where entity→ticker mapping creates real,
tradable value. On clinical trials and recalls, treat a *match* as a high-signal event
(a J&J recall, a Pfizer trial) and accept that most rows legitimately have no ticker.

## Why this is worth doing

- **Watchlists & screeners** — filter any public dataset down to traded companies.
- **Event/catalyst signals** — a new federal contract or 8-K, already attached to a
  ticker, is a research-ready alert.
- **Portfolio surveillance** — monitor your holdings' government revenue, filings, or
  recall exposure.

All of it uses **official, public data** — public-record, not material non-public
information.

## Skip the plumbing

If you'd rather not maintain the index, aliases, and edge cases, two of these actors do
the entity→ticker mapping for you and output ticker + CIK on every record:

- **[SEC EDGAR Filing Monitor](https://apify.com/lxraa/edgar-filing-monitor)** — exact
  CIK join, ~96% mapped.
- **[US Federal Spending Monitor](https://apify.com/lxraa/usaspending-monitor)** —
  recipient → ticker, ~87% mapped on defense/IT awards.

Both run on a schedule, so you get a fresh, ticker-mapped feed instead of a one-off pull.
