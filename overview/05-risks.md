# Risks

A short list of the things that could kill or slow this project, with what we'd do about each. Not a comprehensive risk register and not a contingency plan. This is a one-page snapshot for the people who will ask.

---

## Content Quality

**The risk.** Garbage in, garbage out. If the historical data we're indexing is too thin, too noisy, or too internally contradictory, the agent's drafts will be plausible-sounding nonsense. Reps will lose trust quickly, and once they're checking everything by hand the system has a negative ROI.

**Why it matters.** The Zendesk archive is the single most important data source and we don't yet know how clean the resolution context is in those tickets. Salesforce cases post-migration are likely thin. Confluence is already known to be low-confidence.

**What we do about it.** Phase 1 has manual validation built into the exit criteria (20 representative test queries returning a relevant top-3 result at 80%+). If we can't hit that bar, we don't ship Phase 2. The deflection step in the agent is also designed to draw only from sources we trust (GitLab docs, support guide), with the noisier sources reserved for the investigation step where the rep is the safety net.

---

## LLM Cost

**The risk.** Token spend balloons past the budget once we're processing real volume. Most likely failure mode is the initial extraction pass being more expensive than estimated, or the delta runs growing faster than expected as new sources come online.

**Why it matters.** The current cost estimates are back-of-envelope and assume volume numbers we haven't confirmed. A 5x miss on either side of the estimate would change the conversation.

**What we do about it.** Confirm record counts against actual sources before authorizing the initial pass. Run the first pass against a 5% sample to validate the extraction prompt and cost-per-record before committing to the full archive. Budget alerting at the 50% and 80% marks of the monthly cap, with a hard pause at 100% so we never get a surprise invoice.

---

## Customer Trust

**The risk.** A draft response goes out with a wrong answer or, worse, exposes information from another customer's ticket. Either of those would be a recoverable mistake operationally and an unrecoverable one with the customer.

**Why it matters.** The whole project is premised on the support team being faster, not less careful. Any incident that suggests we got faster at being wrong is a serious setback.

**What we do about it.** The human review gate. Every response is approved by a rep before it leaves. This is a permanent design constraint, not a phase-one workaround. The data handling design also strips customer identifiers from the chunks that feed the LLM where possible, and the audit log gives us a path to investigate any incident after the fact.

---

## Data Residency and Compliance

**The risk.** Customer data goes through the Anthropic API or Voyage AI without the contractual coverage we need, and a customer or auditor catches it.

**Why it matters.** Some of our customers may have contractual data residency or sub-processor restrictions that haven't been mapped against our LLM and embedding vendors yet.

**What we do about it.** Confirm the DPA, retention posture, and regional endpoints before any production data is sent. If a specific customer cohort can't be sent through the Anthropic API at all, the trigger excludes their cases from the agent flow until the contract changes. Detail in [Data Handling](../design/06-data-handling.md).

---

## Latency in the Sync Agent

**The risk.** End-to-end latency from case creation to draft delivery exceeds the 2-minute threshold often enough that reps stop relying on the draft and start working cases manually before it arrives. If that becomes the default behavior, the agent has no value.

**Why it matters.** The dominant time cost is the LLM call, which is bounded by the Anthropic API and not something we control directly.

**What we do about it.** The non-functional requirements set the latency budget explicitly so we have something to measure against. If a Phase 2 pilot shows latency consistently above 2 minutes, we evaluate caching common queries, switching to a faster model for first-pass drafting, or pre-computing drafts on case creation rather than on rep open. None of these are decisions to make now, but the path to address it exists.

---

## Migration Disruption

**The risk.** A Salesforce-to-HubSpot migration kicks off mid-build and consumes ops bandwidth, derailing Phase 1 or 2.

**Why it matters.** This is the reason for the external orchestration decision, but it doesn't immunize us from the disruption itself.

**What we do about it.** The architecture is intentionally decoupled. If migration starts, the knowledge pipeline keeps running unaffected. Phase 2's Salesforce trigger gets rewritten to a HubSpot trigger, and the Salesforce data connector becomes a HubSpot data connector. The rest of the system is untouched. This is a slowdown risk, not a kill risk.

---

## Support Team Adoption

**The risk.** The team uses the agent for a few weeks and then quietly drifts back to working cases manually because the drafts aren't quite useful enough or the workflow isn't quite smooth enough.

**Why it matters.** A Phase 2 launch with no behavioral change is the same as not launching.

**What we do about it.** Tight feedback loops in Phase 2. Reps flag drafts that are wrong, missing context, or worse than starting from scratch, and those flags drive iteration on the extraction prompt and search retrieval. The human review queue location decision also lives or dies on this point: if the review interface adds friction, adoption suffers regardless of draft quality.
