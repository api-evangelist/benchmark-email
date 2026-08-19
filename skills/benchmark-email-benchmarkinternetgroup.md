---
name: Benchmarkinternetgroup
description: Use when building integrations with Benchmark Email, automating contact and campaign management, syncing data between systems, or retrieving email performance reports. Agents should reach for this skill when users need to programmatically manage contacts, lists, campaigns, or access analytics via REST API.
metadata:
    mintlify-proj: benchmarkinternetgroup
    version: "1.0"
---

# Benchmark Email API Skill

## Product summary

The Benchmark Email API is a REST service for managing email marketing operations programmatically. Use it to create and manage contacts, organize them into lists, build and configure email campaigns, and retrieve performance reports. All requests require an API key in the `X-API-Key` header and are scoped by permissions (contacts:read, contacts:write, campaigns:read, campaigns:write, reports:read, domains:read).

**Key files and endpoints:**
- Base URL: `https://api-{region}-{cluster}.benchmarkemail.io` (find yours in Account Settings > API Keys)
- Authentication: `X-API-Key: bme_us_...` header (50-character key starting with `bme_`)
- Primary docs: https://developers.benchmarkemail.io
- Scenario guides cover: manage contacts, manage campaigns, manage lists, search contacts, view reports, manage custom fields, migration from legacy

## When to use

Reach for this skill when:
- A user needs to **create, update, or delete contacts** programmatically (requires `contacts:write` scope)
- A user wants to **search for contacts** by email, custom field, or list membership (requires `contacts:read`)
- A user needs to **create or manage email campaigns** — draft, configure, duplicate, delete (requires `campaigns:write` or `campaigns:read`)
- A user wants to **organize contacts into lists** — create, rename, merge, duplicate lists (requires `contacts:write`)
- A user needs to **retrieve email performance data** — dashboard summaries, campaign reports, engagement metrics (requires `reports:read`)
- A user is **integrating Benchmark Email with another system** — Zapier, n8n, custom app, CRM sync
- A user needs to **bulk import or migrate data** from another email platform
- A user wants to **export contacts** or **view contact history and events**

Do not use this skill for: scheduling/sending campaigns (UI-only), managing user accounts, billing operations, or authentication setup.

## Quick reference

### API Key Setup
1. Log in as account owner → Settings > API Keys
2. Click Create API Key, name it, select scopes
3. Copy immediately (shown only once)
4. Format: `bme_us_a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0u1v` (50 chars)

### Essential Scopes
| Scope | Grants | Use for |
|-------|--------|---------|
| `contacts:read` | Read contacts, lists, search, export | Read-only contact access |
| `contacts:write` | Create, update, delete contacts & lists | Full contact management |
| `campaigns:read` | Read campaigns, browse templates | View campaign data |
| `campaigns:write` | Create, update, delete, duplicate campaigns | Full campaign management |
| `reports:read` | Dashboard, campaign reports, engagement | Analytics and metrics |
| `domains:read` | View sending domains | Domain verification checks |

### Rate Limits
- **Hourly:** 3,600 requests/hour (~1 per second)
- **Monthly:** Contact limit × 10 (free) or × 100 (paid)
- **Headers:** Check `X-RateLimit-Remaining`, `X-Monthly-Remaining` on every response
- **Exceeded:** Returns `429 Too Many Requests` with `Retry-After` header

### Core Resources
| Resource | Endpoints | Key operations |
|----------|-----------|-----------------|
| **Contacts** | `/api/contact`, `/api/contact/{id}`, `/api/contact/search` | Create, read, update, delete, search, export |
| **Lists** | `/api/contact-structure/{id}/lists` | Create, read, update, delete, merge, duplicate |
| **Campaigns** | `/api/email/campaign`, `/api/email/campaign/{id}` | Create, read, update, delete, duplicate |
| **Reports** | `/api/reports/dashboard`, `/api/reports/email/*` | Dashboard, overall stats, per-campaign metrics |
| **Contact Structure** | `/api/contact-structure` | Fetch field definitions (required for contact ops) |

### Common Request Pattern
```bash
curl -X GET \
  -H "X-API-Key: bme_us_..." \
  -H "Content-Type: application/json" \
  "https://api-us-west-2-a.benchmarkemail.io/api/contact"
```

## Decision guidance

### When to use POST vs PATCH vs PUT for contacts
| Scenario | Method | Notes |
|----------|--------|-------|
| Create new contact | POST `/api/contact` | Requires `key` (email), `contactStructureId`, optional fields/lists |
| Update only status | PATCH `/api/contact/{id}` | Only accepts `status` field; simpler than PUT |
| Update any field (email, custom fields, lists, status) | PUT `/api/contact/{id}` | Requires full contact body + current `__v` version; replaces entire record |
| Delete contact | DELETE `/api/contact/{id}` | Permanent; cannot be undone |

### When to search vs list all contacts
| Scenario | Use |
|----------|-----|
| Find one contact by email (dedup check) | POST `/api/contact/search` with `KEY` filter |
| Find contacts in a specific list | POST `/api/contact/search` with `LIST_ID` filter |
| Find contacts by custom field value | POST `/api/contact/search` with `FIELD_ID` filter |
| Get all contacts (small database) | GET `/api/contact` (no pagination) |
| Get all contacts (large database) | POST `/api/contact/search` with pagination |

### When to use campaign endpoints vs UI
| Task | Use API | Use UI |
|------|---------|--------|
| Create draft campaign | ✓ | ✓ |
| Configure campaign (subject, body, lists) | ✓ | ✓ |
| Schedule campaign | ✗ | ✓ |
| Send campaign | ✗ | ✓ |
| Cancel sending campaign | ✗ | ✓ |
| Test-send campaign | ✗ | ✓ |
| View campaign reports | ✓ | ✓ |

## Workflow

### Typical contact management workflow
1. **Fetch contact structure** — `GET /api/contact-structure` to get field IDs (required for all contact ops)
2. **Check for duplicates** — `POST /api/contact/search` with email filter to avoid creating duplicate
3. **Create contact** — `POST /api/contact` with email, structure ID, field values, list assignments
4. **Update contact** — `GET /api/contact/{id}` to fetch current `__v`, then `PUT` with updated fields + `__v`
5. **Change status** — `PATCH /api/contact/{id}` to unsubscribe or reactivate (note: cannot reactivate Inactive contacts)
6. **Delete contact** — `DELETE /api/contact/{id}` (permanent)

### Typical campaign workflow
1. **List campaigns** — `GET /api/email/campaign?page=1&size=25` to see existing campaigns
2. **Create campaign** — `POST /api/email/campaign` with name, from address, subject, body
3. **Update campaign** — `PATCH /api/email/campaign/{id}` with `__v` to modify subject, lists, etc.
4. **Assign recipients** — Include `lists` array in create or update
5. **View templates** — `GET /api/email/template` to browse starting points
6. **Duplicate campaign** — `POST /api/email/campaign/{id}/duplicate` to reuse as template
7. **Send via UI** — Switch to web app (API cannot schedule/send)

### Typical report workflow
1. **Dashboard snapshot** — `GET /api/reports/dashboard?pastDays=30` for high-level metrics
2. **Overall performance** — `GET /api/reports/email/overall?pastDays=30` for aggregate stats
3. **Time-series data** — `GET /api/reports/email/overall/histogram?pastDays=30` for charting
4. **Campaign-specific** — `GET /api/reports/email/{campaignId}` for detailed metrics + link clicks
5. **Engagement timeline** — `GET /api/reports/email/{campaignId}/engagement?period=week` for when recipients opened/clicked

## Common gotchas

- **Contact status is case-sensitive.** Use `"Active"` and `"Inactive"`, not `"active"` or `"inactive"`. When setting Inactive, both `primary` and `secondary` are required: `{"primary": "Inactive", "secondary": "Unsubscribe"}`.
- **Inactive contacts cannot be reactivated via API.** This is intentional (compliance). To re-add an unsubscribed contact, they must opt in again.
- **PUT requires exact `__v` version.** Always fetch fresh with GET before PUT. If `__v` is stale, you get a `400 ConcurrencyError`. Increment `__v` on every successful update.
- **Search requires `source` array.** POST `/api/contact/search` will fail if you don't specify which fields to return: `"source": ["_id", "key", "fields"]`.
- **Campaign send/schedule is UI-only.** API can create and configure campaigns but cannot schedule or send them. Use the web app for those actions.
- **API key is shown only once.** Copy it immediately after creation. If lost, regenerate from API Keys page (invalidates old key).
- **Rate limit is per account, not per key.** Multiple keys share the same 3,600 req/hour budget.
- **Monthly quota resets at billing period start.** Check `X-Monthly-Remaining` header to avoid quota exhaustion.
- **Contact structure ID is required for all contact operations.** Fetch it first with `GET /api/contact-structure`.
- **List operations are nested under contact structure.** Lists belong to a structure: `/api/contact-structure/{id}/lists`.
- **Duplicate campaign names are rejected by default.** Set `"allowDuplicates": true` in POST body to allow same name.
- **From address must be verified.** Campaign `from` field must belong to a verified sending domain. Check with `GET /api/email/domain`.

## Verification checklist

Before submitting work with the Benchmark Email API:

- [ ] API key is valid and has required scopes (check Settings > API Keys)
- [ ] Base URL matches account region (found in API Keys page)
- [ ] All required fields are present in request body (e.g., `contactStructureId` for contacts, `source` for search)
- [ ] Contact status values use correct capitalization (`Active`, `Inactive`, `Unsubscribe`)
- [ ] PUT requests include current `__v` value from a fresh GET
- [ ] Search requests include non-empty `source` array with valid field names
- [ ] Rate limit headers are monitored (`X-RateLimit-Remaining`, `X-Monthly-Remaining`)
- [ ] Error responses are checked for `errorType` and `message` (not just HTTP status)
- [ ] Scope errors (403) are resolved by updating key permissions, not retrying
- [ ] Campaign send/schedule operations are deferred to UI (not attempted via API)
- [ ] Contact dedup check is performed before creating new contacts (search by email first)

## Resources

**Comprehensive navigation:** https://developers.benchmarkemail.io/llms.txt (page-by-page index of all docs)

**Critical reference pages:**
- [Introduction & Quick Start](https://developers.benchmarkemail.io/introduction) — API overview, base URL discovery, first request
- [Authentication & Scopes](https://developers.benchmarkemail.io/authentication) — API key creation, scope reference, account standing
- [Rate Limits & Errors](https://developers.benchmarkemail.io/rate-limits) — rate limit headers, retry logic, monthly quotas
- [Manage Contacts Scenario](https://developers.benchmarkemail.io/scenarios/manage-contacts) — CRUD operations, status rules, version control
- [Search Contacts Scenario](https://developers.benchmarkemail.io/scenarios/search-contacts) — filter operators, pagination, dedup patterns
- [Manage Campaigns Scenario](https://developers.benchmarkemail.io/scenarios/manage-campaigns) — campaign lifecycle, template browsing, limitations
- [View Reports Scenario](https://developers.benchmarkemail.io/scenarios/view-reports) — dashboard, performance metrics, engagement timelines

---

> For additional documentation and navigation, see: https://developers.benchmarkemail.io/llms.txt