# Engineering case studies

Write-ups of production data systems I've worked on. Employer, client and vendor names are omitted and the code
itself is proprietary. Each one covers the problem, the architecture, the key decisions, and links to public,
from-scratch versions of the core patterns.

### Overviews

| Case study | Domain | Highlights |
|---|---|---|
| [Ad-tech data platform & AI revenue triage](ad-tech-data-platform.md) | Programmatic advertising | Composer/Airflow estate, BigQuery, an ADK agent on Agent Runtime that triages revenue anomalies and answers questions over the warehouse and DAGs |
| [BI, billing automation & AI support QA](saas-bi-and-automation.md) | B2B SaaS + hardware | Company warehouse, billing/CRM/shipping automations, a ticket-QA agent |

### Data engineering deep-dives

| Case study | What it shows |
|---|---|
| [Unifying revenue from ~20 demand partners](multi-source-revenue-etl.md) | Raw-first multi-source ETL (APIs, emailed reports, ad-server exports), mapping-driven transforms, raw ↔ transformed reconciliation, unmapped-inventory checks |
| [Web analytics pipelines & reconciliation](web-analytics-pipelines.md) | GA4 Data API + BigQuery export across accounts, ETL vs GA4 vs tag reconciliation, freshness and definition bugs |
| [Content recommendations & contextual classification](content-recommendations-and-classification.md) | Incremental NLP classification (IAB taxonomy, sentiment, Gemini translation), recommendation pipeline served via Firestore/CDN |
| [Running a large Airflow estate](airflow-platform-operations.md) | Composer migration (Airflow 2.6 → 2.11), self-healing queue, failed-task digests, metadata cleanup, CI/CD |
| [SaaS revenue history & RAG on BigQuery](saas-revenue-data-and-rag.md) | Snapshot + MERGE ledger to outlive a 90-day API window; docs chunking + embeddings with BigQuery vector search |

**Public companion repositories**

- [gcp-data-platform-template](https://github.com/thanzap/gcp-data-platform-template): connectors, idempotent BigQuery
  loads, quality gates, Airflow 3, Terraform, CI/CD with Workload Identity Federation
- [ad-revenue-anomaly-agent](https://github.com/thanzap/ad-revenue-anomaly-agent): robust anomaly detection plus a
  Google ADK triage agent with evals
- [llm-document-extraction](https://github.com/thanzap/llm-document-extraction): LLM document extraction behind
  deterministic guardrails (bookkeeping)
- [padel_coach](https://github.com/thanzap/padel_coach): full-stack coaching platform plus pose-estimation video analysis
