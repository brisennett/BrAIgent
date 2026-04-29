# Problem and Vision

## The Problem

Support today is reactive. A customer hits a wall, submits a ticket, and waits. Someone on the team manually reads the case, parses the attached logs, hunts through Datadog and past cases for context, and writes a response. Often from scratch even when we've seen the same problem before.

This works at a certain scale. It stops working when the customer base grows faster than the team.

Five specific gaps:

**Manual investigation.** Every case starts with the same drill: read the description, parse the logs, check Datadog, search past cases, build a mental picture. The first 15 to 30 minutes of every case is manual correlation, regardless of support engineer tenure.

**Inconsistent research.** Whether a support engineer finds the right past case depends on memory and how long they've been on the team. That knowledge doesn't transfer between people.

**Drafting from scratch.** Every customer response gets written from a blank page, even for issues we've resolved ten times before.

**Unstructured intake.** "Describe your issue" produces paragraphs of context-free text. The first exchange is figuring out what the customer is actually asking.

**No self-service path.** Customers who could solve their own problem have nowhere to go. The only option is a ticket.

---

## The Vision

An AI layer that sits between the customer and the support team. The agent handles the rote parts of the workflow: parsing, correlating, looking things up. The team handles the parts that need judgment.

We build inside the team first, then move outward toward the customer.

**Investigation first.** When a case is created, the agent ingests the customer's description and attached logs, correlates them with server-side signal from Datadog, looks up similar past cases in our indexed history, and presents structured findings to the support engineer before the support engineer has opened the case. The support engineer walks in already pointed at the likely cause.

**Drafting next.** Once findings are reliable, the agent drafts the customer-facing response too. Same workflow, same human-review gate, just with the writing pass done in advance.

**Intake after that.** Structured prompts gather the right information up front, reducing the back-and-forth that adds days to resolution.

**Customer-facing service last.** Eventually the same engine that helps the support engineer can help the customer directly. The form is still open. It could be free self-service deflection (docs and past resolutions surfaced when the customer describes their issue) or, longer term, a paid product that lets customers troubleshoot in real time from their admin console. That's a product and business decision rather than a support team decision, and it's flagged in the [roadmap](03-phased-roadmap.md) for when we get there.

The support engineer decides what gets sent in Phases 1 through 3. Always. Customer-facing surfaces ship last because that's where a mistake costs the most.

---

## What Success Looks Like

The outcomes we're building toward, in the order we ship them:

- Support engineers open a case already knowing what's likely happening, instead of building the picture from scratch every time
- Customer responses go out faster because the draft is already written and the evidence is already cited
- Intake collects complete information up front, so the first exchange isn't asking the customer for context
- Some cases never get opened because the customer found the answer themselves

Resolution time drops without adding headcount. That's the outcome that ties them all together.
