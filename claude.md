# BrAIgent — Project Context for Claude

This file gives Claude the context needed to pick up where we left off without re-explaining the entire project.

---

## What This Is

An AI-assisted support agent for an internal support team. Built external to Salesforce by design. The system has two layers — an async knowledge pipeline that runs on a schedule, and a real-time sync agent that handles customer interactions.

GitHub repo: https://github.com/brisennett/BrAIgent

---

## Who Is Building This

Brian — support team lead, Salesforce admin/developer background. Comfortable with Flows, Apex, HTTP callouts, and Docker. Not a traditional developer but has built real integrations. Works solo on this with ops support for deployment.

---

## Key Decisions Already Made

**External orchestration** — the agent is built outside Salesforce. Salesforce is a trigger and data source only. Reason: active HubSpot migration rumors, portability matters. See `decisions/adr-001-external-orchestration.md`.

**Orchestration engine** — n8n, self-hosted, Docker-native. Chosen for visual workflow builder, native Salesforce/Jira/GitLab nodes, and alignment with existing infrastructure comfort.

**LLM** — Anthropic (Claude) via API.

**Data store** — not yet decided. Pending ops/engineering input. Options: Chroma, Qdrant, pgvector. See `decisions/adr-002-data-store-tbd.md`.

**Human review gate** — non-negotiable. No response reaches a customer without rep approval. Always.

---

## Existing Assets We Are Building On

- Salesforce Flow + Apex REST class already in production for support portal onboarding — same pattern the agent trigger will use
- Zendesk archive running in Docker, queryable via API — built in a prior session with Claude
- GitLab public docs — accessible, WIP but usable
- Jira — best structured data source for known issues
- Confluence — low confidence, deprioritized
- Support guide — exists, feeds triage logic and guided intake

---

## Architecture Summary

```
KNOWLEDGE PIPELINE (async, scheduled)
Source Connectors → Processing Engine (LLM extraction) → Vector Index

SYNC AGENT (real time, per interaction)
SF Trigger → Orchestrator → Customer Context Lookup → Investigation → Draft → Human Review
```

The pipeline writes to the data store. The agent reads from it. They share the store but operate independently.

---

## Sync Agent Flow (in order)

1. **Deflection** — classify issue, search public GitLab docs and playbooks, surface answer if found. Jira is NOT used here.
2. **Guided intake** — if not deflected, collect structured info before case creation. Support guide feeds what to ask by issue category.
3. **Customer context** — pull segment, ARR, renewal status from Salesforce via Apex REST endpoint.
4. **Investigation** — search knowledge index: SF cases → Zendesk archive → Confluence → Jira (known bugs/workarounds).
5. **Draft generation** — LLM produces response with source references and confidence level.
6. **Human review** — rep receives draft via Slack or email, approves before anything is sent.

---

## Phased Roadmap

- **Phase 1** — Build the knowledge pipeline. Stand up data store, connect Zendesk/GitLab/Jira, validate index quality.
- **Phase 2** — Sync agent post-ticket. SF trigger → investigation → draft → human review.
- **Phase 3** — Pre-ticket deflection via Experience Cloud.
- **Phase 4** — Guided intake and automated case field population.

**Current status: Phase 1 not started. Design complete.**

---

## What's Blocking Phase 1

Ops and engineering need to answer the open questions in `design/05-open-questions.md` — specifically data store selection and hosting environment. Those answers gate everything else.

---

## Doc Structure

```
/overview/          ← leadership-facing, non-technical
/design/            ← technical design, ops and engineering audience
/decisions/         ← architecture decision records, why we chose what we chose
```

---

## How to Use This File

Paste the contents of this file at the start of a new Claude session to restore project context. Then continue from wherever the work left off.
