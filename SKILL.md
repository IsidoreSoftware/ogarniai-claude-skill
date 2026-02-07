---
name: ogarniai-api
description: >
  Guide for working with the Ogarni.AI personal finance API. Use this skill when users ask about
  programmatic access to their Ogarni.AI data, API tokens, making API requests, building integrations,
  or using the MCP server. Covers authentication, endpoints, rate limits, error handling, and code examples.
---

# Ogarni.AI API Guide

## Authentication

All API requests require an `X-API-Key` header with a personal API token:

```
X-API-Key: oai_YOUR_TOKEN
```

Tokens are created at **app.ogarni.ai → Settings → API Tokens**.

### Token Format
- Prefix: `oai_` followed by base64url-encoded payload and HMAC-SHA256 signature
- Token is shown only once at creation — store it securely

### Token Scopes
| Scope | Access |
|-------|--------|
| `read` | Read-only access to all resources |
| `write` | Read + write access |
| `admin` | Full access including token management |

## Base URLs

| Environment | URL |
|------------|-----|
| Production | `https://api.ogarni.ai` |

## Available Endpoints (Read-Only)

All endpoints below require `read` scope or higher.

### Purchase Documents
- `GET /api/PurchaseDocuments/my` — List receipts (params: `from`, `to`, `sortBy`, `sortDirection`)
- `GET /api/PurchaseDocuments/{id}` — Get single receipt details
- `GET /api/PurchaseDocuments/{id}/image` — Get receipt image
- `GET /api/PurchaseDocuments/{id}/duplicates` — Get duplicate suggestions

### Categories & Tags
- `GET /api/Categories` — List expense/income categories with subcategories
- `GET /api/Tags` — List user-defined tags

### Summaries
- `GET /api/weekly-summaries/latest` — Latest weekly spending summary
- `GET /api/weekly-summaries` — List weekly summaries (params: `page`, `pageSize`)
- `GET /api/summaries/periods` — Custom date range summary (params: `startDate`, `endDate`, `granularity`)
- `GET /api/summaries/presets` — Preset period summary (params: `preset`, `granularity`)

### Notifications
- `GET /api/Notifications` — List notifications (params: `isRead`, `type`, `category`, `pageSize`, `pageNumber`)
- `GET /api/Notifications/{id}` — Get notification details
- `GET /api/Notifications/unread-count` — Unread count

### Groups
- `GET /api/Groups` — List finance groups (params: `showArchived`)
- `GET /api/Groups/{groupId}` — Get group details

### Other
- `GET /api/inbound-mailboxes` — List inbound email addresses
- `GET /api/Deduplication/suggestions` — List duplicate document suggestions
- `GET /api/external_loyalty` — List loyalty program connections
- `GET /api/BankStatement/supported-banks` — List supported banks
- `GET /api/RecurringExpenses` — List recurring expenses (params: `includeInactive`)

## Rate Limits

| Scope | Hourly Limit |
|-------|-------------|
| `read` | 1,000 requests |
| `write` | 2,000 requests |
| `admin` | 5,000 requests |

Response headers: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`, `X-RateLimit-Reset-After`

On 429, check `Retry-After` header.

## Error Handling

All errors return JSON:
```json
{
  "error": "error_code",
  "message": "Human-readable description",
  "timestamp": "ISO 8601",
  "path": "/api/endpoint",
  "requestId": "req_xxxxx"
}
```

Common codes: `401` (invalid/expired token), `403` (insufficient scope), `404` (not found), `429` (rate limited).

## Security Best Practices

1. **Store tokens in env vars** — never hardcode in source
2. **Use `read` scope** unless write access is needed
3. **Rotate tokens** every 90 days
4. **Never log tokens** — they appear in full only at creation
5. **Use HTTPS** — all API endpoints require it

## MCP Server

An MCP server is available at `typescript/ogarniai-mcp-server/` providing all read-only endpoints as tools. See the README there for installation instructions with Claude Code, Cursor, and other MCP clients.

## Reference Files

For detailed endpoint parameters, response schemas, and code examples, load these reference files:
- `references/endpoints.md` — Complete endpoint reference with request/response schemas
- `references/examples.md` — Code examples in curl, Python, and JavaScript
