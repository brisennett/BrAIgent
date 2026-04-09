# Open Questions

These are unresolved decisions that need input from engineering, ops, or leadership before Phase 1 can start. Organized by who needs to answer them.

---

## Engineering / Ops

**Data store selection**
What vector database do we use? Options depend on what infrastructure already exists. Chroma and Qdrant run in Docker and require no existing database. pgvector runs inside Postgres — if we have a managed Postgres instance, this may be the path of least resistance. See [ADR-002](../decisions/adr-002-data-store-tbd.md).

**Orchestration tooling**
Do we have an existing scheduler or pipeline tool? If engineering already runs Airflow or Prefect, we should use it. If not, n8n is the current default assumption.

**Secrets management**
How do we store and access API credentials for Zendesk, Jira, GitLab, Salesforce, and the LLM provider? This needs to be answered before any connector is built.

**Hosting environment**
Where does n8n live? Where does the vector database live? Same server, separate services, cloud-hosted? Ops needs to scope this.

**Zendesk Docker ownership**
The Zendesk archive is running in a Docker container. Who owns that going forward — support or ops? What's the access pattern for the data collector?

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
