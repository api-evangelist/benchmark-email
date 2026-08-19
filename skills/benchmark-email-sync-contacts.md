---
name: benchmark-email-sync-contacts
description: Keep contacts in a Benchmark Email account in sync with an external source of truth (CRM, store, POS), without creating duplicates and without violating unsubscribe state.
api: benchmark-email:benchmark-email-api
generated: '2026-08-13'
method: generated
source: openapi/benchmark-email-api-openapi.json + https://developers.benchmarkemail.io/scenarios/manage-contacts
scopes:
  - contacts:write
operations:
  - get_api_contact_structure
  - post_api_contact_search
  - post_api_contact
  - get_api_contact_by_contactId
  - put_api_contact_by_contactId
  - patch_api_contact_by_contactId
---

# Sync contacts into Benchmark Email

Base URL is per account: `https://api-{region}-{cluster}.benchmarkemail.io`, copied from
Settings > API Keys. Every call carries `X-API-Key: bme_...`. There is no sandbox — you are
writing to live contact data.

## 1. Read the contact structure first

`GET /api/contact-structure` (`get_api_contact_structure`)

You cannot write a contact without it. The response gives you the structure `_id` and each
field definition's `_id`. Contact writes reference fields by that id, never by label. Cache
it for the run.

If the source system has a field Benchmark does not, add it with
`PUT /api/contact-structure/{contactStructureId}` (`put_api_contact_structure_by_contactStructureId`)
before importing — send the full field list, including the existing entries with their ids.

## 2. Check for an existing contact before creating one

`POST /api/contact/search` (`post_api_contact_search`) with a `KEY` filter on the email
address, and a non-empty `source` array such as `["_id","key","fields","__v"]`. Omitting
`source` returns `400 ValidationError`.

This read-before-write is the only dedup mechanism available: Benchmark Email publishes no
`Idempotency-Key` header, so a retried create makes a second contact.

## 3. Create or update

- Not found: `POST /api/contact` (`post_api_contact`) with `key` (the email),
  `contactStructureId`, `fields[]` of `{_id, value}`, and any list assignments.
- Found, and only the status is changing: `PATCH /api/contact/{contactId}`
  (`patch_api_contact_by_contactId`). This accepts `status` only.
- Found, and field values are changing: `GET /api/contact/{contactId}`
  (`get_api_contact_by_contactId`) to read the current `__v`, then
  `PUT /api/contact/{contactId}` (`put_api_contact_by_contactId`) with the full body plus
  that `__v`. PUT replaces the record.

## 4. Respect unsubscribe state

`status.primary` is case-sensitive: `Active` or `Inactive`. Setting `Inactive` requires both
keys, e.g. `{"primary":"Inactive","secondary":"Unsubscribe"}`.

**An Inactive contact cannot be reactivated through the API.** This is deliberate compliance
behaviour, not a bug. If your source system marks someone as subscribed again, they must opt
in through a Benchmark form. Never treat this as an error to retry around.

## 5. Handle the failures you will actually hit

| Response | errorType | What to do |
|---|---|---|
| 400 | `ValidationError` | Fix the field named in `field` — usually a missing `contactStructureId` or an empty `source`. Do not retry unchanged. |
| 400 | `ConcurrencyError` | Your `__v` is stale. Re-GET, reapply, resend. |
| 403 | `ForbiddenError` | The message names the missing scope. Grant it on the key; retrying will not help. |
| 429 | `TooManyRequestsError` | Honour `Retry-After`, then back off `min(Retry-After * 2^attempt, 300)`. |

Watch `X-RateLimit-Remaining` (3,600/hour, shared across every key on the account) and
`X-Monthly-Remaining` on each response and pace at roughly one request per second.
