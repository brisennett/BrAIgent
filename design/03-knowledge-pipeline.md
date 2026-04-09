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
- Vector similarity search (semantic / meaning-based)
- Metadata filtering (filter by source, record type, product area, date)
- Write path from the pipeline
- Read path from the sync agent
- Record updates without full re-index

Technology selection is deferred to ops/engineering. See [ADR-002](../decisions/adr-002-data-store-tbd.md).
