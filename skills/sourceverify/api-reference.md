# SourceVerify API Reference

Base URL: `https://sourceverify.ai/api/verify-reference`

## Authentication

All requests require a Bearer token:
```
Authorization: Bearer sk_...
```

## Endpoints

### POST /api/verify-reference
Submit references for verification.

**Request:**
```json
{
  "references": ["Reference text 1", "Reference text 2"]
}
```
- Max 10 references per request
- Max 1,200 characters per reference

**Response (200):**
```json
{
  "jobIds": ["job_abc123", "job_def456"],
  "documentId": "doc_xyz789",
  "message": "2 references submitted for verification",
  "tokensAvailable": 48
}
```

### POST /api/verify-reference/results
Poll for verification results.

**Request:**
```json
{
  "jobIds": ["job_abc123", "job_def456"]
}
```

**Response (200):**
```json
{
  "jobs": [
    {
      "id": "job_abc123",
      "state": "completed",
      "result": {
        "status": "verified",
        "message": "### Reference Match\n...",
        "reference": "Kuhn, T. S. (1962). *The structure of scientific revolutions*. University of Chicago Press.",
        "field_results": {
          "title": "MATCH",
          "authors": "MATCH",
          "year": "MATCH",
          "venue": "MATCH",
          "doi": "ABSENT",
          "url": "ABSENT"
        },
        "match_set_size": 4,
        "contained_set_size": 0
      }
    },
    {
      "id": "job_def456",
      "state": "active",
      "progress": {}
    }
  ]
}
```

**Job states:** `completed`, `active`, `waiting`, `failed`, `not-found`

**Verification statuses:** `verified`, `verified-with-error`, `unverified`, `needs-human`

**Field labels:** `MATCH`, `MATCH_WITH_TYPO`, `CONTAINS`, `ABSENT`, `UNCONFIRMED`, `CONTRADICTION`

### GET /api/verify-reference/balance
Check token balance.

**Response (200):**
```json
{
  "totalAvailable": 50,
  "subscriptionTokens": 50,
  "purchasedTokens": 0,
  "pendingTokens": 0,
  "plan": "Starter",
  "context": "user"
}
```

### GET /api/verify-reference/history
List past verification documents.

**Query params:** `limit` (default 20, max 50), `offset` (default 0)

**Response (200):**
```json
{
  "documents": [
    {
      "id": "doc_xyz",
      "title": "API Verification",
      "status": "COMPLETED",
      "referenceCount": 5,
      "createdAt": "2026-03-07T...",
      "references": [...]
    }
  ],
  "pagination": {
    "total": 12,
    "limit": 20,
    "offset": 0,
    "hasMore": false
  }
}
```

### POST /api/verify-reference/cancel
Cancel pending jobs and refund tokens.

**Request:**
```json
{
  "documentId": "doc_xyz789"
}
```
Or:
```json
{
  "jobIds": ["job_abc123"]
}
```

**Response (200):**
```json
{
  "success": true,
  "cancelledJobs": 3,
  "refundedTokens": 3,
  "documentId": "doc_xyz789",
  "documentStatus": "FAILED"
}
```

## Error Codes

| Status | Meaning |
|--------|---------|
| 400 | Invalid request body |
| 401 | Missing or invalid API key |
| 402 | Insufficient tokens |
| 403 | Expired API key |
| 429 | Rate limited (10 req/min) |

## Rate Limits

10 requests per minute per IP. Each request can contain up to 10 references.
