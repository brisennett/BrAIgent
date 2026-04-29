# ADR-003 — MCP as the Pattern for Live Data Sources

**Status:** Proposed (pending engineering confirmation)  
**Date:** April 2026

---

## Decision

Live data sources that the sync agent queries during a case investigation are integrated via Model Context Protocol (MCP) servers running in our infrastructure, rather than via direct API clients written into the orchestrator.

The first instance of this pattern is the Datadog MCP introduced in Phase 1. The same pattern applies to any future live data source the agent needs to query at investigation time.

This is distinct from the indexed sources (Zendesk, GitLab, Jira, Salesforce historical cases). Those are pulled by source connectors in the knowledge pipeline and stored in the vector index. They don't go through MCP.

---

## Context

The Phase 1 investigative aid needs server-side context from Datadog at investigation time. That signal can't be pre-indexed because it's time-correlated to the customer's reported window and the customer hasn't reported it yet when the pipeline runs.

We have three plausible patterns for getting Datadog data into the agent flow:

1. Direct Datadog API client written into the n8n workflow.
2. A Datadog MCP server running in our infrastructure.
3. A third-party hosted Datadog MCP.

Each comes with different ergonomics, different ops surface, and different reuse properties when the next live data source comes along.

---

## Decision Rationale

The MCP pattern wins on three things.

**Reusability.** When the next live data source shows up (Sentry, PagerDuty, an internal log aggregator, the customer's own product environment), the integration shape is the same: stand up or adopt an MCP server, point the orchestrator at it. We're not rewriting integration plumbing every time. A direct API client buys nothing reusable.

**Separation of concerns.** The MCP server speaks to Datadog. The orchestrator speaks to the MCP. The LLM speaks through the orchestrator. Each layer has one job. When Datadog changes its API, only the MCP changes. When we change orchestrators or add a new data source, only the orchestrator changes.

**Ecosystem.** MCP has working server implementations for many of the systems we'd want to query (Datadog, Sentry, GitHub, Linear, Slack, PagerDuty, others). Adopting an existing implementation is cheaper than writing a new API client.

The third-party hosted MCP option is rejected outright. Datadog credentials and observability data are too sensitive to route through a vendor we don't control. If we need to host an MCP server, we host it ourselves.

---

## Trade-offs

**What we give up:**
- An additional process to deploy and monitor (the MCP server itself).
- A small additional latency hop on every investigation call (typically negligible, but real).
- A dependency on the specific MCP server implementation being maintained, or our willingness to maintain a fork.

**What we gain:**
- Reusable pattern for the next live data source.
- Clean separation between source integration and orchestration.
- Datadog credentials stay in our infrastructure, not embedded in n8n workflow exports.
- Ability to swap out orchestrators (n8n today, possibly Airflow or Prefect later) without rewriting the data integration.

---

## What This Doesn't Cover

This ADR is specifically about the integration pattern for live data sources during a case investigation. It does not cover:

- Indexed sources (Zendesk, GitLab, Jira, Salesforce historical). Those are handled by the knowledge pipeline's source connectors.
- The Salesforce trigger or the Apex REST endpoint used for case ingestion. Those are existing integrations and the pattern is already established.
- The LLM API. Anthropic is called directly by the orchestrator, not through an MCP. The orchestrator is the agent in the MCP sense, and the LLM is the model the agent uses.

---

## Revisit Conditions

This ADR moves from Proposed to Accepted once engineering confirms the hosting pattern for the Datadog MCP and we have at least one working MCP integration in production.

Revisit if:
- A vendor we trust offers a hosted MCP with terms acceptable to legal, and the operational savings outweigh the loss of control.
- The MCP ecosystem stops being actively maintained, in which case the reusability argument weakens.
- We end up with only one live data source long-term and the reusability gain isn't realized. In that case, a direct API client may have been the right call all along.
