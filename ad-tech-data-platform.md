# Ad-tech data platform and AI revenue triage

**Role:** Data Engineer → [TODO: current title] · **2022–present** · European programmatic advertising network (publisher monetisation)
**Stack:** GCP (Cloud Composer/Airflow, BigQuery, Cloud Functions, Cloud Run, Cloud Build, GCS, Secret Manager), Python,
SQL, Terraform, Google ADK + Gemini on Agent Runtime, Looker Studio

> Employer and client names are omitted and the source code is proprietary. This page covers the problems, the architecture and my part in it. A from-scratch public
> version of the core patterns lives in
> [gcp-data-platform-template](https://github.com/thanzap/gcp-data-platform-template) and
> [ad-revenue-anomaly-agent](https://github.com/thanzap/ad-revenue-anomaly-agent).

## Context

An ad network earns its revenue across dozens of demand partners (SSPs, ad exchanges, video platforms) on behalf of
[TODO: N] publishers. Every partner reports differently, on its own schedule, in its own currency and taxonomy.
Publisher payouts, sales decisions and day-to-day ad-ops all depend on one trustworthy revenue picture, available
every morning.

## What I built and run

**1. The ingestion platform: a team-built estate of 80+ Airflow DAG families on Cloud Composer, which I help maintain and extend.**
- Connectors for SSP / ad-server / analytics APIs (including asynchronous "request → poll → download" report APIs),
  GA4 custom events, web-vitals/CrUX, ads.txt and page-quality crawlers.
- Loads into partitioned BigQuery tables with idempotent per-day merges, so backfills and retries never duplicate revenue.
- Ran the migration of the whole DAG estate between Composer generations (Airflow 2.6 → 2.11), including dependency
  pinning for the new image and a Cloud Build deployment pipeline that syncs DAGs to the environment.
- [TODO: volume, e.g. "~X M rows/day across Y partners", "€Z revenue/month reported through the platform"]

**2. Revenue anomaly detection with AI triage.**
- Daily detection of publisher-level revenue and ad-density anomalies against same-weekday baselines, with a
  drill-down report per incident.
- Built an **ADK agent (Gemini) deployed on Agent Runtime** with read-only tools over the warehouse, pipeline status,
  logs and a written spec of the revenue data flow. It triages each anomaly (real demand change vs. data gap vs.
  single-partner outage) and writes the explanation into the morning email.
- Key design decision: evidence gathering and the first classification are **deterministic code**. The LLM only
  explains. An earlier version that let the model loop over sites skipped and misclassified sites, and moving the
  logic into tools made it reliable. The agent is optional: if it fails or times out, the email ships unchanged.
- [TODO: impact, e.g. "time to root cause from ~X h to Y min", "N false alarms/week avoided"]

**3. Forecasting, pricing and recommendations.**
- Revenue forecasting pipelines (Prophet and weekly-seasonality methods) used for [TODO: planning / publisher payouts].
- Floor-price and CPC optimisation analyses; contextual-advertising and article recommendation-system pipelines.

**4. Reliability and governance.**
- Found that the weekly BigQuery backup (Data Transfer Service) had been silently failing for months: tables had been
  re-created with different clustering, which breaks copy-over-existing. Fixed it, ran a restore test, and documented
  the remaining gaps (service-account ownership, failure alerts, dated retention).
- Traced recurring "revenue drops" on dashboards to data-freshness traps (late GA4 exports, a missing day in an
  upstream table, by-design zeros in Looker formulas) and fixed them at the source: retries on the upstream DAG,
  earlier schedules, and documented checks for whoever is on call.

## What I'd highlight in an interview

- Designing for **re-runnability**: every load is a pure function of (day, source).
- Putting **LLMs behind deterministic guardrails** in a revenue-critical workflow, with graceful degradation.
- Operating a large Airflow estate: upgrades, dependency hell, zombie tasks, timeouts as policy.
