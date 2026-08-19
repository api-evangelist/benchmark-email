---
name: benchmark-email-pull-reports
description: Pull Benchmark Email campaign performance and contact engagement into a dashboard, BI tool or digest.
api: benchmark-email:benchmark-email-api
generated: '2026-08-13'
method: generated
source: openapi/benchmark-email-api-openapi.json + https://developers.benchmarkemail.io/scenarios/view-reports
scopes:
  - reports:read
  - contacts:read
operations:
  - get_api_reports_dashboard
  - get_api_reports_email_overall
  - get_api_reports_email_overall_histogram
  - get_api_reports_email_by_id
  - get_api_reports_email_by_id_engagement
  - get_api_contact_events
  - get_api_contact_by_contactId_events
---

# Pull Benchmark Email reporting

A `reports:read` key is enough for everything in this skill except the event feed, which
needs `contacts:read`. Reports are computed projections, not stored records — there is
nothing to write back.

## Account level

- `GET /api/reports/dashboard?pastDays=30` (`get_api_reports_dashboard`) — the snapshot the
  Benchmark dashboard renders. Start here for a weekly digest.
- `GET /api/reports/email/overall?pastDays=30` (`get_api_reports_email_overall`) — aggregate
  send performance across campaigns.
- `GET /api/reports/email/overall/histogram?pastDays=30`
  (`get_api_reports_email_overall_histogram`) — the same data bucketed by event type over
  time. This is your chart series.

## Campaign level

- `GET /api/reports/email/{id}` (`get_api_reports_email_by_id`) — per-campaign metrics
  including link clicks. `{id}` is the campaign `_id` from
  `GET /api/email/campaign` (`get_api_email_campaign`), whose records already carry a `stats`
  block with `clicks.rate`, `sentCount` and `failedCount` if a summary is all you need.
- `GET /api/reports/email/{id}/engagement?period=week`
  (`get_api_reports_email_by_id_engagement`) — when recipients opened and clicked.

## Contact level

`GET /api/contact/events` (`get_api_contact_events`) is the account-wide activity feed, and
`GET /api/contact/{contactId}/events` (`get_api_contact_by_contactId_events`) narrows it to
one person. Filter with `types` from the 14-value enum — `contact-created`,
`contact-updated`, `contact-update-failed`, `contact-unsubscribed`, `contact-reactivated`,
`email-sent`, `email-bounced`, `email-delivered`, `email-delayed`, `email-rejected`,
`email-complaint`, `email-opened`, `email-clicked`, `email-skipped` — and bound it with
`pastDays` (1-90, default 30). Paginate with `page` and `size`.

**This is the only event surface on the v1 API and it is pull-only.** There are no v1
webhooks. If you need push, it exists only on the classic v3.0 API
(`POST /Contact/{ListID}/Webhooks`), which is a different host and a different auth header —
see `asyncapi/benchmark-email-events.yml`.

## Pacing

A nightly report job that walks 90 days of events one page at a time will eat into the same
3,600 requests/hour that the rest of the account shares. Read `X-RateLimit-Remaining` and
`X-Monthly-Remaining` on every response, cache aggregates, and back off on 429 using
`Retry-After`.
