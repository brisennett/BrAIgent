# Data Collector

## What It Is

The data collector is the async half of the system. It runs on a schedule, connects to every data source we have, pulls raw content, and feeds it into the processing engine. It has no interaction with customers and no awareness of real-time events.

Think of it as the research team that works overnight so the agent has something to work with during the day.

---

## Components

### Orchestration Engine

Responsible for:
- Scheduling runs (e.g. nightly, or on-demand)
- Sequencing connector jobs in the right order
- Passing outputs from one stage to the next
- Handling failures gracefully — one source being down should not kill the entire run
- Logging what ran, when, how many records were processed, and what failed

### Source Connectors

One per data source. Each connector is responsible for:
- Authenticating with the source API
- Determining what to pull (full sync vs. delta since last run)
- Respecting rate limits
- Returning raw records to the processing engine

| Source | What Gets Pulled | Auth Method | Priority |
|---|---|---|---|
| Zendesk Archive | Ticket threads, resolutions | API token | Phase 1 |
| GitLab Docs | Documentation pages | Personal access token | Phase 1 |
| Jira | Issues, status, workarounds | API token | Phase 1 |
| Salesforce | Case history, customer context | Connected App / OAuth | Phase 2 |
| Confluence | KB pages | API token | Phase 2 |

### Processing Engine

Raw data from connectors is not directly useful. A Zendesk ticket thread is a back-and-forth conversation — the processing engine's job is to extract meaning from it.

For each record it receives, it:
1. Cleans the raw content (strips HTML, removes noise)
2. Runs it through an LLM extraction prompt to produce structured output
3. Chunks large documents into smaller pieces
4. Passes structured chunks to the data store for indexing

The extraction prompt varies by source type. A ticket thread is processed differently than a GitLab doc page.

**Example — Zendesk ticket extraction output:**
```
problem_summary: "Customer unable to authenticate after SSO config change"
symptoms: ["401 errors on login", "affects all users in org", "started after admin changed IdP settings"]
product_area: "Authentication / SSO"
resolution: "Reverted IdP entity ID to match SP metadata. Customer confirmed resolved."
resolution_type: "configuration"
```

This structured record — not the raw thread — is what gets indexed.

---

## Data Flow

```
Schedule fires
      ↓
Orchestrator starts connector jobs
      ↓
Each connector pulls records from its source API
      ↓
Records passed to processing engine
      ↓
LLM extracts structured fields from each record
      ↓
Structured records chunked if needed
      ↓
Chunks written to data store / vector index
      ↓
Run log updated — records processed, errors, duration
```

---

## Open Questions for Ops/Engineering

Before choosing specific technology, the following questions should be answered:

- Do we have an existing orchestration tool? (Airflow, Prefect, cron, etc.)
- Is there a managed database we standardize on that could support vector storage?
- Do we have a secrets manager for API credentials?
- Is there an existing message queue or event bus that could replace the n8n scheduler?
- What is our preferred hosting environment — cloud provider, self-hosted VPS, on-prem?

The component model above is technology-agnostic. Technology selection should follow from what already exists.
