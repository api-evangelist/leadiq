# LeadIQ (leadiq)

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

LeadIQ is a B2B sales intelligence and contact data platform headquartered in Santa Clara, California (with engineering in Singapore and Brisbane) that helps outbound sales teams identify, capture, enrich, and engage prospects across Salesforce, HubSpot, Outreach, Salesloft, Gong, and other revenue tools. The product portfolio includes Prospector (Chrome extension for contact capture with verified work emails and direct-dial mobile phones), Scribe (AI message writer), Refresh / CRM Enrichment (continuous contact and account hygiene for Salesforce and HubSpot), Champion Tracking (job-change alerts on existing contacts), AI Account Prospecting (ICP-fit account and persona discovery), and Lando — an agentic AI assistant that fuses first-party CRM data with LeadIQ's third-party intelligence behind a conversational interface. LeadIQ exposes a public GraphQL Data API at https://api.leadiq.com/graphql for programmatic people search, company search, advanced grouped search, prospect-list management, usage reporting, and data-feedback submission, authenticated via HTTP Basic auth with an API key issued in the LeadIQ dashboard. The same API surface is also reachable through a Model Context Protocol (MCP) server so AI agents (including Claude) can query verified contact and account data conversationally. Rate limits default to 10 requests/minute on Free and 60 requests/minute on paid plans, with credit-based metering where an email costs 1 credit, a phone 10 credits, and account enrichment 3 credits.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/leadiq/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/leadiq/refs/heads/main/apis.yml)

## Scope

- **Type:** Index
- **Position:** Provider
- **Access:** 3rd-Party

## Tags

- Sales Intelligence
- B2B Data
- Contact Data
- Lead Generation
- Prospecting
- CRM Enrichment
- Sales Engagement
- GraphQL
- Model Context Protocol
- Revenue Operations
- Go To Market

## Timestamps

- **Created:** 2026-05-25
- **Modified:** 2026-05-25

## APIs

### LeadIQ Data API

Public GraphQL Data API exposing LeadIQ's verified B2B contact and company intelligence. Supports people search by name + company / domain / LinkedIn URL / email, company search by name or domain, grouped and flat advanced search with ContactFilter and CompanyFilter inputs (industries, titles, seniorities, technologies, geography), prospect list management (createList, addProspectToList), account / usage / plan inspection, the workatoToken query for low-code integration, and submitPersonFeedback / markAsInvalid mutations for closed-loop data correction. Premium data fields (mobile phone) and ProfileFilter inputs (HasWorkEmail, HasVerifiedWorkEmail, HasWorkPhone, HasVerifiedWorkPhone, HasPersonalEmail) gate higher-cost lookups. Authentication is HTTP Basic with the API key as the username and an empty password.

- **Human URL:** [https://developer.leadiq.com/](https://developer.leadiq.com/)
- **Base URL:** `https://api.leadiq.com/graphql`

#### Tags

- Sales Intelligence
- Contact Data
- GraphQL
- Prospecting

#### Properties

- [Documentation](https://developer.leadiq.com/)
- [Documentation](https://leadiqhelp.zendesk.com/hc/en-us/articles/35509840045083-LeadIQ-Public-API-Overview)
- [Documentation](https://leadiqhelp.zendesk.com/hc/en-us/articles/29375289152795-LeadIQ-Public-API-Guide)
- [OpenAPI](openapi/leadiq-data-api-openapi.yml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Postman Collection](collections/leadiq-data-api.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/leadiq-data-api.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)
- [A P I Blueprint](https://github.com/leadiq/dataiq-api-specs/blob/master/apiary.apib)
- [Repository](https://github.com/leadiq/dataiq-api-specs)
- [Samples](https://github.com/leadiq/api-samples)
- [Graph Q L](https://api.leadiq.com/graphql)
- [Authentication](https://leadiqhelp.zendesk.com/hc/en-us/sections/29405194605723-API)
- [Rate Limits](https://leadiqhelp.zendesk.com/hc/en-us/articles/35509840045083-LeadIQ-Public-API-Overview)
- [Pricing](https://leadiq.com/pricing)
- [Sign Up](https://leadiq.com/sign-up)
- [Support](https://leadiqhelp.zendesk.com/hc/en-us)
- [Contact](mailto:api@leadiq.com)

### LeadIQ MCP Server

Model Context Protocol server that exposes LeadIQ's verified B2B contact and company intelligence to AI agents (Claude Desktop, Claude Code, Cursor, and any MCP-compatible client). Provides conversational access to the same searchPeople, searchCompany, advanced search, prospect-list, and account-usage capabilities offered by the Data API, with credentialing handled via the LeadIQ API key.

- **Human URL:** [https://leadiq.com/blog/claude-mcp/](https://leadiq.com/blog/claude-mcp/)

#### Tags

- Model Context Protocol
- AI Agents
- Sales Intelligence

#### Properties

- [Documentation](https://leadiq.com/blog/claude-mcp/)
- [Authentication](https://leadiqhelp.zendesk.com/hc/en-us/sections/29405194605723-API)
- [Postman Collection](collections/leadiq-data-api.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/leadiq-data-api.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

## Common Properties

- [Website](https://leadiq.com)
- [Developer Portal](https://developer.leadiq.com/)
- [Documentation](https://leadiqhelp.zendesk.com/hc/en-us/sections/29405194605723-API)
- [Pricing](https://leadiq.com/pricing)
- [Sign Up](https://leadiq.com/sign-up)
- [Login](https://account.leadiq.com/login)
- [Blog](https://leadiq.com/blog)
- [Podcast](https://leadiq.com/podcast)
- [Case Studies](https://leadiq.com/case-studies)
- [Support](https://leadiqhelp.zendesk.com/hc/en-us)
- [Status](https://status.leadiq.com)
- [Security](https://leadiq.com/security)
- [Privacy](https://leadiq.com/privacy)
- [Terms](https://leadiq.com/terms)
- [Careers](https://leadiq.com/careers)
- [About](https://leadiq.com/about)
- [Contact](https://leadiq.com/contact)
- [Integrations](https://leadiq.com/integrations)
- [Repository](https://github.com/leadiq)
- [A P I Repository](https://github.com/leadiq/dataiq-api-specs)
- [A P I Samples](https://github.com/leadiq/api-samples)
- [LinkedIn](https://www.linkedin.com/company/leadiq)
- [Twitter](https://twitter.com/leadiq)
- [YouTube](https://www.youtube.com/@LeadIQ)
- [Crunchbase](https://www.crunchbase.com/organization/leadiq)
- [G2](https://www.g2.com/products/leadiq/reviews)

## Maintainers

**FN:** Kin Lane
**Email:** kin@apievangelist.com
