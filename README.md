# Benchmark Email

Benchmark Email is an email marketing platform with a REST API for managing contacts, lists, email campaigns, automations, reports, and transactional email delivery. The platform serves businesses globally and processes billions of emails per month.

## API

The Benchmark Email RESTful API v3.0 is available at `https://clientapi.benchmarkemail.com`. It provides programmatic access to:

- **Contacts and Lists** — create, search, update, and remove contacts; manage lists with batch operations
- **Email Campaigns** — create campaign shells, duplicate previous campaigns, and retrieve campaign details
- **Automation** — build and manage automation workflows
- **Reports and Analytics** — pull campaign performance metrics into BI tools and dashboards
- **Images** — upload, retrieve, update, and delete gallery assets
- **Webhooks** — receive real-time event notifications for subscriber activity, unsubscribes, field changes, and email events
- **Domain Management** — read-only access to sending domain verification status

## Authentication

The API uses an API key passed via the `AuthToken` request header. Keys support per-resource permission scoping (No Access / Read / Read & Write) across contacts, campaigns, reports, and domains. Keys can have optional expiration dates and can be deactivated or reactivated without rotation.

## Rate Limits

- 500 calls per 2 minutes
- 60,000 calls per day
- Hourly and monthly quotas also apply
- Rate limit status is returned in response headers on every call

## Pricing

API access is included on all plans at no extra cost.

| Plan | Contacts | Sends/Month | Price |
|------|----------|-------------|-------|
| Free | 500 | 2,500 | $0/mo |
| Pro | 1,000–100,000 | 10x contacts | $19–$499/mo |
| Enterprise | 100,000+ | Custom | Custom |

## Resources

- **Developer Portal**: https://developer.benchmarkemail.com/
- **GitHub**: https://github.com/BenchmarkEmail/RESTful-API-v3
- **Pricing**: https://www.benchmarkemail.com/pricing/
- **Status**: https://www.benchmarkemail.com/status/
- **Blog**: https://www.benchmarkemail.com/blog/
- **Product Updates**: https://www.benchmarkemail.com/product-updates/

## Maintainer

**Kin Lane** — kin@apievangelist.com
