# Solution Summary

## Two Systems, Not One

This is not a single agent. It's two systems that work together — one that runs continuously in the background, and one that responds in real time when a customer needs help.

---

## The Knowledge Pipeline (Background)

Runs on a schedule. Reads every data source we have — Salesforce cases, Zendesk history, Jira, GitLab docs, Confluence, the support guide — and builds a searchable knowledge index.

This is the part most people skip, and why most AI support tools underperform. The agent is only as useful as what it can look up. Raw tickets and scattered docs aren't searchable in a meaningful way. The pipeline's job is to process that raw material into something the agent can actually use.

It runs async, separate from any customer interaction. When it's done, the index is ready.

---

## The Support Agent (Real Time)

Triggered when a case is created in Salesforce. Phase 1 is support-team-facing only. Guided intake comes in Phase 3. A customer-facing surface comes in Phase 4 with the exact form still open: free self-service deflection, a paid self-healing tier, or some combination of the two.

What it does, in order:

**1. Ingest case data**
Pulls case description, attachments, and customer account context from Salesforce.

**2. Parse logs and customer-submitted content**
Extracts timestamps, error codes, and identifiers from attached logs and pasted error output so the next steps have something structured to work with.

**3. Correlate with Datadog**
Queries the Datadog MCP for time-correlated and customer-correlated server-side signal during the relevant window. Errors, anomalies, recent deployments, related alerts.

**4. Search the knowledge index**
Looks up similar past cases in the index built by the knowledge pipeline. Existing Salesforce cases, Zendesk archive, Jira.

**5. Synthesize findings**
The LLM combines the customer's evidence, the Datadog signal, and the historical matches into a structured findings document: a one-line summary, the supporting evidence with citations, a confidence indicator, and a list of suggested next investigative actions.

**6. Deliver to the support engineer**
Findings show up wherever the team works the case (Slack, case comment, email; decision pending). The same findings can flow to engineering if the support engineer escalates.

Customer-facing drafts come in Phase 2. The agent does not write to a customer in Phase 1. A customer-facing surface (free deflection, paid self-healing tier, or both) comes in Phase 4.

---

## What the Agent Does Not Do

- Send responses to customers without human approval in Phases 1 through 3 (permanent design constraint for the support workflow)
- Make decisions about escalation or priority
- Access systems it hasn't been explicitly connected to
- Replace the support team's judgment on complex issues
- Write a draft customer response in Phase 1 (added in Phase 2)
- Talk to customers directly in Phases 1 through 3 (added in Phase 4, form TBD)
