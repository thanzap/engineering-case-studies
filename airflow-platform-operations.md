# Running a large Airflow estate on Cloud Composer

**Domain:** data platform operations · **Stack:** Cloud Composer 2 (Airflow 2.6 → 2.11), Cloud Build, Terraform, GCS,
Secret Manager, Python

## Context

The team's data platform runs 80+ DAG families: partner ETL, analytics, recommendations, monitoring.
They share one Composer environment. With that many pipelines, platform health is its own job: a single stuck task
can starve the scheduler, and an upgrade can break dozens of DAGs at once.

## What was built

**Environment migration (Airflow 2.6 → 2.11, new Composer generation).**
- **Zero downtime:** stood up the new environment alongside the old one, deployed a duplicate of every DAG there
  paused, then switched them over one by one (enable in the new environment, pause in the old), so every pipeline
  stayed live throughout.
- Rebuilt the dependency set for the new image. Some provider packages that the Airflow constraints file lists are
  not pre-installed in the image and had to be pinned explicitly.
- Split deployment: a Cloud Build pipeline rsyncs `dags/` to the environment bucket, while a private secrets-client
  module (a git submodule) ships through its own build, so it never travels with the main repo.

**Self-healing and visibility.**
- **Queue unclogging DAG:** finds task instances stuck in `running` or `queued` far beyond their expected duration,
  marks them failed so the scheduler frees the slots, and emails the list of what it touched.
- **Failed-task digest:** a twice-daily report of every failed task across all DAGs, with log links, instead of
  hundreds of individual failure emails.
- **Metadata DB maintenance:** scheduled cleanup of old DAG runs, task instances, XComs and logs to keep the
  scheduler fast.
- **Environment backup:** a scheduled export of the environment's configuration and DAG bucket, later moved under
  Terraform.

**Reliability policies.**
- Execution timeouts and retries made a review requirement after a scraper with no timeout created zombie jobs.
- `depends_on_past` removed where one red day would block all following days.
- Schedules moved to UTC, so daylight-saving changes don't reorder dependent DAGs.

## Public version

The conventions (timeouts, retries, owners, fast-parsing DAGs, DAG-integrity tests in CI, deployment via Workload
Identity Federation) are in [gcp-data-platform-template](https://github.com/thanzap/gcp-data-platform-template).
