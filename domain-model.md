# Domain Model

This document describes entities, fields, constraints and indexes for OptiOffice domain.

## Entities

### User
| Field | Type | Constraints |
|-------|------|-------------|
| id | UUID | primary key |
| email | string | unique, not null |
| name | string |  |
| auth | json | auth data |

### Org
| Field | Type | Constraints |
|-------|------|-------------|
| id | UUID | primary key |
| name | string | not null |
| ownerId | UUID | FK -> User.id |

**Indexes**: unique(name); index on ownerId

### OfficeSpace
| Field | Type | Constraints |
|-------|------|-------------|
| id | UUID | primary key |
| orgId | UUID | FK -> Org.id |
| title | string | not null |
| description | text | |
| address | string | |
| geo | point | |
| amenities | string[] | |
| area | float | |
| price | decimal | |
| currency | string | |
| status | enum(draft/active/archived) | |
| createdAt | timestamp | default now |

**Indexes**: unique(orgId, title)

### MediaAsset
| Field | Type | Constraints |
|-------|------|-------------|
| id | UUID | primary key |
| ownerType | enum(office/document) | |
| ownerId | UUID | references entity by ownerType |
| kind | enum(photo/doc) | |
| url | string | |
| meta | json | |

**Indexes**: index(ownerType, ownerId)

### Listing
| Field | Type | Constraints |
|-------|------|-------------|
| id | UUID | primary key |
| officeId | UUID | FK -> OfficeSpace.id |
| channel | enum(avito/cian/site) | |
| extId | string | nullable |
| status | enum(draft/published/archived) | |
| priceOverride | decimal | nullable |
| titleOverride | string | nullable |
| publishedAt | timestamp | nullable |
| lastSyncAt | timestamp | |

**Indexes**: unique(officeId, channel)

### Lead
| Field | Type | Constraints |
|-------|------|-------------|
| id | UUID | primary key |
| channel | enum | |
| listingId | UUID | FK -> Listing.id, nullable |
| contact | json | |
| message | text | |
| status | enum(new/processing/closed) | |
| sourcePayload | json | |

**Indexes**: index(listingId)

### Payment
| Field | Type | Constraints |
|-------|------|-------------|
| id | UUID | primary key |
| orgId | UUID | FK -> Org.id |
| date | date | |
| amount | decimal | |
| currency | string | |
| counterparty | string | |
| method | enum(card/transfer/cash) | |
| memo | string | nullable |
| officeId | UUID | FK -> OfficeSpace.id, nullable |
| contractId | UUID | FK -> Document.id, nullable |
| status | enum(matched/unmatched) | |

**Indexes**: index(orgId, date)

### BankStatement
| Field | Type | Constraints |
|-------|------|-------------|
| id | UUID | primary key |
| orgId | UUID | FK -> Org.id |
| period | daterange | |
| fileRef | string | |
| parsedRows | json[] | |

### MatchingRule
| Field | Type | Constraints |
|-------|------|-------------|
| id | UUID | primary key |
| orgId | UUID | FK -> Org.id |
| pattern | string | |
| targetType | enum(payment/office) | |
| targetId | UUID | |

### Document
| Field | Type | Constraints |
|-------|------|-------------|
| id | UUID | primary key |
| orgId | UUID | FK -> Org.id |
| officeId | UUID | FK -> OfficeSpace.id, nullable |
| type | enum(contract/invoice/act) | |
| number | string | |
| date | date | |
| fileRef | string | |
| signer | string | |
| status | enum(draft/signed/cancelled) | |

**Indexes**: unique(orgId, type, number)

### AuditEvent
| Field | Type | Constraints |
|-------|------|-------------|
| id | UUID | primary key |
| actor | UUID | FK -> User.id |
| action | string | |
| target | json | |
| ts | timestamp | |
| payload | json | |

