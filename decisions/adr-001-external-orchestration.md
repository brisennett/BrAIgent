# ADR-001 — External Orchestration

**Status:** Accepted  
**Date:** April 2026

---

## Decision

The agent orchestration layer will be built outside of Salesforce, not within it.

---

## Context

Salesforce offers native AI tooling — Agentforce and Einstein Copilot — that could theoretically handle some of this. The question was whether to build inside Salesforce's ecosystem or outside it.

Key factors:
- The org migrated from Zendesk to Salesforce six months ago. There are active rumors of a potential future move to HubSpot.
- Building inside Salesforce creates tight coupling. Migrating the agent would mean rebuilding it.
- Salesforce AI features require specific licensing that is not currently in place.
- The team has existing comfort with self-hosted Docker workloads, which maps well to external tooling.

---

## Decision Rationale

Building externally keeps the CRM as a trigger and data source rather than the system of intelligence. The orchestration logic, knowledge index, and LLM layer remain independent of whatever CRM is in use.

If the org moves to HubSpot, only the trigger integration and the Salesforce data connector need to change. Everything else carries over.

---

## Trade-offs

**What we give up:**
- Native Salesforce UI integration (agents and drafts would need to surface via Slack/email rather than inline in the case record — at least initially)
- Agentforce-specific features and Salesforce-managed LLM infrastructure
- Reduced implementation complexity if Agentforce licensing were available

**What we gain:**
- CRM portability
- Control over the full stack
- No dependency on Salesforce licensing decisions
- Ability to connect to any data source, not just Salesforce-native ones

---

## Revisit Conditions

This decision should be revisited if:
- The org commits to Salesforce long-term with no active CRM migration discussions
- Agentforce licensing becomes available and the native tooling closes the feature gap
- The external orchestration approach proves significantly more complex to maintain than anticipated
