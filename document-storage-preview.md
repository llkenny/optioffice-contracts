# Document Storage & Preview

## Overview
This spec describes how contracts, invoices, and other documents are stored and previewed.
Documents are modeled as `MediaAsset` objects backed by blob storage and linked to an
`Organization`.

## MediaAsset & Validations
- **Types**: `pdf`, `docx`, `png`.
- **Max size**: 10 MB.
- **Validation**: content type and size checked on upload.

## Metadata
Each document records business metadata:
- `number`: external document number or invoice code.
- `date`: issuance date (ISO 8601).
- `counterparty`: contracting organization or individual.
- `purpose`: contract, invoice, act, or other free text.

## Pre‑signed URLs
Clients never access storage directly. Uploads and downloads use pre‑signed URLs:
- `TTL`: 10 minutes from generation.
- `scope`: `read` (download) or `write` (upload).

## Access Policies
- Only members of the owning organization can generate URLs.
- Upload allowed for `owner` and `accountant` roles.
- Read allowed for all organization members.
- URLs are single‑use and expire after TTL.

## Data Model
`Document` table:
| column           | type      | note |
|------------------|-----------|------|
| `id`             | uuid pk   |      |
| `org_id`         | uuid fk   | owning organization |
| `media_asset_id` | uuid fk   | blob reference |
| `number`         | text      | document number |
| `date`           | date      | issue date |
| `counterparty`   | text      | contracting party |
| `purpose`        | text      | business purpose |
| `created_at`     | timestamptz | |

**Indexes**
- `(org_id, date)` for filtering by period.
- `(org_id, number)` for quick lookup.

## API Examples
### Upload
1. `POST /orgs/{orgId}/documents`
   ```json
   {"number":"77","date":"2024-02-01","counterparty":"ACME","purpose":"contract","ext":"pdf"}
   ```
   → `201` with `id` and `upload_url` (pre‑signed, scope `write`).
2. Client `PUT` file to `upload_url`.

### Download / Preview
1. `GET /orgs/{orgId}/documents/{id}` → metadata.
2. `GET /orgs/{orgId}/documents/{id}/content` → `200` with pre‑signed `read` URL.

## Acceptance
- Uploads restricted to allowed types and size.
- URLs expire after 10 minutes and respect scope.
- Metadata searchable by number, date, and counterparty.
- Only organization members can manage documents.
