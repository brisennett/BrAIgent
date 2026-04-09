# Why We Can Do This

This isn't a greenfield project. Most of the hard integration work already exists in some form.

---

## Salesforce Integration Is Already Proven

The support portal onboarding flow uses a Salesforce Flow with an HTTP callout to an external API endpoint, backed by an Apex REST class that exposes user data. That's the same pattern the agent will use — triggering an external system and pulling customer context. No new Salesforce development paradigm required.

---

## The Historical Data Is Accessible

The Zendesk archive is already running in a Docker container and queryable via API. This was built specifically to preserve resolution history after the migration to Salesforce. It's one of the most valuable data assets for training the knowledge index and it's ready to use.

---

## Infrastructure Comfort

The team is comfortable with self-hosted Docker workloads. The orchestration engine (n8n) and the vector database both run in Docker. Ops can deploy and manage them alongside existing services without introducing new infrastructure paradigms.

---

## Domain Knowledge Is the Hard Part

Most AI projects fail because the people building the system don't understand the problem well enough. The support team knows what good looks like, what the common failure patterns are, what information a rep actually needs to resolve a case, and where the current process breaks down. That knowledge is what makes the difference between an agent that's useful and one that produces plausible-sounding nonsense.

---

## Portability Is Built In

The architecture is intentionally external to Salesforce. If the org moves to HubSpot or another platform, only the trigger and data-pull integrations change. The orchestration logic, knowledge index, and LLM layer are unaffected.
