# Engineering case studies

Write-ups of production systems I've built at work. Employer, client and vendor names are omitted and the code
itself is proprietary. Each one covers the problem, the architecture, the decisions I made, and links to public,
from-scratch versions of the core patterns.

| Case study | Domain | Highlights |
|---|---|---|
| [Ad-tech data platform & AI revenue triage](ad-tech-data-platform.md) | Programmatic advertising | 80+ Airflow DAG families on Composer, BigQuery, an ADK agent on Agent Runtime triaging revenue anomalies |
| [BI, billing automation & AI support QA](saas-bi-and-automation.md) | B2B SaaS + hardware | Company warehouse, billing/CRM/shipping automations, a ticket-QA agent |

**Public companion repositories**

- [gcp-data-platform-template](https://github.com/thanzap/gcp-data-platform-template): connectors, idempotent BigQuery
  loads, quality gates, Airflow 3, Terraform, CI/CD with Workload Identity Federation
- [ad-revenue-anomaly-agent](https://github.com/thanzap/ad-revenue-anomaly-agent): robust anomaly detection plus a
  Google ADK triage agent with evals
- [llm-document-extraction](https://github.com/thanzap/llm-document-extraction): LLM document extraction behind
  deterministic guardrails (bookkeeping)
- [padel_coach](https://github.com/thanzap/padel_coach): full-stack coaching platform plus pose-estimation video analysis
