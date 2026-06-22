---
title: How to Track Company Job Openings via ATS APIs (Greenhouse, Lever, Ashby)
tags: [api, jobs, recruiting, data]
canonical_target: dev.to or Medium
---

# How to Track Company Job Openings via ATS APIs (Greenhouse, Lever, Ashby)

Want to monitor who's hiring at a set of companies — for recruiting, sales timing,
or competitive intelligence? You don't need to scrape job boards. Most companies post
jobs through an applicant tracking system (ATS), and the big ones expose **public JSON
APIs** for their job boards. No keys, no blocking.

## The endpoints

```
Greenhouse:  https://boards-api.greenhouse.io/v1/boards/{company}/jobs
Lever:       https://api.lever.co/v0/postings/{company}?mode=json
Ashby:       https://api.ashbyhq.com/posting-api/job-board/{company}
```

The `{company}` is the board slug — usually the company name as it appears in their
careers URL (e.g. `greenhouse:stripe`, `ashby:ramp`). Each returns the company's
current open roles as JSON.

## DIY (Python)

```python
import requests

def greenhouse(slug):
    d = requests.get(f"https://boards-api.greenhouse.io/v1/boards/{slug}/jobs").json()
    return [{"title": j["title"], "location": j.get("location", {}).get("name"),
             "url": j["absolute_url"]} for j in d.get("jobs", [])]

print(greenhouse("stripe")[:3])
```

Lever and Ashby return slightly different shapes (title under `text`, location under
`categories.location` for Lever; `title`/`location` for Ashby), so you normalize each
into a common record.

## The no-code version

I built an actor that takes a list like `["greenhouse:stripe", "ashby:ramp",
"lever:plaid"]` and returns every open role across all of them, normalized:
**[Company Jobs Aggregator](https://apify.com/lxraa/company-jobs-aggregator)**.

```json
{
  "company": "stripe",
  "ats": "greenhouse",
  "title": "Account Executive, AI Sales",
  "location": "San Francisco, CA",
  "department": "Sales",
  "url": "https://job-boards.greenhouse.io/stripe/jobs/...",
  "postedAt": "2026-06-10T00:00:00Z"
}
```

Schedule it weekly to watch a list of target companies and see new roles as they open.

## Use cases

- **Recruiters / sourcers** — track openings across a portfolio of companies.
- **Sales** — a new role (e.g. "Head of Data") is a buying signal; time your outreach.
- **Market intelligence** — see where competitors are investing by what they hire.

Official ATS APIs, public data — no scraping required.
