---
name: benchmark-email-create-campaign
description: Create and configure a Benchmark Email campaign via the API, and understand where the API stops and the Benchmark builder takes over.
api: benchmark-email:benchmark-email-api
generated: '2026-08-13'
method: generated
source: openapi/benchmark-email-api-openapi.json + https://developers.benchmarkemail.io/scenarios/manage-campaigns
scopes:
  - campaigns:write
  - domains:read
operations:
  - get_api_email_domain
  - get_api_email_template
  - getEmailTemplateCategories
  - get_api_email_campaign
  - post_api_email_campaign
  - get_api_email_campaign_by_campaignId
  - patch_api_email_campaign_by_campaignId
  - post_api_email_campaign_by_campaignId_duplicate
---

# Create a Benchmark Email campaign

## Know the boundary before you start

The API can create and configure a campaign. It **cannot** schedule one, send one, cancel a
send, or test-send. Those are UI-only, by design — this is what Benchmark Email calls "safe
sending by design". An agent that promises to send a campaign is promising something the API
does not do.

## 1. Verify the sender

`GET /api/email/domain` (`get_api_email_domain`) — the campaign's `from` address must sit on
a verified sending domain. The response carries the DKIM records and a `status` of `pending`
or `success`. Read-only: **verifying a new domain is a web flow**, so if the domain is not
verified, stop and hand back to a human.

`GET /api/email/domain/grey-label` (`get_api_email_domain_grey_label`) returns the account's
grey-label domain if there is one.

## 2. Pick a starting point

- Browse designs: `GET /api/email/template` (`get_api_email_template`), filtered by the
  categories from `GET /api/email/template/categories` (`getEmailTemplateCategories`).
- Or reuse a previous send: `POST /api/email/campaign/{campaignId}/duplicate`
  (`post_api_email_campaign_by_campaignId_duplicate`). Duplicating inherits the design,
  which is the practical way to get a good-looking campaign out of the API.

## 3. Create

`POST /api/email/campaign` (`post_api_email_campaign`) with the campaign name, `from`
address, subject, preview text, body and the `lists` array of recipients.

Duplicate campaign names are rejected. Set `"allowDuplicates": true` if you actually want
one.

## 4. Configure

`PATCH /api/email/campaign/{campaignId}` (`patch_api_email_campaign_by_campaignId`) — include
the current `__v`, read from `GET /api/email/campaign/{campaignId}`
(`get_api_email_campaign_by_campaignId`). A stale `__v` returns `400 ConcurrencyError`.

## 5. Hand off

Tell the user the campaign is a `draft` and that scheduling and sending happen in the
Benchmark builder. Campaign `status` moves through `draft`, `scheduled`, `sending`, `sent`,
and can also be `paused`, `failed` or `cancelled`.

`DELETE /api/email/campaign/{campaignId}` (`delete_api_email_campaign_by_campaignId`) returns
204 and is not reversible — confirm with a human first.
