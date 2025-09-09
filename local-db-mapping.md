# iOS Local DB → New Domain Model Mapping

## Local storage overview
The current iOS client keeps business data in plain Swift structures and in JSON files via `FileStorageManager`; no CoreData, Realm or SQLite is used.

## Entity mapping
| Local entity/field | Type | Target entity/field | Notes |
|---|---|---|---|
| **Building** | | | |
| Building.id | UUID | – | Building concept removed; used only to group offices |
| Building.name | String | OfficeSpace.title (prefix) | Copied into office title: `"<building> <office number>"` |
| Building.address | String | OfficeSpace.address | Duplicated for every office |
| Building.imageData | Data | MediaAsset(kind: photo) | Uploaded per related office |
| Building.offices | [Office] | OfficeSpace | Each office becomes an `OfficeSpace` |
| **Office** | | | |
| Office.id | UUID | OfficeSpace.id | |
| Office.buildingId | UUID? | Org.id? | If multiple orgs exist, map building to owning `Org`, otherwise constant |
| Office.counterpartyId | UUID? | Payment.counterparty | Counterparty name looked up and written as string |
| Office.counterpartyName | String? | Payment.counterparty | |
| Office.number | String | OfficeSpace.title | Combined with building name |
| Office.floor | Int? | OfficeSpace.description | Stored as text `"Floor: N"` |
| Office.square | Double? | OfficeSpace.area | |
| Office.price | Decimal? | OfficeSpace.price | Currency assumed RUB |
| Office.planImageData | Data? | MediaAsset(kind: doc) | Floor plan attachment |
| Office.imageData | Data? | MediaAsset(kind: photo) | Main photo |
| Office.isRented | Bool | OfficeSpace.status | `true` → `archived`, `false` → `active` |
| **Counterparty** | | | |
| Counterparty.id | UUID | – | No entity in new model |
| Counterparty.name | String | Payment.counterparty | Stored as free-form string |
| Counterparty.inn | String | – | Not represented |
| Counterparty.contactName | String | – | Could go to Document.signer or Lead.contact |
| Counterparty.contactPhone | String | – | Could go to Lead.contact |
| Counterparty.payments | [Payment] | Payment | Each payment migrated separately |
| **Payment** | | | |
| Payment.id | UUID | Payment.id | |
| Payment.counterpartyId | UUID? | Payment.counterparty | Look up counterparty.name |
| Payment.rentPayYear + rentPayMonth | Int? | Payment.date | First day of month |
| Payment.servicePayYear + servicePayMonth | Int? | Payment.date | Second payment record if both months present |
| – | – | Payment.amount / currency / method | Not available, defaults required |

## Risks and considerations
* Building information is not modeled separately in the new domain; images and addresses will be duplicated across offices.
* Counterparty details (INN, contacts) have no dedicated place and may be lost unless stored in document metadata.
* Legacy payments lack amount, currency and method; these must be defaulted or collected manually during migration.
* Month-based rent/service fields need conversion into exact dates; ambiguity when both rent and service parts exist.

## Migration plan (one‑off)
1. Export legacy data to JSON using existing models.
2. For each building → office pair create `OfficeSpace` records and related `MediaAsset` entries.
3. For each counterparty payment create `Payment` records with derived dates and counterparty name.
4. Review and manually enrich missing attributes (amounts, org ownership, contact details).
5. Import constructed entities to the new backend via DataConnect or REST API.
