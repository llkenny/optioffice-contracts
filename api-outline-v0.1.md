# API Outline v0.1

## Overview
- **Base URL:** `/api/v1`
- **Media Type:** `application/json; charset=utf-8`
- **Authentication:** Bearer JWT (access & refresh tokens). `orgId` provided in the token or `X-Org-Id` header.
- **Errors:** `application/problem+json` as defined by [RFC7807](https://datatracker.ietf.org/doc/html/rfc7807).
- **Status Codes:** `200`, `201`, `202`, `204` for success; `4xx` for client errors; `5xx` for server errors.

### Error Schema
```json
{
  "type": "https://api.example.com/errors/validation",
  "title": "Validation failed",
  "status": 422,
  "detail": "price must be > 0",
  "instance": "/api/v1/offices"
}
```

### Pagination & Filtering
```json
{
  "items": [],
  "page": 1,
  "limit": 50,
  "total": 1234
}
```
- Use `page` and `limit` query parameters.
- Filtering via field-specific query parameters.
- Sorting via `sort` parameter, prefix with `-` for descending.

## Auth & Account
| Method | Path | Description | Success |
|--------|------|-------------|---------|
| POST | `/auth/register` | Register new user | `201` |
| POST | `/auth/login` | Exchange credentials for tokens | `200` |
| POST | `/auth/token/refresh` | Refresh access token | `200` |
| POST | `/auth/logout` | Invalidate refresh token | `204` |
| GET | `/me` | Current user profile | `200` |
| PATCH | `/me` | Update user profile | `200` |
| GET | `/orgs` | List organizations | `200` |
| POST | `/orgs` | Create organization | `201` |
| GET | `/orgs/{id}` | Get organization | `200` |
| PATCH | `/orgs/{id}` | Update organization | `200` |

**POST `/auth/register`**
```json
{
  "email": "user@example.com",
  "password": "secret",
  "name": "Alice"
}
```

**POST `/orgs`**
```json
{
  "name": "Acme LLC"
}
```

**PATCH `/me`**
```json
{
  "name": "Alice B."
}
```

## Office Spaces
| Method | Path | Description | Success |
|--------|------|-------------|---------|
| GET | `/offices` | List offices | `200` |
| POST | `/offices` | Create office | `201` |
| GET | `/offices/{id}` | Get office | `200` |
| PATCH | `/offices/{id}` | Update office | `200` |
| DELETE | `/offices/{id}` | Delete office | `204` |
| POST | `/offices/{id}/photos` | Upload photo | `201` |
| DELETE | `/photos/{photoId}` | Remove photo | `204` |

**POST `/offices`**

_Request_
```json
{
  "title": "Open space 120m²",
  "description": "Bright open layout near metro",
  "address": {
    "city": "Moscow",
    "street": "Tverskaya 1",
    "lat": 55.7558,
    "lon": 37.6176
  },
  "area": 120.0,
  "price": 50000,
  "currency": "RUB",
  "amenities": ["wifi", "parking"],
  "status": "draft"
}
```

_Response `201`_
```json
{
  "id": "space_123",
  "title": "Open space 120m²",
  "description": "Bright open layout near metro",
  "address": {
    "city": "Moscow",
    "street": "Tverskaya 1",
    "lat": 55.7558,
    "lon": 37.6176
  },
  "area": 120.0,
  "price": 50000,
  "currency": "RUB",
  "amenities": ["wifi", "parking"],
  "status": "draft"
}
```

**PATCH `/offices/{id}`**
```json
{
  "status": "published"
}
```

## Listings
| Method | Path | Description | Success |
|--------|------|-------------|---------|
| GET | `/listings` | List listings | `200` |
| POST | `/offices/{id}/listings` | Create listing | `201` |
| GET | `/listings/{id}` | Get listing | `200` |
| PATCH | `/listings/{id}` | Update listing | `200` |
| DELETE | `/listings/{id}` | Delete listing | `204` |
| POST | `/listings/{id}/publish` | Publish listing | `202` |
| POST | `/listings/{id}/unpublish` | Unpublish listing | `202` |
| GET | `/channels` | Supported channels | `200` |

**POST `/offices/{id}/listings`**
```json
{
  "channel": "avito",
  "overrides": {
    "price": 100000
  }
}
```

**PATCH `/listings/{id}`**
```json
{
  "status": "paused"
}
```

**POST `/listings/{id}/publish`**

_Response `202`_
```json
{
  "taskId": "task_123"
}
```

## Leads
| Method | Path | Description | Success |
|--------|------|-------------|---------|
| GET | `/leads` | List leads | `200` |
| POST | `/leads` | Create lead | `201` |
| GET | `/leads/{id}` | Get lead | `200` |
| PATCH | `/leads/{id}` | Update lead | `200` |
| POST | `/leads/{id}/notes` | Add note | `201` |

**POST `/leads`**
```json
{
  "name": "Bob",
  "phone": "+7 999 123 4567",
  "channel": "avito"
}
```

**PATCH `/leads/{id}`**
```json
{
  "status": "won"
}
```

## Payments & Bank Import
| Method | Path | Description | Success |
|--------|------|-------------|---------|
| GET | `/payments` | List payments | `200` |
| POST | `/payments` | Create payment | `201` |
| POST | `/bank-statements` | Upload statement | `202` |
| GET | `/bank-statements/{id}` | Get statement | `200` |
| POST | `/payments/match` | Apply matching rules | `200` |
| GET | `/matching-rules` | List rules | `200` |
| POST | `/matching-rules` | Create rule | `201` |

**POST `/payments`**
```json
{
  "officeId": "abc123",
  "amount": 50000,
  "currency": "RUB",
  "date": "2024-02-01"
}
```

**POST `/matching-rules`**
```json
{
  "pattern": "rent",
  "account": "revenue"
}
```

## Documents
| Method | Path | Description | Success |
|--------|------|-------------|---------|
| GET | `/documents` | List documents | `200` |
| POST | `/documents` | Create document metadata | `201` |
| POST | `/documents/{id}/file` | Upload file | `201` |
| GET | `/documents/{id}` | Get document | `200` |
| PATCH | `/documents/{id}` | Update document metadata | `200` |

**POST `/documents`**
```json
{
  "type": "contract",
  "officeId": "abc123",
  "title": "Lease agreement"
}
```

**PATCH `/documents/{id}`**
```json
{
  "title": "Lease agreement v2"
}
```

## Integrations
| Method | Path | Description | Success |
|--------|------|-------------|---------|
| GET | `/integrations` | List available & connected integrations | `200` |
| POST | `/integrations/cian/connect` | Connect CIAN | `201` |
| POST | `/integrations/avito/connect` | Connect Avito | `201` |
| POST | `/integrations/{provider}/sync` | Trigger sync | `202` |
| POST | `/webhooks` | Register webhook | `201` |
| DELETE | `/webhooks/{id}` | Remove webhook | `204` |

**POST `/integrations/cian/connect`**
```json
{
  "apiKey": "cian-secret"
}
```

**POST `/webhooks`**
```json
{
  "url": "https://example.com/hooks",
  "events": ["lead.created", "payment.matched"]
}
```

## Admin / Support
| Method | Path | Description | Success |
|--------|------|-------------|---------|
| GET | `/audit` | Audit log search | `200` |
| GET | `/health` | Service health | `200` |
| GET | `/metrics` | Prometheus metrics | `200` |
