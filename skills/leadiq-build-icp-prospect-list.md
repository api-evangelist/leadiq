---
name: leadiq-build-icp-prospect-list
description: Build a prospect list from an ideal-customer-profile definition — run an advanced search, page it safely, enrich the matches, and save them into a named LeadIQ Prospector list.
api: graphql/leadiq.graphql, openapi/leadiq-prospector-api-openapi.yml
surface: https://api.leadiq.com/graphql, https://prospector.leadiq.com
operations: [flatAdvancedSearch, groupedAdvancedSearch, searchPeople, 'POST /v1/lists', 'POST /v1/lists/{listId}/prospects', 'POST /v1/lists/{listId}/prospects/batch', 'GET /v1/lists/{listId}/prospects']
generated: '2026-08-13'
method: generated
source: https://developer.leadiq.com/, github.com/leadiq/api-samples
---

# Build an ICP prospect list

This mirrors LeadIQ's own six-step sample pipeline (`full_pipeline` in github.com/leadiq/api-samples). It crosses two surfaces with two different credentials — read the auth note carefully.

## Two surfaces, one key, two encodings

| Surface | Base | Header |
|---|---|---|
| GraphQL Data API | `https://api.leadiq.com/graphql` | `Authorization: Basic <SECRET_BASE64_KEY>` (verbatim) |
| Prospector REST API | `https://prospector.leadiq.com` | `X-API-Key: <BASE64-DECODED KEY>` |

LeadIQ's own bash sample does the conversion with `printf '%s' "$LEADIQ_API_KEY" | base64 -d`. Sending the wrong form to either surface returns **401**.

> Note: LeadIQ's help-centre guide lists the Prospector paths without the `/v1` prefix (e.g. `POST /lists`) and mentions a `GET /lists/{listId}/export`. The published OpenAPI at `https://prospector.leadiq.com/openapi.json` and LeadIQ's own sample scripts both use `/v1/...` and there is no export endpoint — page `GET /v1/lists/{listId}/prospects` instead. Trust the spec and the samples.

## 1. Search for the ICP

Use `flatAdvancedSearch` for a flat list of people, `groupedAdvancedSearch` when you want them grouped by company (better for ABM and multi-threading).

```graphql
query Flat($input: FlatSearchInput!) {
  flatAdvancedSearch(input: $input) {
    totalPeople
    people { id name title }
    after { key value }
  }
}
```

Filters live in `ContactFilter` and `CompanyFilter`: job title, seniority, role/function, industry, company size, revenue, funding, location, technologies, NAICS/SIC, and `JobChangeFilter` for people who recently changed jobs or were promoted.

Select **only ids and profile fields here** — 0.1 UC per result. Do not pull emails or phones inside a search; enrich the shortlist afterwards.

## 2. Page it safely

Two pagination styles, and the choice matters:

- **Offset** (`skip` + `limit`) — jump to any page, but `skip + limit` **may not exceed 10,000** or the request is rejected, and it gets slower the deeper you go. Use it only for browsing the first pages.
- **Cursor** (`after`) — constant time at any depth, the only way to read past 10,000, and the only method that guarantees you see every record exactly once. Use it for every complete read.

Cursor loop: request page one with `limit` only (no `skip`, no `after`) and select `after` alongside your results; pass that `after` back **unchanged** with every filter and sort identical; repeat until the response returns no results or `after` comes back `null`.

Cursors are opaque, never expire and hold no server-side state, so a paging job can be paused and resumed. They are **not snapshots** — results can shift slightly if the underlying data changes mid-page.

Request `totalPeople` / `totalCompanies` on the **first page only**; it is expensive and, for large result sets, approximate.

## 3. Enrich the shortlist

Follow `skills/leadiq-enrich-known-contacts.md` for the people you actually intend to contact. Enrich after filtering, never before.

## 4. Create the list

```
POST https://prospector.leadiq.com/v1/lists
X-API-Key: <decoded key>
Content-Type: application/json

{ "name": "Sales Leaders in New Hampshire", "description": "Q3 outbound" }
```

`201` returns the `List` with a 24-char hex `id`. **`409` means a list with that name already exists** — read it back with `GET /v1/lists` rather than retrying with the same name.

## 5. Add prospects

- One at a time: `POST /v1/lists/{listId}/prospects` — `201` on success, `422` on an invalid body, `502` on a temporary upstream error.
- In bulk: `POST /v1/lists/{listId}/prospects/batch` — up to **100 items**; over that is a `400`. The `200` body carries `succeeded[]` and `failed[]`, where `failed[i].index` points at the position in your request array. Successes are in input order; failures are sorted by index. **A 200 here does not mean everything landed — always read `failed[]`.**

**Neither endpoint is idempotent.** LeadIQ states plainly: each call creates a distinct prospect even when the body is identical, with no deduplication by email, LinkedIn URL, or anything else. If a call times out or returns 5xx, do **not** blind-retry — track a stable client-side request id and only retry when the previous attempt did not return a 2xx, or reconcile with `GET /v1/prospects` first.

To attach a prospect that already exists to another list, use `POST /v1/lists/{listId}/prospects/{prospectId}` — this one **is** idempotent, returns 200 both times, and will not duplicate the list id in `listIds`.

## 6. Read the list back

`GET /v1/lists/{listId}/prospects?limit=100` then follow `nextCursor` until it is absent. `limit` accepts 1–100 and defaults to 25.

## Throughout

- 60 requests/minute; `429` on exhaustion with no `Retry-After` header — back off on your own schedule.
- List management and saved-prospect access are **free**; only search and enrichment consume credits.
- Report total UC spent by comparing the free `account` query before and after.
