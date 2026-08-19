---
name: leadiq-enrich-known-contacts
description: Enrich people you already identify (LinkedIn URL, work email, or name + company) with verified work emails, direct phones, title, seniority and employer — while controlling exactly how many LeadIQ credits the call spends.
api: graphql/leadiq.graphql
surface: https://api.leadiq.com/graphql
operations: [searchPeople, searchPeoplePreview, account]
generated: '2026-08-13'
method: generated
source: https://developer.leadiq.com/, https://leadiqhelp.zendesk.com/hc/en-us/articles/29375289152795-LeadIQ-Public-API-Guide, github.com/leadiq/api-samples
---

# Enrich known contacts with LeadIQ

Use when you have an identifier for a person and need their verified contact data.

## Authenticate

POST every request to `https://api.leadiq.com/graphql` with:

- `Content-Type: application/json`
- `Authorization: Basic <LEADIQ_API_KEY>` — the "Secret Base64" key from Settings -> API, sent **verbatim**. It is already the base64 payload; do not encode it again.

## Check the balance first — it is free

Run the `account` query before any paid work. It consumes no credits and returns the plan, the period cap, and `available` / `used`.

```graphql
query Account {
  account {
    universalPlan { name product status nextBillingPeriod available used }
    dataHubPlan   { name product status nextBillingPeriod available used }
  }
}
```

If `available` will not cover the estimated spend below, stop and report the shortfall rather than issuing partial calls. Exhaustion returns HTTP **402**.

## Estimate the spend before you select fields

On this surface **field selection is the billing contract** — "a query only charges for the data points you actually select":

| Selected | Cost |
|---|---|
| profile (name, title, seniority, LinkedIn URL, employer name) | 0.1 UC per record |
| `emails` (verified work email) | 1 UC per person |
| `phones` (direct phone) | 10 UC per person |
| `companyInfo` firmographics | 3 UC per company |

Larger unlocks replace the profile fee rather than stacking — email + phone is 11 UC, not 11.1. Company firmographics are charged **per company**, so a person holding two current positions costs 6 UC of company data. Anything you unlocked in the past year is free to look up again.

**Never select `phones` unless the task explicitly needs a phone number.** It is a 100x multiplier over a profile lookup.

## Optional pre-flight

`searchPeoplePreview` is a lightweight check on whether a work email and/or phone can likely be produced for a person. Use it when you are about to spend on a large batch and want to skip records that will come back empty.

## Enrich

```graphql
query SearchPerson($input: SearchPeopleInput!) {
  searchPeople(input: $input) {
    totalResults
    hasMore
    results {
      id
      name { fullName first last }
      currentPositions {
        title
        seniority
        function
        companyInfo { id name domain }
        emails { value status type }
      }
      linkedin { linkedinUrl }
    }
  }
}
```

Input identifiers, most precise first:

| Field | Note |
|---|---|
| `linkedinUrl` | most precise identifier |
| `email` | known email address |
| `firstName` + `lastName` + `company.domain` and/or `company.name` | name + company combination |
| `id` | LeadIQ person id (`PersonID-<uuid>`) |
| `limit` / `skip` | pagination |

## Handle the response correctly

1. **A 200 is not success.** GraphQL errors arrive in the `errors` array with HTTP 200. Inspect `errors` on every response before reading `data`. This is the single most common integration bug on this API.
2. `401` — invalid or missing key. Check you sent the Secret Base64 form, not the decoded one (the decoded form belongs to the Prospector REST API).
3. `402` — insufficient credits.
4. `429` — rate limit. The published limit is **60 requests/minute**. There is **no `Retry-After` and no `RateLimit-*` header** on any LeadIQ response, so back off on a fixed schedule of your own (start at 1 s, double, cap at 60 s) rather than waiting for a signal that never arrives.
5. `500` — retry; contact api@leadiq.com if persistent.

Reads are safe to retry. See `skills/leadiq-save-and-export-prospects.md` before retrying anything that writes.

## Report the spend

Re-run `account` afterwards and report `used` before vs after, so the operator can see what the task actually cost.
