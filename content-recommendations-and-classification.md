# Content recommendations and contextual classification for publishers

**Domain:** publisher engagement and contextual advertising · **Stack:** Airflow, GA4 Data API, Google Cloud Natural
Language API, Gemini, BigQuery, Firestore / CDN JSON, Cloud Functions, web scraping

## Problem

The network offered publishers an "articles you may like" widget and contextual (cookieless) ad targeting. Both need
to know, for every article on every site, **what it is about** and **what's popular**, refreshed daily and served
fast to the browser.

## Pipeline

```mermaid
flowchart LR
    GA[GA4: page paths + views<br/>per site] --> F["filter to real articles<br/>(path length, min events,<br/>exclusions such as /app/)"]
    F --> C["contextual classification<br/>scrape → translate (Gemini)<br/>→ NL API categories → IAB taxonomy<br/>+ sentiment"]
    C --> BQ[(page categories<br/>append-only, new URLs only)]
    F --> R1["recommender 1:<br/>popular on the same site"]
    BQ --> R2["recommender 2:<br/>same IAB category"]
    R1 & R2 --> U["unify + de-duplicate<br/>(no self-recs, no consecutive<br/>same-domain items)"]
    U --> S[(Firestore / CDN JSON<br/>served to the widget)]
    U --> MON[monitoring: coverage<br/>per site]
```

- **Article detection without a CMS.** Only GA4 page paths are available, so heuristics separate articles from
  section pages and apps: minimum path length, minimum event count, and an exclusion list.
- **Classification.** Article text is scraped and translated to English (moved from a free translation endpoint that
  rate-limited to Gemini Flash-Lite), then classified with the Natural Language API. Categories are mapped to the IAB
  content taxonomy that ad buyers target, and sentiment is attached so negative news can be excluded from brand-safe
  campaigns.
- **Incremental.** Only URLs not classified before are processed (append-only), which keeps API cost proportional to
  new content, not to traffic.
- **Recommendations.** Popular-on-site and same-category candidates are merged, de-duplicated and shuffled, so one
  domain never dominates consecutive slots. The pool was enlarged (to 20+ candidates) so enough valid items survive
  filtering.

## How it evolved

- The recommendation flow was rebuilt on GA4-based input with the article filters, and a deprecated recommender was retired.
- Classification became incremental and was hardened against translation-API rate limits, then the translation step moved to Gemini.
- Both pipelines moved to the new Composer environment with Terraform-defined resources.

**Scale:** the widget runs on **~20 publisher sites**, and **~600 new articles** are classified a day.
