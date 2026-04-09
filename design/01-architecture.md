# Architecture

## Overview

The system has two distinct layers that share a data store but operate independently.

```
┌─────────────────────────────────────────────────────┐
│                 KNOWLEDGE PIPELINE                  │
│           Async — runs on a schedule                │
│                                                     │
│  Source Connectors → Processing Engine → Index      │
└──────────────────────────┬──────────────────────────┘
                           │ writes to
                    ┌──────▼──────┐
                    │  DATA STORE │
                    │ Vector Index│
                    └──────┬──────┘
                           │ reads from
┌──────────────────────────▼──────────────────────────┐
│                   SYNC AGENT                        │
│             Real time — per interaction             │
│                                                     │
│  SF Trigger → Orchestrator → LLM → Human Review    │
└─────────────────────────────────────────────────────┘
```

---

## Principles

**External to Salesforce.** Salesforce is a trigger and a data source. It is not the brain. This keeps the system portable if the CRM changes.

**Async before sync.** The knowledge pipeline runs independently of any customer interaction. By the time the agent needs to look something up, the work is already done.

**Human in the loop.** No response reaches a customer without rep approval. This is a design constraint, not a roadmap item.

**Component boundaries are explicit.** Each component has one job. Orchestration does not process data. Connectors do not store data. This makes it easier to swap out individual pieces as requirements evolve.

---

## Component Summary

| Component | Job | Layer |
|---|---|---|
| Orchestration Engine | Schedules and sequences jobs, handles failures | Both |
| Source Connectors | Pull raw data from each API | Pipeline |
| Processing Engine | Cleans, chunks, extracts via LLM | Pipeline |
| Data Store | Persists structured knowledge for search | Shared |
| SF Trigger | Fires on case creation or EC submission | Sync |
| Sync Agent | Searches index, drafts response, routes to human | Sync |
| Human Review Queue | Slack or email — rep approves before send | Sync |

---

## Technology Decisions

Some decisions are made. Others depend on what ops and engineering already have.

**Decided:**
- Orchestration: n8n (self-hosted, Docker-native, visual workflow builder)
- LLM: Anthropic API (Claude) via API calls
- Salesforce integration: Flow + HTTP callout + Apex REST class

**To be determined with ops/engineering:**
- Vector database (Chroma, Qdrant, or pgvector depending on existing stack)
- Hosting environment for n8n and the data store
- Secrets management for API credentials
- Whether an existing data pipeline or message queue can be reused

See [ADR-001](../decisions/adr-001-external-orchestration.md) and [ADR-002](../decisions/adr-002-data-store-tbd.md) for the reasoning behind these decisions.
