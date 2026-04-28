# Non-Functional Requirements

## What This Doc Is For

The other design docs cover what the system does. This one covers how well it has to do it: latency, throughput, availability, and the operational targets that engineering and ops need to size against.

Numbers below are starting positions. Some are calibrated against current support volume. Some are educated guesses I want pushback on. None of them should be treated as final until the team confirms.

---

## Throughput

**Knowledge pipeline (async).** A full initial pass processes Phase 1 volume (roughly 80,000 chunks) within one off-hours window of approximately 8 hours. Delta runs complete within 30 minutes for an expected daily delta of low hundreds of records.

**Sync agent (real time).** One concurrent customer interaction at a time per Phase 2 case creation rate. No requirement for parallel handling of multiple cases in Phase 2. This can grow if Phase 3 deflection moves the workload upstream.

---

## Latency

**Sync agent end-to-end (case creation to draft delivered to rep).** Target under 60 seconds median, under 120 seconds p95. The rep is not actively waiting at this point, so single-digit seconds is not required, but anything above 2 minutes risks the draft showing up after the rep has already started working the case manually.

**Vector index search.** Under 500 milliseconds for a typical query. Both pgvector and Qdrant comfortably hit this at Phase 1 volume.

**LLM call (extraction or draft).** Bounded by the Anthropic API. Typical observed latency is 5 to 30 seconds depending on response length. The orchestrator should treat this as the dominant time cost in the sync agent flow.

---

## Availability

**Knowledge pipeline.** Best-effort. A scheduled run failing once is acceptable as long as the next run picks up the missed delta. An entire day of failed runs without alerting is not acceptable.

**Sync agent.** Should be available during business hours of the support team. A Phase 2 outage means reps work cases manually without a draft, which is a degradation rather than an outage. No customer-facing impact.

**Phase 3 (deflection in Experience Cloud) raises this bar.** When the agent is in the customer path, downtime means a worse customer experience. Higher availability target to be defined when Phase 3 scoping begins.

---

## Recoverability

**Index data.** Reproducible from source. If the index is lost, the cost of rebuilding is dominated by the LLM extraction pass (see knowledge pipeline cost estimate). A full rebuild is therefore expensive but possible without data loss. Daily snapshots of the index avoid the rebuild cost in most failure modes.

**Pipeline run history and audit logs.** Backed up daily. Retained per the org's audit policy, default 12 months unless contractual requirements differ.

**Salesforce trigger and Apex REST class.** Already covered by Salesforce backup and DR. No additional concern.

---

## Scaling

Phase 1 fits comfortably on a single moderately-sized host. The growth vectors are:

**Source volume.** If a new data source comes online (e.g. a customer-facing community, additional Salesforce orgs), processing volume grows proportionally. Pipeline scales horizontally by running more extraction workers if needed.

**Sync agent concurrency.** If Phase 3 deflection happens, simultaneous customer sessions grow with traffic to Experience Cloud. The bottleneck shifts to LLM API rate limits before anything in our infrastructure becomes the constraint.

**Index size.** pgvector and Qdrant both handle indexes an order of magnitude larger than current Phase 1 volume on the same hardware. The next bottleneck is search latency, which is mitigated with appropriate index tuning when it becomes a problem rather than now.

---

## Observability Targets

(Detail in [knowledge-pipeline.md](03-knowledge-pipeline.md). Summarized here.)

- Pipeline run status, record counts, and token spend logged per run.
- Health signal answering "did the last run succeed and how many records were added in the last 24 hours."
- Slack alerts on run failure, zero-record runs when records were expected, and daily token spend exceeding 2x rolling average.
- Sync agent end-to-end timing logged per draft.
- Quarterly manual sample of extracted records to catch silent quality drift.

---

## Security and Access Control

- All external API calls authenticated with credentials stored in the secrets manager, not env files or code.
- Read-only API tokens for source connectors wherever the API supports it.
- The vector index is not exposed externally. Reads come from the orchestrator only.
- The sync agent is triggered exclusively by Salesforce, with the trigger payload signed or shared-secret authenticated. No public endpoint.
- Audit log of every LLM call as described in [Data Handling](06-data-handling.md).

---

## What's Explicitly Not in Scope for Phase 1

- High availability of the sync agent. Single-instance is fine for Phase 2 launch.
- Multi-region deployment. Single region until business need or compliance requirement says otherwise.
- Real-time index updates. Async with a daily delta is the design and that's intentional.
- SLAs to customers. Nothing in the agent is customer-facing in Phase 1 or 2.

---

## Open Questions

1. Is there an existing on-call rotation that would absorb pipeline alerting, or do we need to set up a new one?
2. What's the org's standard for service-level monitoring (Datadog, Grafana, Prometheus, something else)? The pipeline should emit metrics to whatever already exists.
3. Are there cost-control mechanisms in place for LLM spend (budget alerts, hard caps)? If not, Phase 1 needs at least a soft alert at the budget threshold.
