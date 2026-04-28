# Knowledge Pipeline

## The Problem It Solves

The raw data in our systems — ticket threads, doc pages, Jira issues — is not directly searchable in a meaningful way. Keyword search finds the word you typed. It doesn't find a ticket about "auth failure" when the customer typed "can't log in."

The knowledge pipeline converts raw content into a vector index — a format where search works on meaning, not keywords. This is what makes the agent's research useful rather than brittle.

---

## How Vector Search Works (Plain Terms)

When a document is indexed, it gets converted into a mathematical representation of its meaning — called an embedding. When the agent searches, the query gets converted the same way, and the index returns the records with the closest meaning match.

Two practical implications:

1. The index needs to be built before the agent can use it. This is why the pipeline runs async.
2. The quality of the index depends on the quality of what gets put into it. Garbage in, garbage out. This is why the processing step extracts structured fields rather than dumping raw text.

---

## What Gets Indexed

Each record in the index contains:

- **Text content** — the chunk of text that gets embedded and searched
- **Source metadata** — where it came from, when it was created, record ID
- **Structured fields** — extracted problem summary, symptoms, resolution, product area
- **Record type** — ticket, doc page, Jira issue, KB article

The structured fields allow the sync agent to filter results. For example: "find tickets in the Authentication product area that were marked resolved."

---

## Index Maintenance

**Initial load:** First run processes all historical records. This is the expensive pass — it makes a lot of LLM API calls. Run once, off-hours.

**Delta runs:** Subsequent runs only process records created or modified since the last run. Much cheaper and faster. This is what runs on the regular schedule.

**Re-indexing:** If the extraction prompt changes significantly, records may need to be re-indexed. This should be a deliberate decision, not something that happens automatically.

---

## Source-Specific Notes

**Zendesk archive**
The most valuable source for resolution patterns. Tickets have a clear problem/resolution structure even if it's buried in conversation threads. The extraction prompt needs to handle cases where the resolution is implied rather than stated explicitly (e.g. "customer confirmed working" with no explanation of what changed).

**GitLab docs**
Cleanest data. Pages are structured and authoritative. Chunk by section heading. High confidence in output quality.

**Jira**
Issues have structured fields (status, type, labels, fix version) that map cleanly to index metadata. The description and comment thread feed the text content. Particularly useful for the "is this a known bug?" lookup in the sync agent.

**Salesforce cases**
Likely thin on resolution detail — most resolution context lives in email threads or call notes that may not be captured. Index what exists but flag this as a gap. Quality will improve over time as cases accumulate post-migration.

**Confluence**
Treat as low-confidence. Index it, but weight it lower in search results until content quality is validated. A wrong answer pulled from a stale Confluence page is worse than no answer.

---

## Data Store Requirements

The data store needs to support:
- Vector similarity search (semantic, meaning-based)
- Metadata filtering (filter by source, record type, product area, date)
- Write path from the pipeline
- Read path from the sync agent
- Record updates without full re-index

Technology selection is in [ADR-002](../decisions/adr-002-data-store-tbd.md). Current proposal is pgvector if a managed Postgres exists, Qdrant otherwise.

---

## Volume Estimates

Numbers below are approximate and need to be confirmed against actual record counts. They're here so ops and engineering can size infrastructure rather than guess.

| Source | Estimated raw records | Chunks after extraction | Notes |
|---|---|---|---|
| Zendesk archive | ~50,000 tickets | ~75,000 chunks | One ticket may produce multiple chunks if the thread covers more than one issue |
| GitLab docs | ~300 pages | ~1,500 chunks | Chunk per section heading |
| Jira | ~5,000 issues | ~5,000 chunks | One chunk per issue, typically |
| **Phase 1 total** | **~55,000 records** | **~80,000 chunks** | |

A chunk is roughly 500 tokens, so the full Phase 1 index is on the order of 40 million tokens of indexed content. Storage footprint depends on the embedding dimensionality but plan for low single-digit GB including metadata, well within what a single Qdrant or pgvector instance handles comfortably.

**Sync agent query rate:** 1 query per case in Phase 2, same as Salesforce case creation rate. Estimate based on current support volume, not theoretical max.

---

## Embedding Model

The pipeline writes embeddings to the data store, but Anthropic does not currently offer a hosted embeddings API. We need a separate embedding model.

Two viable options:

**Voyage AI** (Anthropic's preferred embedding partner). Hosted API, low cost, good performance on retrieval benchmarks. Adds a second vendor relationship.

**Self-hosted open source** (e.g. BGE, E5). Free at inference time, runs in Docker, no extra vendor. Slower than hosted, more ops surface to maintain.

I'd default to Voyage for Phase 1 to keep ops surface low. Revisit if cost or vendor concentration becomes an issue.

---

## Cost Estimate (Phase 1, Initial Pass)

Back-of-envelope, using April 2026 Anthropic and Voyage list prices. Real numbers will vary.

**LLM extraction (one-time):**
- 55,000 records × ~1,500 input tokens + ~200 output tokens
- Roughly 85M input + 11M output tokens total
- Claude Sonnet at current pricing: approximately $250-400 one-time
- Claude Haiku alternative: approximately $50-75 one-time, with reduced extraction quality

**Embeddings (one-time):**
- 80,000 chunks × ~500 tokens = 40M tokens
- Voyage AI list pricing: approximately $5-10 one-time

**Delta runs (ongoing):**
- Process only records changed since last run
- Estimated daily delta: low hundreds of records across all sources
- Expected monthly cost: well under $50/month

**Recommendation:** Run the first pass with Sonnet to get good extraction quality. If results are clean, consider switching delta runs to Haiku to lower ongoing cost. Decision can be made after Phase 1 data is on the ground.

These numbers assume ticket volumes in the table above. Confirm actual record counts before committing budget.

---

## Observability

A pipeline this size will fail eventually. We need to know when, where, and how badly without inspecting databases by hand.

**Per-run logging.** Each scheduled run records: start time, end time, records pulled per source, records processed successfully, records failed, total tokens consumed, total cost. Stored somewhere queryable.

**Health signal.** A simple endpoint or dashboard tile that answers two questions: was the last run successful, and how many records were added in the last 24 hours. If the second number is zero on a day where records were expected, something is wrong.

**Alerts.** A scheduled run that fails outright. A run that completes but processes zero records. A daily token spend that exceeds 2x the rolling average. These get a notification in Slack to whoever's on duty.

**Index quality drift.** Periodically sample a few extracted records by hand and check that the structured fields are still being filled correctly. If the source format changes (new Zendesk schema, new Jira field) the extraction prompt may silently degrade. A quarterly review is enough at Phase 1 scale.
