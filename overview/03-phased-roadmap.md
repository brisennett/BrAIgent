# Phased Roadmap

The goal is to deliver something real at each phase, not just build toward a future state that may never arrive.

---

## Phase 1 — Investigative Aid and Knowledge Foundation

Phase 1 has two workstreams running in parallel. Together they produce the first version of a real, support-team-facing tool.

### Workstream A — Investigative Aid (support-team-facing output)

When a case is created in Salesforce, the agent ingests the customer-submitted data, correlates it with server-side signals from Datadog, and produces a structured findings summary plus suggested next investigative actions. Output goes to the support engineer working the case, with a path to surface the same findings to engineering on escalation.

**Scope:**
- Salesforce Flow triggers the external agent on case creation, using the same Apex REST pattern as the support portal onboarding flow
- Agent ingests case data: description, attachments, parsed log content, customer identifiers
- Agent queries Datadog via MCP for time-correlated, customer-correlated server-side context
- Agent runs LLM analysis across both data sources
- Findings are delivered to the support engineer (via Slack or case comment, decision pending in Open Questions)
- Findings include a one-line summary, the supporting evidence, and a list of suggested next investigative actions

**Why this delivers value first:** Every customer case produces a visible output the day this ships. No reliance on knowledge-index quality, no customer-facing surface, no human-in-the-loop bottleneck. The support engineer gets a faster path to "what's actually happening here?" on the cases they're already opening.

### Workstream B — Knowledge Pipeline

Same scope as the original Phase 1 plan. Index Zendesk archive, GitLab docs, and Jira so the investigative aid can answer "have we seen this before?" alongside the live signal.

**Scope:**
- Stand up the data collector and orchestration engine
- Connect to Zendesk archive, GitLab docs, and Jira (in that order)
- Process raw data into structured, searchable knowledge
- Validate that the index returns useful results when queried manually

**Build order matters.** Zendesk first because it's the highest-volume historical source and the extraction prompt will need the most iteration there. GitLab next because it's the cleanest data and acts as a sanity check that the pipeline works on well-structured input. Jira last because the value is narrower (known-bug lookups) and we want the index already populated when we wire it in.

**Why this runs in parallel, not later:** The investigative aid in Workstream A is materially better when it can pull in similar past cases from the index. Without history, the agent only sees the current case and current Datadog signal. With history, it can flag "we saw this exact pattern in three cases last quarter, and the resolution was X." That second answer is the one that saves the support engineer real time.

### Phase 1 is done when:

The two workstreams have a combined gate. All of these have to hold for two consecutive weeks before Phase 1 ships.

1. The Salesforce trigger fires reliably on case creation and the agent receives the case payload without data loss.
2. The agent successfully parses log attachments for the top three log formats we see in real cases.
3. The Datadog MCP integration returns server-side context for at least 80% of cases where Datadog has signal during the relevant window.
4. The knowledge pipeline runs end-to-end on a schedule and recovers from a single-source failure without manual intervention.
5. The index contains processed records from all three sources at expected volume (within 10% of source counts).
6. A set of 20 representative test queries, drawn from real historical cases, returns a relevant top-3 result at least 80% of the time, judged manually by the support team.
7. End-to-end findings (summary plus suggested next actions) are produced within the latency budget defined in the [non-functional requirements](../design/07-non-functional-requirements.md).
8. Findings are observable and auditable: someone can answer "what did the agent see and what did it suggest" for any case after the fact.
9. Support engineers using the findings rate them "useful" or better on at least 70% of cases in a two-week pilot, judged by an in-product feedback prompt.

**What we'll know at the end:** Whether case data plus live server-side context plus historical knowledge actually produces faster and better investigations, and where the gaps are between where we are and a finding the support engineer would call "useful" on more than 70% of cases.

---

## Phase 2 — Customer Response Drafting

Once the investigative aid is producing reliable findings, layer customer-facing draft responses on top.

**Scope:**
- The same agent that produced findings in Phase 1 also drafts a customer-ready response
- Draft is routed to the support engineer for review via the same channel as findings
- Support engineer approves and edits before anything is sent
- Customer context (segment, ARR, renewal status) shapes tone and priority of the draft

**Why this comes after Phase 1:** Drafting a customer response without first being good at understanding the case is the wrong order. Phase 1 builds the understanding. Phase 2 adds the response on top.

**What we'll know at the end:** Whether the draft quality is good enough to save the support engineer meaningful time on the writing pass, on top of the investigation time saved in Phase 1.

---

## Phase 3 — Guided Intake

Structured intake that populates case fields automatically and reduces the first-exchange back-and-forth.

**Scope:**
- Conversational prompts collect version, error messages, logs, environment details before case submission
- Case fields auto-populated on creation
- Support guide feeds what to ask based on issue category

**What we'll know at the end:** Whether intake quality has improved enough to reduce the first-response back-and-forth that currently adds days to resolution time.

---

## Phase 4 — Customer-Facing Surface (Form TBD)

The customer-facing surface is the last thing we build, both because it has the highest blast radius if anything goes wrong and because the right form for it is still an open question. Two distinct possibilities are on the table.

**Option A — Free self-service deflection.** Customer describes their issue in Experience Cloud or a help widget before submitting a ticket. Agent checks public docs, playbooks, and the support guide. If it looks resolvable, presents an answer and asks if it helped. If not, hands off to guided intake. Closer to traditional support deflection. Lowest implementation cost and the natural extension of what Phases 1 through 3 already produce.

**Option B — Paid self-healing service.** A productized version of the investigative aid, exposed to customer admins as a feature in their admin console. Customer enters a problem (error message, attached logs, what they were trying to do), the agent uses the same engine that's serving the internal support team to investigate in real time, and surfaces a resolution path or workaround. Sold as an upcharge or packaged as a premium tier. Higher implementation cost, higher revenue potential, and a different conversation with product and legal than Option A.

These aren't mutually exclusive. Option A could ship first as a free baseline, with Option B layered as a premium tier on top. Or we go straight to Option B if the product and business case lines up. The form decision is a product and business call, not a support team call, and lives outside the scope of this doc.

**Why this is last regardless of form:** Customer-facing surfaces require the agent to be reliable enough that a wrong answer in front of a customer is rare. We get there through Phases 1 through 3, where the same engine runs internally with support engineers as the safety net. By the time it's ready for a customer surface, we have the operational data to know whether it works.

**What we'll know at the end:** Depends on which form ships. For Option A: deflection rate and what categories of issues are deflectable vs. not. For Option B: whether customers actually use the service when offered, what the resolution rate is unattended, and what it tells us about packaging and pricing.

---

## Beyond Phase 4 — Strategic Note

The investigative aid we're building in Phase 1 is essentially a smart agent that fixes customer problems by correlating their evidence with our knowledge and live infrastructure data. Today that agent serves an internal support engineer. Nothing in the architecture prevents the same engine, suitably packaged, from serving a customer admin directly.

If the org decides to productize this as "self-healing support" or similar (included as a premium tier, sold as an add-on, or bundled into an enterprise plan), the technical lift past Phase 1 is meaningful but not foundational. Most of the work is product packaging: the customer-facing UI, billing integration, customer-specific data scoping so customer A never sees customer B's resolutions, confidence thresholds tuned for customer-visible output, and the contractual changes that come with sending a customer's own logs through our LLM provider on their behalf.

Worth flagging now so the architecture decisions in Phase 1 don't paint us into a corner. None of the current decisions do, but the question is worth keeping in mind when we evaluate trade-offs.

---

## What's Not on the Roadmap (Yet)

- Automated customer responses without human review (permanent, not a phase)
- Integration with GitLab source code as a knowledge source
- Confluence as a primary knowledge source (dependent on content quality)
- Agentforce or Einstein. Architecture stays external for now ([ADR-001](../decisions/adr-001-external-orchestration.md))
