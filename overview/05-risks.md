# Risks

A short list of the things that could kill or slow this project, with what we'd do about each. Not a comprehensive risk register and not a contingency plan. This is a one-page snapshot for the people who will ask.

---

## Content Quality

**The risk.** Garbage in, garbage out. If the historical data we're indexing is too thin, too noisy, or too internally contradictory, the agent's drafts will be plausible-sounding nonsense. Support engineers will lose trust quickly, and once they're checking everything by hand the system has a negative ROI.

**Why it matters.** The Zendesk archive is the single most important data source and we don't yet know how clean the resolution context is in those tickets. Salesforce cases post-migration are likely thin. Confluence is already known to be low-confidence.

**What we do about it.** Phase 1 has manual validation built into the exit criteria (20 representative test queries returning a relevant top-3 result at 80%+). If we can't hit that bar, we don't ship Phase 2. The deflection step in the agent is also designed to draw only from sources we trust (GitLab docs, support guide), with the noisier sources reserved for the investigation step where the support engineer is the safety net.

---

## LLM Cost

**The risk.** Token spend balloons past the budget once we're processing real volume. Most likely failure mode is the initial extraction pass being more expensive than estimated, or the delta runs growing faster than expected as new sources come online.

**Why it matters.** The current cost estimates are back-of-envelope and assume volume numbers we haven't confirmed. A 5x miss on either side of the estimate would change the conversation.

**What we do about it.** Confirm record counts against actual sources before authorizing the initial pass. Run the first pass against a 5% sample to validate the extraction prompt and cost-per-record before committing to the full archive. Budget alerting at the 50% and 80% marks of the monthly cap, with a hard pause at 100% so we never get a surprise invoice.

---

## Customer Trust

**The risk.** A draft response goes out with a wrong answer or, worse, exposes information from another customer's ticket. Either of those would be a recoverable mistake operationally and an unrecoverable one with the customer.

**Why it matters.** The whole project is premised on the support team being faster, not less careful. Any incident that suggests we got faster at being wrong is a serious setback.

**What we do about it.** The human review gate. Every response is approved by a support engineer before it leaves. This is a permanent design constraint, not a phase-one workaround. The data handling design also strips customer identifiers from the chunks that feed the LLM where possible, and the audit log gives us a path to investigate any incident after the fact.

---

## Data Residency and Compliance

**The risk.** Customer data goes through the Anthropic API or Voyage AI without the contractual coverage we need, and a customer or auditor catches it.

**Why it matters.** Some of our customers may have contractual data residency or sub-processor restrictions that haven't been mapped against our LLM and embedding vendors yet.

**What we do about it.** Confirm the DPA, retention posture, and regional endpoints before any production data is sent. If a specific customer cohort can't be sent through the Anthropic API at all, the trigger excludes their cases from the agent flow until the contract changes. Detail in [Data Handling](../design/06-data-handling.md).

---

## Latency in the Sync Agent

**The risk.** End-to-end latency from case creation to draft delivery exceeds the 2-minute threshold often enough that support engineers stop relying on the draft and start working cases manually before it arrives. If that becomes the default behavior, the agent has no value.

**Why it matters.** The dominant time cost is the LLM call, which is bounded by the Anthropic API and not something we control directly.

**What we do about it.** The non-functional requirements set the latency budget explicitly so we have something to measure against. If a Phase 2 pilot shows latency consistently above 2 minutes, we evaluate caching common queries, switching to a faster model for first-pass drafting, or pre-computing drafts at case creation rather than waiting until the support engineer opens the case. None of these are decisions to make now, but the path to address it exists.

---

## Migration Disruption

**The risk.** A Salesforce-to-HubSpot migration kicks off mid-build and consumes ops bandwidth, derailing Phase 1 or 2.

**Why it matters.** This is the reason for the external orchestration decision, but it doesn't immunize us from the disruption itself.

**What we do about it.** The architecture is intentionally decoupled. If migration starts, the knowledge pipeline keeps running unaffected. Phase 2's Salesforce trigger gets rewritten to a HubSpot trigger, and the Salesforce data connector becomes a HubSpot data connector. The rest of the system is untouched. This is a slowdown risk, not a kill risk.

---

## Support Team Adoption

**The risk.** The team uses the agent for a few weeks and then quietly drifts back to working cases manually because the findings aren't quite useful enough or the delivery channel isn't quite where they need it.

**Why it matters.** A Phase 1 launch with no behavioral change is the same as not launching. The whole project is premised on support engineer workflow getting faster.

**What we do about it.** Tight feedback loops in Phase 1 itself. Support engineers flag findings that are wrong, missing context, or worse than starting from scratch, and those flags drive iteration on the synthesis prompt and retrieval logic. The findings delivery channel decision lives or dies on this point: if the interface adds friction, adoption suffers regardless of finding quality. The Phase 1 exit criteria require 70% "useful or better" feedback over a two-week pilot before the phase ships, so we won't be guessing.

---

## Datadog MCP Reliability

**The risk.** The Datadog MCP is the newest piece of plumbing in the system and we don't have one yet. If it's flaky, slow, or gets rate-limited, the investigative aid loses one of its strongest signals and findings degrade to "what the case data shows." If we built a direct API client instead of an MCP, more of the integration is on us to maintain.

**Why it matters.** Roughly half the value of the investigative aid comes from correlating customer evidence with server-side signal. If that correlation is unreliable, the agent is producing case-data summaries with extra steps.

**What we do about it.** The Datadog correlation step is designed to fail open. If the MCP times out or returns nothing, the synthesis step proceeds with whatever it has and notes the absence. The MCP runs in our infrastructure with our credentials, so reliability is something ops can actually fix. The Phase 1 exit criteria require an 80%+ success rate on Datadog queries during the pilot, so we'll know if it's not holding up before we ship.

---

## Customer-Attached Log Variability

**The risk.** Customers attach logs in formats the parser can't handle, or attach things that aren't logs at all (screenshots, exported state, stale snippets). The parser produces nothing useful, the LLM works from raw text, and the findings degrade.

**Why it matters.** Log parsing is the load-bearing step that turns customer evidence into structured input. If it breaks on a meaningful share of real cases, the rest of the flow suffers.

**What we do about it.** Phase 1 covers the top three log formats based on a sample of recent cases. New formats are added on demand as we see them. Anything the parser can't handle is passed through as raw text rather than blocking the flow. Image attachments are surfaced to the support engineer alongside the findings rather than processed in Phase 1. The exit criteria put a floor on parser coverage in real cases, not a ceiling.

---

## Cold-Start Finding Accuracy

**The risk.** During the period before the knowledge pipeline is producing useful retrievals, the investigative aid only sees the current case and current Datadog signal. Findings in that window may not be much better than what the support engineer would produce on their own, and support engineers form a first impression that's hard to walk back later.

**Why it matters.** First impressions of an internal tool drive long-term adoption. If the early version of the findings is "tell me what I already know," support engineers stop reading them.

**What we do about it.** The two Phase 1 workstreams run in parallel rather than sequentially specifically to shorten this cold-start window. We also pilot with a small set of support engineers who understand the system is new and whose feedback is going to drive the iteration. The exit criteria explicitly require the index to be returning useful retrievals (the 80% top-3 bar) and the findings to be rated useful at 70% or better, so we don't ship the phase until both are in a good place.
