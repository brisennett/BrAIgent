# BrAIgent — Project Context for Claude

This file gives Claude the context needed to pick up where we left off without re-explaining the entire project.

---

## What This Is

An AI-assisted support agent for an internal support team. Built external to Salesforce by design. Two layers: an async knowledge pipeline that indexes historical sources on a schedule, and a real-time sync agent that ingests new cases, correlates them with live server-side data, and surfaces investigative findings to the support engineer.

GitHub repo: https://github.com/brisennett/BrAIgent

---

## Who Is Building This

Brian — support team lead, Salesforce admin/developer background. Comfortable with Flows, Apex, HTTP callouts, and Docker. Not a traditional developer but has built real integrations. Works solo on this with ops support for deployment.

---

## Key Decisions Already Made

**External orchestration.** The agent is built outside Salesforce. Salesforce is a trigger and data source only. Reason: active HubSpot migration rumors, portability matters. See `decisions/adr-001-external-orchestration.md`.

**Orchestration engine.** n8n, self-hosted, Docker-native. Subject to override if the org already runs Airflow or Prefect.

**LLM.** Anthropic (Claude) via API.

**Data store.** Proposed: pgvector if a managed Postgres exists, Qdrant otherwise. Pending ops/engineering confirmation. See `decisions/adr-002-data-store-tbd.md`.

**MCP for live data sources.** Live data sources queried at investigation time (Datadog first, others later) integrate via MCP servers in our infrastructure. See `decisions/adr-003-mcp-pattern.md`.

**Human review gate.** Non-negotiable. No response reaches a customer without support engineer approval. Always.

---

## Existing Assets We Are Building On

- Salesforce Flow + Apex REST class already in production for support portal onboarding. Same pattern the agent trigger will use.
- Zendesk archive running in Docker, queryable via API. Built in a prior session.
- GitLab public docs. Accessible, WIP but usable.
- Jira. Best structured data source for known issues.
- Confluence. Low confidence, deprioritized.
- Support guide. Exists, feeds triage logic and guided intake (later phase).
- Datadog. Already in our stack as a vendor; the MCP integration is what's new.

---

## Architecture Summary

```
KNOWLEDGE PIPELINE (async, scheduled)
Source Connectors → Processing Engine (LLM extraction) → Vector Index

SYNC AGENT (real time, per case)
SF Trigger → Ingest Case → Parse Logs → Datadog (MCP) + Vector Index → LLM Synthesis → Findings to Support engineer
```

The pipeline writes to the data store. The agent reads from it during a case investigation alongside live Datadog signal and the customer's own attachments. They share the store but operate independently.

---

## Sync Agent Flow (Phase 1, in order)

1. **Trigger.** SF Flow fires on case creation, calls the agent via HTTP.
2. **Ingest case data.** Pull case description, attachments, and customer account context from Salesforce.
3. **Parse logs.** Extract timestamps, error codes, identifiers from customer-submitted logs and pasted error output.
4. **Correlate with Datadog.** Query Datadog via MCP for time-correlated, customer-correlated server-side signal.
5. **Search knowledge index.** Find similar past cases across SF, Zendesk archive, Jira.
6. **LLM synthesis.** Combine all of the above into a structured findings document: one-line summary, evidence with citations, confidence indicator, suggested next investigative actions.
7. **Deliver to support engineer.** Findings show up in Slack, case comment, or email (decision pending). Same findings flow to engineering on escalation.

Customer-facing drafts come in Phase 2. A customer-facing surface (free deflection, paid self-healing tier, or both) comes in Phase 4.

---

## Phased Roadmap

- **Phase 1.** Two parallel workstreams.
  - **Workstream A.** Investigative aid: SF trigger plus log parsing plus Datadog MCP plus LLM synthesis, producing findings to the support engineer.
  - **Workstream B.** Knowledge pipeline: data store, Zendesk/GitLab/Jira indexed, validated index quality. Feeds Workstream A.
- **Phase 2.** Customer response drafting on top of the Phase 1 findings.
- **Phase 3.** Guided intake. Structured prompts collect case context before submission.
- **Phase 4.** Customer-facing surface, exact form TBD. Could be free self-service deflection in Experience Cloud, a paid self-healing tier in the customer's admin console, or both. Product and business decision, not a support team decision. Strategic note in the roadmap doc.

**Current status: Phase 1 not started. Design complete.**

---

## What's Blocking Phase 1

Ops and engineering need to answer the open questions in `design/05-open-questions.md`. The Phase 1 prerequisites specifically: data store selection, hosting environment, Datadog MCP availability, log format coverage, attachment handling policy, findings delivery channel.

---

## Doc Structure

```
/overview/          leadership-facing, non-technical
/design/            technical design, ops and engineering audience
/decisions/         architecture decision records, why we chose what we chose
```

---

## How to Use This File

Paste the contents of this file at the start of a new Claude session to restore project context. Then continue from wherever the work left off.
