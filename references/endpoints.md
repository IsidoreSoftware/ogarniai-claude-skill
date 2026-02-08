# Ogarni.AI API — Endpoint Reference

## Authentication

All endpoints require an API token passed via the `X-API-Key` header:

```
X-API-Key: oai_YOUR_TOKEN
```

- **Base URL:** `https://api.ogarni.ai`
- **Token creation:** app.ogarni.ai > Settings > API Tokens
- **Token format:** `oai_` prefix + base64url payload + HMAC-SHA256 signature
- **Scopes:** `read` (read-only), `write` (read+write), `admin` (full access)
- **Rate limits:** 1,000/hr (read), 2,000/hr (write), 5,000/hr (admin) — check `Retry-After` header on 429
- **Errors:** JSON with `error`, `message`, `timestamp`, `path`, `requestId` fields

---

## Purchase Documents

### GET /api/PurchaseDocuments/my
List user's purchase documents.

**Query Parameters:**
| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `from` | DateTime | - | Start date filter |
| `to` | DateTime | - | End date filter |
| `sortBy` | string | `purchase_time` | Sort field: `purchase_time` or `created_at` |
| `sortDirection` | string | `desc` | Sort order: `desc` or `asc` |

**Response:** `ProcessedPagedResponse<PurchaseDocumentDto>`
```json
{
  "items": [{
    "id": "uuid",
    "storeName": "Biedronka",
    "purchaseTime": "2024-01-15T14:30:00Z",
    "totalAmount": 45.99,
    "taxAmount": 8.50,
    "currency": "PLN",
    "paymentMethod": "card",
    "items": [{
      "id": "uuid",
      "name": "Mleko 3.2%",
      "price": 4.99,
      "quantity": 2,
      "totalPrice": 9.98,
      "category": "Spozywcze",
      "subcategory": "Nabiał",
      "discount": 0
    }],
    "documentIdentifier": "FV/2024/001",
    "source": 1,
    "createdAt": "2024-01-15T15:00:00Z",
    "hasDocumentFile": true,
    "tags": [{"id": "uuid", "name": "grocery"}],
    "groupId": "uuid",
    "hasDuplicates": false,
    "duplicateCount": 0,
    "ownedAmount": 45.99
  }]
}
```

### GET /api/PurchaseDocuments/{id}
Get single document. Returns `PurchaseDocumentDto` (same schema as above) or `204 No Content` if still processing.

### GET /api/PurchaseDocuments/{id}/image
Get document image. Returns binary image data with appropriate content type.

### GET /api/PurchaseDocuments/{id}/duplicates
Get duplicate suggestions for a document. Returns `DocumentDuplicatesResponse`.

---

## Categories

### GET /api/Categories
List all categories.

**Response:**
```json
{
  "categories": [{
    "polishName": "Spożywcze",
    "englishName": "Groceries",
    "emoji": "🛒",
    "subCategories": [{
      "polishName": "Nabiał",
      "englishName": "Dairy",
      "description": "Milk, cheese, yogurt...",
      "examples": "mleko, ser, jogurt",
      "emoji": "🥛"
    }]
  }]
}
```

---

## Tags

### GET /api/Tags
List user tags.

**Response:**
```json
[
  {"id": "uuid", "name": "grocery", "authorId": "uuid"}
]
```

---

## Weekly Summaries

### GET /api/weekly-summaries/latest
Get the most recent weekly summary.

**Response:** `PeriodSummaryDto`
```json
{
  "userId": "uuid",
  "periodStart": "2024-01-08T00:00:00Z",
  "periodEnd": "2024-01-14T23:59:59Z",
  "summary": "AI-generated spending summary text...",
  "dailySummaries": [{
    "date": "2024-01-08T00:00:00Z",
    "totalAmount": 125.50,
    "categoryBreakdown": [
      {"category": "Spożywcze", "totalAmount": 80.00},
      {"category": "Transport", "totalAmount": 45.50}
    ],
    "summary": "Daily summary text..."
  }],
  "categoryTotals": [
    {"category": "Spożywcze", "totalAmount": 350.00}
  ],
  "detailedCategoryTotals": [
    {"category": "Spożywcze", "subcategory": "Nabiał", "totalAmount": 120.00}
  ],
  "totalAmount": 850.00,
  "isSent": true,
  "createdAt": "2024-01-15T00:00:00Z"
}
```

### GET /api/weekly-summaries
List summaries. Params: `page` (default 1), `pageSize` (default 10). Response includes `X-Total-Count` header.

### GET /api/summaries/periods
Custom date range summary. Params: `startDate` (required), `endDate` (required), `granularity` (Day/Week/Month, default Day).

### GET /api/summaries/presets
Preset period summary. Params: `preset` (required: `current-week`, `current-month`, `last-week`, `last-month`), `granularity` (Day/Week/Month, default Day).

---

## Notifications

### GET /api/Notifications
List notifications.

**Query Parameters:**
| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `isRead` | bool | - | Filter by read status |
| `type` | enum | - | `Error`=1, `Warning`=2, `Info`=3, `Success`=4 |
| `category` | enum | - | `DocumentProcessing`=1, `WeeklySummary`=2, `System`=3, `DuplicateDetection`=4 |
| `pageSize` | int | 50 | Items per page |
| `pageNumber` | int | 1 | Page number |

**Response:**
```json
[{
  "id": "uuid",
  "type": 3,
  "category": 1,
  "title": "Document processed",
  "message": "Your receipt from Biedronka has been processed.",
  "isRead": false,
  "isUrgent": false,
  "readAt": null,
  "actionUrl": "/documents/uuid",
  "referenceId": "uuid"
}]
```

### GET /api/Notifications/unread-count
**Response:** `{"count": 5}`

---

## Groups

### GET /api/Groups
List groups. Params: `showArchived` (bool, optional).

**Response:**
```json
[{
  "id": "uuid",
  "name": "Our Apartment",
  "description": "Shared expenses",
  "ownerId": "uuid",
  "isActive": true,
  "isArchived": false,
  "memberCount": 2,
  "createdAt": "2024-01-01T00:00:00Z"
}]
```

### GET /api/Groups/{groupId}
Get group details. Same schema as above with owner details.

---

## Inbound Mailboxes

### GET /api/inbound-mailboxes
**Response:**
```json
[{
  "id": "uuid",
  "address": "user123@receipts.ogarni.ai",
  "localPart": "user123",
  "domain": "receipts.ogarni.ai",
  "isPrimary": true,
  "status": 1,
  "lastActivityAt": "2024-01-15T10:00:00Z"
}]
```

---

## Deduplication

### GET /api/Deduplication/suggestions
**Response:**
```json
[{
  "id": "uuid",
  "sourceDocumentId": "uuid",
  "targetDocumentId": "uuid",
  "confidenceScore": 0.95,
  "matchReason": "Same store, amount, and date",
  "status": 0
}]
```
Status: `0`=Pending, `1`=Accepted, `2`=Rejected, `3`=Dismissed

---

## Loyalty Accounts

### GET /api/external_loyalty
**Response:**
```json
[{
  "id": "uuid",
  "program": "Biedronka",
  "maskedLogin": "***@email.com",
  "status": 1,
  "createdAt": "2024-01-01T00:00:00Z"
}]
```

---

## Bank Statements

### GET /api/BankStatement/supported-banks
**Response:** `["mBank", "PKO BP", "ING", "Santander", ...]`

---

## Recurring Expenses

### GET /api/RecurringExpenses
Params: `includeInactive` (bool, default false).

**Response:**
```json
[{
  "id": "uuid",
  "name": "Netflix",
  "description": "Monthly subscription",
  "frequency": 1,
  "dayOfMonth": 15,
  "isActive": true,
  "category": "Rozrywka",
  "subCategory": "Streaming",
  "nextOccurrenceDate": "2024-02-15T00:00:00Z",
  "createdAt": "2024-01-01T00:00:00Z"
}]
```
Frequency: `0`=Weekly, `1`=Monthly, `2`=Quarterly, `3`=Yearly
