---
title: How to Get ClinicalTrials.gov Data as Clean JSON (No Code)
tags: [api, data, healthcare, webscraping]
canonical_target: dev.to or Medium
---

# How to Get ClinicalTrials.gov Data as Clean JSON (No Code)

If you've ever tried to pull data out of [ClinicalTrials.gov](https://clinicaltrials.gov),
you know the registry is a goldmine — over 500,000 studies — but getting it into a
spreadsheet or a database is more annoying than it should be. Here's the quick way to
do it, both the DIY route and a no-code option.

## The official API (and why it's fiddly)

ClinicalTrials.gov has a modern, free, public API (v2):

```
GET https://clinicaltrials.gov/api/v2/studies?query.cond=breast%20cancer&filter.overallStatus=RECRUITING&pageSize=100
```

No key required. It returns JSON — which is great, until you look at the shape. Each
study is deeply nested under `protocolSection`, split across modules like
`identificationModule`, `statusModule`, `sponsorCollaboratorsModule`,
`designModule`, `armsInterventionsModule`, and `contactsLocationsModule`. To get a
flat row like *"NCT id, title, phase, sponsor, enrollment, status, country"* you have
to dig through five or six nested objects per record — and then handle pagination via
`nextPageToken` for anything beyond the first 100 results.

That's fine if you're a developer with an afternoon. If you just want the data, it's
friction.

## The DIY flatten (Python)

If you do want to code it, the core loop is straightforward:

```python
import requests

def fetch(cond, status="RECRUITING", max_results=200):
    out, token = [], None
    while len(out) < max_results:
        params = {"query.cond": cond, "filter.overallStatus": status, "pageSize": 100}
        if token: params["pageToken"] = token
        data = requests.get("https://clinicaltrials.gov/api/v2/studies", params=params).json()
        for s in data.get("studies", []):
            ps = s.get("protocolSection", {})
            idm = ps.get("identificationModule", {})
            out.append({
                "nctId": idm.get("nctId"),
                "title": idm.get("briefTitle"),
                "status": ps.get("statusModule", {}).get("overallStatus"),
                "sponsor": ps.get("sponsorCollaboratorsModule", {}).get("leadSponsor", {}).get("name"),
                # ...repeat for phase, enrollment, conditions, locations...
            })
        token = data.get("nextPageToken")
        if not token: break
    return out
```

You'll still want to add the phase, enrollment, interventions, and location parsing,
plus retries and rate-limit handling.

## The no-code option

If you'd rather skip all that, I packaged the same logic into a small tool that
returns clean, flat records and handles pagination for you:
**[Clinical Trials Monitor](https://apify.com/lxraa/clinical-trials-monitor)**.

You give it a condition (and optionally a sponsor or status filter), and it returns
records like:

```json
{
  "nctId": "NCT06331169",
  "title": "Anlotinib With Trastuzumab Deruxtecan for HER2-Low Advanced Breast Cancer",
  "status": "RECRUITING",
  "phase": "PHASE1",
  "leadSponsor": "Fudan University",
  "enrollment": 42,
  "conditions": ["Breast Cancer"],
  "interventions": ["DRUG: Anlotinib", "DRUG: Trastuzumab deruxtecan"],
  "countries": ["China"],
  "url": "https://clinicaltrials.gov/study/NCT06331169"
}
```

Export to JSON/CSV, or schedule it to monitor newly recruiting trials in your area of
interest. It pulls from the official API, so it's accurate and within terms.

## Who this is for

- **Pharma / biotech** tracking competitor and pipeline activity.
- **CROs and recruiters** finding recruiting trials by condition or region.
- **Researchers and data teams** who want a structured dataset, not a web page.

Either way — code it yourself with the snippet above, or grab the no-code tool. Both
beat copy-pasting from the website.
