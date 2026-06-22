---
title: How to Pull US Federal Contract Data for Sales & Market Intelligence
tags: [api, data, sales, govtech]
canonical_target: dev.to or Medium
---

# How to Pull US Federal Contract Data for Sales & Market Intelligence

The US government publishes every contract and grant it awards on
[USAspending.gov](https://www.usaspending.gov). For B2B sales, GovCon, and market
research, this is one of the cleanest "who has budget" signals available — and it's
100% public, no personal data. Here's how to get it as usable data.

## The official API

USAspending has a free API. Award search is a `POST` with a filter body:

```
POST https://api.usaspending.gov/api/v2/search/spending_by_award/
```

```json
{
  "filters": {
    "award_type_codes": ["A", "B", "C", "D"],
    "time_period": [{"start_date": "2025-01-01", "end_date": "2026-06-22"}],
    "keywords": ["artificial intelligence"]
  },
  "fields": ["Award ID", "Recipient Name", "Award Amount", "Awarding Agency", "Start Date"],
  "page": 1, "limit": 100, "sort": "Award Amount", "order": "desc"
}
```

`award_type_codes` `A–D` are contracts; `02–05` are grants. You paginate with `page`
until `page_metadata.hasNext` is false. Each result includes a
`generated_internal_id` you can turn into a link:
`https://www.usaspending.gov/award/{generated_internal_id}`.

## DIY (Python)

```python
import requests

def awards(keywords, start, end, max_results=200):
    url = "https://api.usaspending.gov/api/v2/search/spending_by_award/"
    out, page = [], 1
    while len(out) < max_results:
        body = {
            "filters": {
                "award_type_codes": ["A","B","C","D"],
                "time_period": [{"start_date": start, "end_date": end}],
                "keywords": keywords,
            },
            "fields": ["Award ID","Recipient Name","Award Amount","Awarding Agency","Start Date"],
            "page": page, "limit": 100, "sort": "Award Amount", "order": "desc",
        }
        d = requests.post(url, json=body).json()
        out += d.get("results", [])
        if not d.get("page_metadata", {}).get("hasNext"): break
        page += 1
    return out
```

## The no-code version

I packaged this into a tool that handles pagination, the contracts/grants toggle, and
link-building, returning flat records:
**[US Federal Spending Monitor](https://apify.com/lxraa/usaspending-monitor)**.

```json
{
  "recipient": "ECS FEDERAL, LLC",
  "amount": 120575059.35,
  "awardingAgency": "Department of Defense",
  "startDate": "2020-04-23",
  "url": "https://www.usaspending.gov/award/CONT_AWD_W911QX20C0023_9700_-NONE-_-NONE-"
}
```

Search by keyword, agency, award type, and date; export to JSON/CSV; or schedule it to
track new awards in your market.

## Use cases

- **B2B / GovCon sales** — find prime contractors and budget holders by topic.
- **Market intelligence** — see spend trends by agency or competitor.
- **Research & journalism** — structured federal-spending datasets.

Public money, public data — just made usable.
