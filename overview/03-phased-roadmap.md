# Phased Roadmap

The goal is to deliver something real at each phase — not just build toward a future state that may never arrive.

---

## Phase 1 — Build the Knowledge Foundation

Before the agent can do anything useful, the knowledge pipeline has to exist and produce reliable output.

**Scope:**
- Stand up the data collector and orchestration engine
- Connect to Zendesk archive, GitLab docs, and Jira as first sources
- Process raw data into structured, searchable knowledge
- Validate that the index returns useful results when queried manually

**Why this first:** The sync agent is only as good as what it can look up. Building the agent before the knowledge layer exists produces something that guesses instead of researches.

**What we'll know at the end:** Whether our existing data is good enough to support AI-assisted resolution, and where the gaps are.

---

## Phase 2 — Sync Agent, Post-Ticket

Wire up the real-time agent to the knowledge index. Trigger it when a case is created in Salesforce.

**Scope:**
- Salesforce Flow triggers the external agent on case creation
- Agent pulls customer context from Salesforce (segment, ARR, renewal status)
- Agent searches the knowledge index and drafts a response
- Draft is routed to the rep for review via Slack or email
- Rep approves before anything is sent

**What we'll know at the end:** Whether the draft quality is good enough to save the rep meaningful time, and what sources are contributing the most useful information.

---

## Phase 3 — Deflection Before the Ticket

Move the agent upstream into the Experience Cloud intake flow.

**Scope:**
- Customer describes their issue before submitting
- Agent checks public docs, playbooks, and support guide
- If it looks resolvable, presents an answer and asks if it helped
- If not, moves to guided intake before case creation

**What we'll know at the end:** What percentage of sessions don't become tickets, and what categories of issues are deflectable vs. not.

---

## Phase 4 — Guided Intake and Data Collection

Structured intake that populates case fields automatically.

**Scope:**
- Conversational prompts collect version, error messages, logs, environment details
- Case fields auto-populated on creation
- Support guide feeds what to ask based on issue category

**What we'll know at the end:** Whether intake quality has improved enough to reduce the first-response back-and-forth that currently adds days to resolution time.

---

## What's Not on the Roadmap (Yet)

- Automated responses without human review
- Integration with GitLab source code (Phase 2+)
- Confluence as a primary knowledge source (dependent on content quality)
- Agentforce or Einstein — keeping the architecture external for now
