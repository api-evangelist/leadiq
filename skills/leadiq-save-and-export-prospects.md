---
name: leadiq-save-and-export-prospects
description: Save prospects, verify their email deliverability, and export them into the caller's Salesforce as Leads or Contacts — with the duplicate-detection and non-idempotency guards this path requires.
api: openapi/leadiq-prospector-api-openapi.yml
surface: https://prospector.leadiq.com
operations: ['POST /v1/prospects', 'GET /v1/prospects', 'GET /v1/prospects/{prospectId}', 'GET /v1/verify-email', 'POST /v1/prospects/{prospectId}/verify-email', 'POST /v1/prospects/{prospectId}/export/salesforce', 'GET /v1/whoami']
generated: '2026-08-13'
method: generated
source: https://prospector.leadiq.com/openapi.json
---

# Save, verify and export LeadIQ prospects

**Read the Salesforce section before running anything in it.** That operation writes into a system of record you cannot roll back through this API.

## Authenticate

`X-API-Key: <base64-DECODED LeadIQ key>` against `https://prospector.leadiq.com`. A bearer JWT is also accepted (`Authorization: Bearer <jwt>`), but LeadIQ documents no way to obtain one outside the MCP OAuth flow.

Confirm who you are acting as before any write: `GET /v1/whoami` returns `{ user, team }`.

## Save a prospect

- `POST /v1/prospects` — creates a prospect with no list membership. `201` returns the `Prospect`.
- `POST /v1/lists/{listId}/prospects` — create and attach in one call. Prefer this when you know the destination list.

Both are **explicitly not idempotent**. Retrying after a network error or timeout creates a duplicate. Guard with a client-side request id and reconcile via `GET /v1/prospects` (search by `email`, or `firstName` + `lastName`) before any retry.

`413` means the body exceeded a server-side size cap whose value LeadIQ does not publish — split the payload rather than retrying it.

## Verify email deliverability

- Any address, no prospect required: `GET /v1/verify-email?email=<address>` — returns `{ status }`. 0.1 UC.
- A saved prospect, storing the result on the record: `POST /v1/prospects/{prospectId}/verify-email` — returns `{ status, prospect }`. 0.1 UC.

`409` on the second form means the prospect has no email to verify. `502` means EVS (the email verification service) is unreachable — this one **is** safe to retry, because verification does not create records.

Verify before a sequence send, not after. At 0.1 UC per check it is the cheapest operation LeadIQ sells.

## Export to Salesforce — the irreversible one

`POST /v1/prospects/{prospectId}/export/salesforce`

This runs the same pipeline as the LeadIQ web app, acting as the caller's connected Salesforce user, so their Salesforce permissions, validation rules and duplicate rules all still apply. Whether the record lands as a **Lead** or a **Contact** is decided by the member's "Save to" setting and org governance — it is deliberately not selectable through the API.

Before calling:

1. **Confirm with the operator.** LeadIQ's own MCP connector asks for confirmation before writing; do the same.
2. **Run it once with `force` unset.** A `200` with `status: duplicates_found` means nothing was written and `duplicates[]` lists the matching Salesforce records. Show them to the operator and let them decide.
3. Only set `force: true` on an explicit human instruction to write anyway.

Failure modes, all of which mean *stop*, not *retry*:

| Status | Meaning |
|---|---|
| `403` | `salesforce_export_disabled` (an admin turned it off for the org) or `salesforce_export_not_enabled` (still rolling out to this team) |
| `409` | `salesforce_not_connected`, `salesforce_save_as_unsupported`, `salesforce_account_required`, `quality_skipped`, `export_rejected`, `salesforce_rejected_record` |
| `404` | prospect not found or not visible to the caller |
| `413` | body too large |

**The one trap.** The spec labels `502` on this endpoint "temporary upstream error — safe to retry" *and* labels the endpoint "not idempotent — retrying may create a second record". Those two statements conflict, and the loser is the customer's CRM. **Do not auto-retry a 502 here.** Surface it, and let a human check Salesforce before anything is sent again. There is no delete path in this API.

May consume 3 UC when the prospect carries a company id the org has not already unlocked — the export unlocks company data to populate the Salesforce Account.

A `202` (rather than `200`) means the export was accepted asynchronously; keep the `requestId` from the body, which is the only correlation id LeadIQ returns anywhere.

## Read prospects back

- `GET /v1/prospects?limit=100&cursor=<nextCursor>` — search by `email`, or `firstName` + `lastName`. Mixing incompatible criteria returns `400`.
- `GET /v1/prospects/{prospectId}` — full record. `prospectId` must match `^[0-9a-fA-F]{24}$` or you get a `400`, not a `404`.

## Throughout

Every operation declares `429`. There is no `Retry-After` or `RateLimit-*` header on any response — back off on your own schedule. Errors use `{ code, message, details }`, not RFC 9457 problem details; branch on `code`, not on the message text.
