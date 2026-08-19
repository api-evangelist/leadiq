---
name: leadiq-mcp-agent-operations
description: Operate LeadIQ as an AI agent over its remote MCP connector — connect over OAuth, pick the right one of the seventeen tools, and disclose credit cost before spending a customer's balance.
api: mcp/leadiq-mcp.yml
surface: https://mcp.leadiq.com/mcp
operations: [EnrichPeople, EnrichCompanies, FindPeople, FindCompanies, FindJobChanges, BrowseProspectLists, CreateProspectList, GetProspectList, AddProspectToList, AttachProspectToList, GetProspect, CreateProspect, SearchProspects, VerifyEmail, VerifyProspectEmail, ExportProspectToSalesforce, CheckCredits]
generated: '2026-08-13'
method: generated
source: https://developer.leadiq.com/, https://mcp.leadiq.com/.well-known/oauth-protected-resource
---

# Operate LeadIQ over MCP

## Connect

- **Server URL:** `https://mcp.leadiq.com/mcp`
- **Transport:** Streamable HTTP
- **Auth:** OAuth 2.0, authorization code with PKCE, against `https://leadiq-mcp-prod.us.auth0.com/`
- **Client registration:** dynamic (`/oidc/register`) — there is no client id or secret to configure
- **Scopes:** `leadiq:api`, `offline_access`

An unauthenticated `tools/list` returns `401` with `WWW-Authenticate: Bearer realm="leadiq", resource_metadata="https://mcp.leadiq.com/.well-known/oauth-protected-resource"`. Follow that metadata document to discover the authorization server rather than hard-coding it.

The token carries the signed-in user's own LeadIQ permissions and credit balance. Enterprise-SSO users whose Google domain is linked to LeadIQ should enter their email address directly instead of clicking "Sign in with Google", which errors.

**Authorization is coarse.** `leadiq:api` is a single scope covering all seventeen tools, including the Salesforce write. There is no read-only grant, so an agent given this connector can write to the customer's CRM by construction. Say so when you ask for consent.

## Pick the right tool

| Goal | Tool | Cost |
|---|---|---|
| I know who they are, I need their email/phone | `EnrichPeople` (batch ≤ 10) | 0.1–11 UC per person, +3 UC per company |
| I know the company, I need firmographics | `EnrichCompanies` (batch ≤ 10) | 3 UC per company |
| I need to *find* people matching an ICP | `FindPeople` | 0.1 UC per result, 3 UC with company data |
| I need matching accounts, with contact counts | `FindCompanies` | 3 UC per company, 0.1 UC for names only |
| Who just changed jobs or got promoted | `FindJobChanges` | 0.1 UC per result |
| Anything to do with saved lists/prospects | `BrowseProspectLists`, `CreateProspectList`, `GetProspectList`, `AddProspectToList`, `AttachProspectToList`, `GetProspect`, `CreateProspect`, `SearchProspects` | **Free** |
| Is this address deliverable | `VerifyEmail` / `VerifyProspectEmail` | 0.1 UC per check |
| Push a saved prospect into Salesforce | `ExportProspectToSalesforce` | Free, +3 UC if it must unlock company data |
| What is this costing | `CheckCredits` | **Free** |

The connector also ships a guided `icp` prompt that walks through defining an ideal customer profile, running the search, and saving the top matches into a list with the cost disclosed up front. Prefer it for open-ended prospecting requests.

## Spend discipline

Universal Credits are billed on data actually returned: 0.1 UC per profile record, 1 UC per verified work email, 10 UC per direct phone, 3 UC per company's firmographics, 0.1 UC per email check. Larger unlocks replace the profile fee rather than stacking, so email + phone is 11 UC not 11.1. Company data is charged per company, so a person with two current positions costs 6 UC of company data. Anything unlocked in the past year is free again.

Rules to follow:

1. Call `CheckCredits` **first** on any session that will spend. It is free and returns the live rates for this plan, which override every number written here.
2. State the estimated cost before a paid call, and get explicit consent before a large one. `FindCompanies` at 25 accounts is 75 UC; `EnrichPeople` at 10 people with phones is 110 UC.
3. Prospect for ids first (`FindPeople`, profile only), filter, then enrich only the survivors. Never enrich a raw search result set.
4. Free tools are genuinely free — never ration list management to save credits.

## Write discipline

`ExportProspectToSalesforce` writes into the customer's Salesforce, cannot be undone through this API, and is not idempotent. Always run duplicate detection first, show the operator the duplicates, and never retry a failed export automatically. See `skills/leadiq-save-and-export-prospects.md` for the full failure table.

## What MCP cannot do

The connector does not expose LeadIQ's champion/company-tracking family (`trackedContacts`, `trackedCompanies`, `companyTrackingTask`, `startCompanyTrackingTask`, `cancelCompanyTrackingTask`, `addTrackedCompanies`, `removeTrackedCompanies`, `removeAllTrackedCompanies`), the data-correction mutation `submitPersonFeedback`, or the 100-item bulk prospect create. If a task needs those, fall back to the GraphQL Data API or the Prospector REST API — see `mcp/leadiq-tool-crosswalk.yml` for the full surface divergence.

## Batch limits

Enrichment is batched at up to 10 contacts or companies per request. Documented session guidance elsewhere in LeadIQ's help centre mentions 100 per session — treat 10 per call as the hard limit and pace accordingly against the 60 requests/minute ceiling.
