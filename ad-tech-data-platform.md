# Ad-tech data platform and AI revenue triage

**Domain:** European programmatic advertising network (publisher monetisation)
**Stack:** GCP (Cloud Composer/Airflow, BigQuery, Cloud Functions, Cloud Run, Cloud Build, GCS, Secret Manager), Python,
SQL, Terraform, Google ADK + Gemini on Agent Runtime, Looker Studio

> Employer and client names are omitted and the source code is proprietary. This page covers the problems and the architecture. A from-scratch public
> version of the core patterns lives in
> [gcp-data-platform-template](https://github.com/thanzap/gcp-data-platform-template) and
> [ad-revenue-anomaly-agent](https://github.com/thanzap/ad-revenue-anomaly-agent).

## Context

An ad network earns its revenue across ~20 demand partners (SSPs, ad exchanges, video platforms) on behalf of
~70 publishers. Every partner reports differently, on its own schedule, in its own currency and taxonomy.
Publisher payouts, sales decisions and day-to-day ad-ops all depend on one trustworthy revenue picture, available
every morning.

## What the platform does

**1. The ingestion platform: 80+ Airflow DAG families on Cloud Composer.**
- Connectors for SSP / ad-server / analytics APIs (including asynchronous "request → poll → download" report APIs),
  GA4 custom events, web-vitals/CrUX, ads.txt and page-quality crawlers.
- Loads into partitioned BigQuery tables with idempotent per-day merges, so backfills and retries never duplicate revenue.
- Ran the migration of the whole DAG estate between Composer generations (Airflow 2.6 → 2.11), including dependency
  pinning for the new image and a Cloud Build deployment pipeline that syncs DAGs to the environment.
- Revenue from ~20 SSPs and ad exchanges, for ~70 publishers, lands in one unified table every morning.

**2. Revenue anomaly detection and an AI BI agent.**
- Daily detection of publisher-level revenue and ad-density anomalies against same-weekday baselines, with a
  drill-down report per incident.
- An **ADK agent (Gemini) deployed on Agent Runtime** with read-only tools over BigQuery, Airflow (DAG code, run
  status, logs) and a written spec of the revenue data flow. It works in two modes:
  - **Daily triage:** classifies each anomaly (real demand change vs. data gap vs. single-partner outage) and writes
    the explanation into the morning email.
  - **Ask the data:** answers ad-hoc questions in plain language, such as "why did this site drop yesterday?" or
    "which DAG loads this partner, and did it run?", by querying the warehouse and scanning DAGs, logs and lineage.
- Key design decision: evidence gathering and the first classification are **deterministic code**. The LLM only
  explains. An earlier version that let the model loop over sites skipped and misclassified sites, and moving the
  logic into tools made it reliable. The agent is optional: if it fails or times out, the email ships unchanged.
- **Impact:** finding why revenue moved used to mean opening each publisher's chart and digging through the
  differences by hand. Now the morning email lists only the significant moves, each with its likely cause.

**3. Pricing and recommendations.**
- Floor-price and CPC optimisation analyses; contextual-advertising and article recommendation-system pipelines.

**4. Reliability and governance.**
- Found that the weekly BigQuery backup (Data Transfer Service) had been silently failing for months: tables had been
  re-created with different clustering, which breaks copy-over-existing. Fixed it, ran a restore test, and documented
  the remaining gaps (service-account ownership, failure alerts, dated retention).
- Traced recurring "revenue drops" on dashboards to data-freshness traps (late GA4 exports, a missing day in an
  upstream table, by-design zeros in Looker formulas) and fixed them at the source: retries on the upstream DAG,
  earlier schedules, and documented checks for whoever is on call.

## Key engineering themes

- Designing for **re-runnability**: every load is a pure function of (day, source).
- Putting **LLMs behind deterministic guardrails** in a revenue-critical workflow, with graceful degradation.
- Operating a large Airflow estate: upgrades, dependency hell, zombie tasks, timeouts as policy.
