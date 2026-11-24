# Business intelligence, billing automation and AI support QA

**Role:** [TODO: title] · **2023–present** · B2B SaaS company selling subscriptions and hardware
**Stack:** GCP (BigQuery, Cloud Functions, Pub/Sub, Secret Manager, Firebase), Terraform, Python, SQL, Looker Studio,
Google ADK + Gemini on Agent Runtime; REST/webhook integrations with the company's billing, CRM, shipping,
transactional-email, partner, banking and asset-management platforms

> Employer and vendor names are omitted and the source code is proprietary. This page covers the problems, the architecture and my part in it.

## Context

The company sells subscriptions and hardware. Revenue, customer and support data live in a dozen SaaS tools. The BI team owns
the company warehouse, the dashboards every department uses, and a growing set of automations between those tools.

## What I built and run

**1. Company data warehouse and dashboards.**
- Replication of billing (MRR, entitlements, ramps, features), CRM (deals, line items), marketplace and banking data
  into BigQuery via Cloud Functions; scheduled queries managed in Terraform.
- Looker Studio dashboards for sales, finance, product and support. [TODO: e.g. "used by N people / every department"]

**2. Billing and order automation (event-driven).**
- A billing service handling subscription-billing webhooks (subscriptions, payment sources, approved exclusions,
  auto-collection) and syncing subscription state to the product API, plus a sandbox twin for safe testing.
- CRM deal → shipping-platform order automation for hardware: closed-won deals become shipping orders, with handling
  for CRM edge cases (for example, archived line items that the associations API still returns and that would
  double an order).
- Transactional email moved to a dedicated provider, with bounce/suppression processing and replay tooling.
- Security hardening: webhook token validation that fails closed, removal of legacy app passwords and unused secrets.
- [TODO: impact, e.g. "N orders/month created without manual entry", "billing sync errors down X %"]

**3. AI support-ticket QA agent.**
- An ADK agent on Agent Runtime that scores support tickets against the team's QA rubric. Tickets arrive via Pub/Sub,
  scores land in BigQuery, and coaching summaries go to the support team.
- Diagnosed a production incident where every evaluation came back null: an LLM proxy layer stopped round-tripping
  Gemini "thought signatures" after tool calls, once the model API started requiring them. Moved to the native
  Gemini client and restored service. Lesson recorded: agents need an eval harness in CI, not just uptime checks.
- [TODO: impact, e.g. "100 % of tickets scored vs ~X % manual sample"]

## What I'd highlight in an interview

- Owning **revenue-touching automations** end to end: webhooks, idempotency, sandboxes, alerting.
- Infrastructure as code for analytics (Terraform-managed scheduled queries and functions).
- Running **LLM agents as production services**, including debugging failures that only show in provider behaviour.
