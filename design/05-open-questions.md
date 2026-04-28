# Open Questions

These are decisions that need input from engineering, ops, or leadership before Phase 1 can start. Organized by who needs to answer them.

For the engineering and ops items, each question now has a current recommendation in italics. Treat these as a starting position to react to, not a fait accompli. The faster way to get a decision is usually to give someone something concrete to disagree with.

---

## Engineering / Ops

**Data store selection**
What vector database do we use? Options depend on what infrastructure already exists. Chroma and Qdrant run in Docker and require no existing database. pgvector runs inside Postgres — if we have a managed Postgres instance, this may be the path of least resistance. See [ADR-002](../decisions/adr-002-data-store-tbd.md).

*Recommendation:* If we already run a managed Postgres anywhere, use pgvector. Otherwise go with Qdrant. It runs as a single Docker container, holds up under production load, and the metadata filtering is good. Chroma is fine for a throwaway prototype but I wouldn't run a production index on it. Skip Pinecone and Weaviate for now and reconsider only if self-hosted maintenance starts to hurt.

**Orchestration tooling**
Do we have an existing scheduler or pipeline tool? If engineering already runs Airflow or Prefect, we should use it. If not, n8n is the current default assumption.

*Recommendation:* Use whatever you already run. If there's a working Airflow or Prefect setup, the pipeline goes there. Adopting n8n on top of that adds a second tool no one asked for. If there's nothing in place, n8n stays as the default. The visual workflow plus the native Salesforce, Jira, and GitLab nodes mean I can keep this maintained without ops being on the hook for every change. The known weakness is that n8n isn't as battle-tested as Airflow for heavy data pipelines, so revisit if Phase 1 volume turns out higher than expected.

**Secrets management**
How do we store and access API credentials for Zendesk, Jira, GitLab, Salesforce, and the LLM provider? This needs to be answered before any connector is built.

*Recommendation:* Whatever's already in use. AWS Secrets Manager, Vault, GCP Secret Manager, Doppler, 1Password Connect, any of them works. If nothing's in place, I'd reach for Doppler or 1Password Connect because both have low setup cost and don't require standing up extra infrastructure. Two hard rules either way: no .env files baked into images, nothing committed to git.

**Hosting environment**
Where does n8n live? Where does the vector database live? Same server, separate services, cloud-hosted? Ops needs to scope this.

*Recommendation:* Co-locate everything next to the existing Zendesk archive container. Same host, same ops process, same backup pattern. n8n, the vector database, and the extraction workers all fit on one moderately-sized box for the volume we're looking at in Phase 1. We can split them across hosts later if load demands it, but starting consolidated keeps the surface area small and reuses ops familiarity with that environment.

**Zendesk Docker ownership**
The Zendesk archive is running in a Docker container. Who owns that going forward — support or ops? What's the access pattern for the data collector?

*Recommendation:* Ops owns the container. Patches, backups, uptime, restarts all sit with ops. Support owns the data semantics: what's in there, what's good, what's noise, what gets flagged for re-extraction when the prompt changes. The data collector reads via a service account with a read-only API token. Clear lanes for both teams.

---

## Support / Leadership

**Human review queue location**
Does the draft review happen in Slack, email, or back inside Salesforce as a task? This is a workflow question as much as a technical one. Slack is fastest. Salesforce keeps everything in one place. Email is familiar but adds friction.

**Deflection success definition**
What counts as a successful deflection for reporting purposes? Customer clicks "yes this helped"? Session ends without a ticket? We need a definition before we can measure it.

**Support guide readiness**
The support guide is a key input to both the deflection step and the guided intake. How current is it? Does it need to be reviewed and updated before Phase 1, or do we use it as-is and treat it as a living document?

**Confluence inclusion**
Confluence content is noted as low-confidence. Is there a subset of pages worth cleaning up and including in Phase 1, or do we exclude it entirely until quality improves?

---

## LLM / AI

**LLM provider selection**
Current assumption is Anthropic (Claude) via API. This should be confirmed and a budget estimate attached before Phase 1 spend begins.

**Initial indexing cost estimate**
Processing the full Zendesk archive through an LLM extraction pass will have a one-time API cost. This needs to be estimated against the actual record count before committing.

**Confidence threshold**
What confidence level triggers a flag on the draft response? This will need to be tuned against real cases once Phase 1 is running.
