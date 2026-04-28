# ADR-002 — Data Store Selection

**Status:** Proposed (pending ops/engineering confirmation)  
**Date:** April 2026

---

## Decision

Use **pgvector** if the org already runs a managed Postgres we can extend. If not, default to **Qdrant** in a Docker container.

This is a proposal, not a final call. Ops and engineering own the final selection and can override based on infrastructure context I don't have visibility into. The reasoning is below so the decision is something we can react to rather than something we have to invent from scratch.

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
- Simplest setup. Single Docker container, minimal config.
- Good for prototyping and demos.
- Less battle-tested at scale and the project has shipped breaking changes between versions in the past.
- I wouldn't run a production index on Chroma. Fine for a one-off proof of concept.

### Qdrant *(default if no managed Postgres)*
- More production-ready than Chroma.
- Good filtering on metadata fields, which we need.
- Slightly more config overhead than Chroma but still single-container deployable.
- Strong fit when ops wants something durable from day one and there's no existing Postgres to extend.

### pgvector *(default if managed Postgres exists)*
- Runs as an extension inside an existing Postgres instance.
- No new infrastructure if we already have managed Postgres.
- Familiar tooling for engineering teams already on Postgres. Backups, monitoring, and access patterns are already solved.
- The path of least operational resistance when the prerequisite holds.

### Pinecone / Weaviate (managed cloud)
- Fully managed, no infrastructure to run.
- Recurring cost plus an external dependency we don't have today.
- Reserve as the fallback if self-hosted maintenance becomes painful or scale exceeds what a single Qdrant box can handle.

---

## Questions for Ops/Engineering

These are the things I need confirmed to lock the decision in.

1. Do we have an existing managed Postgres instance? If yes, what version, and is the team comfortable enabling the pgvector extension on it?
2. If no Postgres, who would own a Qdrant container long-term? Same ops process as the existing Zendesk archive container, or a different team?
3. What's the backup and recovery expectation for the index? The data is reproducible from source if we lose it, but a full rebuild would mean another expensive LLM extraction pass.
4. Are there data residency or compliance constraints that rule out either option?

---

## Revisit Conditions

This ADR moves from Proposed to Accepted once ops and engineering confirm the four questions above. The data store decision gates Phase 1. Nothing can be indexed until it's resolved.

If the volume estimates in the knowledge pipeline doc turn out to be off by an order of magnitude in either direction, revisit. pgvector and Qdrant both scale to the volume currently expected, but a 10x change in either direction shifts the calculus.
