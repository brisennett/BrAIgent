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

Triggered when a customer starts an interaction — either through Experience Cloud before a ticket exists, or when a case is created in Salesforce.

What it does, in order:

**1. Deflection attempt**
Reads the customer's description and checks it against public docs, playbooks, and the support guide. If it looks like a how-to or documentation question, it surfaces an answer and asks if that resolves it. If yes — no ticket created.

**2. Guided intake**
If deflection doesn't work, the agent collects structured information before the case is submitted. Version numbers, error messages, logs, relevant context. The support guide feeds what questions to ask based on the type of issue.

**3. Investigation and draft**
Once a case exists, the agent searches the knowledge index — existing SF cases, Zendesk history, Confluence, Jira — and drafts a response based on what it finds.

**4. Human review**
The draft goes to the rep. They review it, edit if needed, and approve before anything is sent to the customer.

---

## What the Agent Does Not Do

- Send responses to customers without human approval
- Make decisions about escalation or priority
- Access systems it hasn't been explicitly connected to
- Replace the support team's judgment on complex issues
