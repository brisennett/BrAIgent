# ADR-002 — Data Store Selection

**Status:** Pending  
**Date:** April 2026

---

## Decision

Not yet made. This document captures the requirements and options so engineering and ops can make an informed recommendation.

---

## Requirements

The data store must support:

- **Vector similarity search** — find records by meaning, not keyword
- **Metadata filtering** — filter results by source, record type, product area, date
- **Write path from the pipeline** — the async data collector writes to it on a schedule
- **Read path from the sync agent** — the real-time agent queries it per interaction
- **Record updates** — new pipeline runs should update existing records, not duplicate them
- **Docker deployment** — consistent with existing infrastructure approach

Nice to have:
- Managed hosting option if self-hosted maintenance becomes a burden
- Built-in UI for inspecting index contents during development

---

## Options

### Chroma
- Simplest setup — single Docker container, minimal config
- Good for getting started quickly
- Less battle-tested at scale
- Best fit if we want to move fast in Phase 1 and revisit later

### Qdrant
- More production-ready than Chroma
- Good filtering capabilities on metadata fields
- Slightly more config overhead than Chroma
- Best fit if ops wants something more robust from the start

### pgvector (Postgres extension)
- Runs inside an existing Postgres instance
- No new infrastructure if we already have managed Postgres
- Familiar tooling for engineering teams already using Postgres
- Best fit if we have an existing managed Postgres we can extend

### Pinecone / Weaviate (managed cloud)
- Fully managed — no infrastructure to run
- Additional cost and an external dependency
- Best fit if self-hosted maintenance is a concern and budget allows

---

## Questions for Ops/Engineering

1. Do we have an existing managed Postgres instance? If yes, pgvector is likely the lowest-friction path.
2. What is our tolerance for self-hosted database maintenance?
3. Is there an existing data infrastructure team or tooling that should own this?
4. What does our backup and recovery story look like for a new data store?

---

## Revisit Conditions

This ADR should be completed once ops and engineering have reviewed the options above and answered the questions. The data store decision gates Phase 1 — nothing can be indexed until it's resolved.
