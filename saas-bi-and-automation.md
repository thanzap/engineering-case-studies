# Data warehouse, billing automation and AI support tooling

**Domain:** B2B SaaS company selling subscriptions and hardware
**Stack:** GCP (BigQuery, Cloud Functions, Pub/Sub, Secret Manager, Firebase), Terraform, Python, SQL,
Google ADK + Gemini on Agent Runtime; REST/webhook integrations with the company's billing, CRM, shipping,
transactional-email, partner, banking and asset-management platforms

> Employer and vendor names are omitted and the source code is proprietary. This page covers the problems and the architecture.

## Context

The company sells subscriptions and hardware. Revenue, customer and support data live in a dozen SaaS tools. The data team owns
the company warehouse and a growing set of automations between those tools.

## What the systems do

**1. Company data warehouse.**
- Replication of billing (MRR, entitlements, ramps, features), CRM (deals, line items), marketplace and banking data
  into BigQuery via Cloud Functions; scheduled queries managed in Terraform.

**2. Billing and order automation (event-driven).**
- A billing service handling subscription-billing webhooks (subscriptions, payment sources, approved exclusions,
  auto-collection) and syncing subscription state to the product API, plus a sandbox twin for safe testing.
- Two-way sync between the CRM and the billing platform, with webhook listeners for billing events, so both systems
  hold the same customer and subscription state.
- CRM deal → shipping-platform order automation for hardware: closed-won deals become shipping orders (**~20 orders a
  day** created with no manual entry), with handling for CRM edge cases (for example, archived line items that the
  associations API still returns and that would double an order).
- A shipping webhook writes shipment details back to the shipment record in the CRM, so sales and support see
  delivery status without leaving it.
- Transactional email moved to a dedicated provider, with bounce/suppression processing and replay tooling.
- Security hardening: webhook token validation that fails closed, removal of legacy app passwords and unused secrets.

**3. AI support-ticket QA agent.**
- An ADK agent on Agent Runtime that scores support tickets against the team's QA rubric, **~150 tickets a day**.
  Tickets arrive via Pub/Sub, scores land in BigQuery, and the results feed ticket-performance evaluation and coaching
  summaries for the support team.
- Diagnosed a production incident where every evaluation came back null: an LLM proxy layer stopped round-tripping
  Gemini "thought signatures" after tool calls, once the model API started requiring them. Moved to the native
  Gemini client and restored service. Lesson recorded: agents need an eval harness in CI, not just uptime checks.

**4. Customer support portal.**
- A web application on Firebase where customers raise support requests. Users are authorised against the company's
  own product API, and each request becomes a ticket in the CRM with the relevant account and device details already
  filled in.

## Key engineering themes

- Owning **revenue-touching automations** end to end: webhooks, idempotency, sandboxes, alerting.
- Infrastructure as code for analytics (Terraform-managed scheduled queries and functions).
- Running **LLM agents as production services**, including debugging failures that only show in provider behaviour.
