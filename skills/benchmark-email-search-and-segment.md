---
name: benchmark-email-search-and-segment
description: Query Benchmark Email contacts by email, custom field, list membership or status, and organise the results into lists before a campaign.
api: benchmark-email:benchmark-email-api
generated: '2026-08-13'
method: generated
source: openapi/benchmark-email-api-openapi.json + https://developers.benchmarkemail.io/scenarios/search-contacts
scopes:
  - contacts:read
  - contacts:write
operations:
  - get_api_contact_structure
  - post_api_contact_search
  - get_api_contact_structure_by_contactStructureId_lists
  - post_api_contact_structure_by_contactStructureId_lists
  - post_api_contact_structure_by_contactStructureId_lists_merge
  - post_api_contact_structure_by_contactStructureId_lists_by_listId_duplicate
  - post_api_contact_export
---

# Search and segment Benchmark Email contacts

## 1. Search, do not list

`GET /api/contact` (`get_api_contact`) returns every contact with **no pagination**. Use it
only on a small account. For anything real, use
`POST /api/contact/search` (`post_api_contact_search`) with `page` and `size`.

The search body needs two things:

- a filter — by `KEY` (email), by `LIST_ID`, or by `FIELD_ID` for a custom field value
- a non-empty `source` array naming the fields to return, e.g. `["_id","key","fields"]`

Omitting `source` is a `400 ValidationError`. This is the single most common mistake against
this endpoint.

## 2. Work with lists

Lists are nested under a contact structure, not top-level. Fetch the structure id from
`GET /api/contact-structure` (`get_api_contact_structure`) first.

- Read: `GET /api/contact-structure/{contactStructureId}/lists`
  (`get_api_contact_structure_by_contactStructureId_lists`), paginated. Use `/lists/all`
  (`get_api_contact_structure_by_contactStructureId_lists_all`) when you want the lot.
- Create: `POST /api/contact-structure/{contactStructureId}/lists`
  (`post_api_contact_structure_by_contactStructureId_lists`) — returns 201.
- Merge several into a new one: `POST .../lists/merge`
  (`post_api_contact_structure_by_contactStructureId_lists_merge`).
- Duplicate: `POST .../lists/{listId}/duplicate`
  (`post_api_contact_structure_by_contactStructureId_lists_by_listId_duplicate`).

A list carries `totalContacts` and `totalCampaigns` so you can size a segment without
enumerating it. There is no direct "get the contacts on this list" operation — that is a
search with a `LIST_ID` filter.

## 3. Export

`POST /api/contact/export` (`post_api_contact_export`) when you need the data out rather than
paged through.

## Rules that bite

- List updates go through `PATCH .../lists/{listId}`
  (`patch_api_contact_structure_by_contactStructureId_lists_by_listId`), and deletes are
  permanent (204).
- `DELETE /api/contact-structure/{contactStructureId}/lists` deletes MULTIPLE lists in one
  call. Confirm with a human before calling it.
- Pace to roughly one request per second; the 3,600/hour budget is shared across every API
  key on the account, so a paginated crawl competes with every other integration.
