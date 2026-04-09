# Sync Agent Flow

## Overview

The sync agent is the real-time half of the system. It handles one customer interaction at a time, triggered either by an Experience Cloud session or a Salesforce case creation event.

It does not make decisions independently. It researches, drafts, and routes to a human.

---

## Trigger Points

**Pre-ticket (Experience Cloud)**
Customer begins describing their issue. Agent is triggered before the case is created. Goal is deflection — resolve the issue without a ticket.

**Post-ticket (Salesforce case creation)**
A case has been created. Agent is triggered via Salesforce Flow HTTP callout. Goal is to research and draft before the rep opens the case.

---

## Flow

### Step 1 — Deflection Attempt

Triggered from Experience Cloud. Customer has described their issue.

The agent:
1. Classifies the issue type — is this a how-to question, a documentation question, an error/bug report, or something else?
2. Searches public GitLab docs and available playbooks
3. Checks the support guide for matching scenarios
4. If a likely answer is found — presents it to the customer and asks if it resolves the issue

If resolved: session ends, no ticket created.
If not resolved: move to Step 2.

**What Jira is NOT used for here.** Jira belongs in the investigation step. At the deflection stage we're trying to answer "can the customer help themselves?" not "is this a known bug?"

---

### Step 2 — Guided Intake

Deflection didn't resolve it. Before the customer submits the case, the agent collects structured information.

The support guide feeds what to ask based on issue category. For example:
- Authentication issues → ask for IdP type, when it started, whether it affects all users
- Performance issues → ask for environment, approximate timing, whether it's consistent or intermittent
- Error messages → ask for the exact error text and steps to reproduce

The agent does not ask for everything. It asks for what matters for this category of issue.

Output: a case submission with structured fields populated and relevant logs/attachments requested.

---

### Step 3 — Customer Context Lookup

Case has been created. Agent pulls customer context from Salesforce via the Apex REST endpoint.

Fields retrieved:
- Market segment
- ARR
- Renewal date / status
- Account tier
- Any existing open cases for this account

This context is passed to the LLM along with the case content. It informs tone and priority — a strategic renewal customer with a critical issue gets treated differently than a low-ARR account with a how-to question.

---

### Step 4 — Investigation

Agent searches the knowledge index with the case description and structured intake data.

Search order:
1. Existing Salesforce cases — has this account or a similar account seen this before?
2. Zendesk archive — what did we do last time something like this came in?
3. Confluence KB — is there an article that addresses this?
4. Jira — is this a known bug or a reported issue? If so, what's the status and is there a workaround?

The agent retrieves the top relevant results from each source and passes them to the LLM with the case context.

---

### Step 5 — Draft Generation

The LLM receives:
- Case description and structured intake fields
- Customer context (segment, ARR, renewal status)
- Top search results from the knowledge index
- Instructions for tone and format

It produces a draft response. The draft includes:
- A direct answer or next step
- Reference to the source it drew from (so the rep can verify)
- A confidence indicator — high, medium, or low — based on how well the search results matched the issue

Low confidence drafts are flagged clearly. The rep should treat them as a starting point, not a recommendation.

---

### Step 6 — Human Review

The draft is sent to the rep via Slack notification or email — TBD based on team preference.

The notification includes:
- Case summary
- Draft response
- Sources used
- Confidence level
- Link to the case in Salesforce

The rep reviews, edits if needed, and approves. The approved response is sent to the customer through the normal Salesforce channel.

**Nothing is sent to the customer without this step.**

---

## What the Agent Does Not Handle

- Escalation decisions — the rep makes those
- Billing or contract questions — out of scope
- Issues requiring access to customer environments
- Anything requiring a phone or video call
