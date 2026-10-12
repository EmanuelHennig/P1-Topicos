# Data Model: Campanhas solidárias

## Campaign

A valid registration creates a publicly visible campaign immediately. Publication is not a separate draft state.

| Field | Type | Rules |
|---|---|---|
| `id` | UUID text | Primary key. |
| `category` | Text enum | Required; clothing, food, animal adoption, or other. |
| `title` | Text | Required. |
| `description` | Text | Required. |
| `institution_name` | Text | Required for the public search filter. |
| `location_label` | Text | Required, publicly displayed venue text supplied by the organizer. |
| `municipality` | Text | Required normalized city name. |
| `state_code` | Text | Required Brazilian UF code; disambiguates municipality names. |
| `postal_code` | Text, nullable | Optional campaign venue CEP; public only if supplied for display. |
| `latitude`, `longitude` | Real | Required municipality-level WGS84 coordinate used for proximity ordering. Campaign venue coordinates are public campaign data. |
| `starts_on`, `ends_on` | Date text | Required `YYYY-MM-DD`; `ends_on >= starts_on`; end date is inclusive in `America/Sao_Paulo`. |
| `management_token_hash` | 32-byte blob | Unique SHA-256 digest of the per-campaign random token; raw token is never stored. |
| `created_at` | UTC timestamp | Required. |

**Lifecycle**: Campaign is public after valid creation. Search includes it while `ends_on` is today or later in the product timezone. There is no draft, moderation, edit, delete or cancellation lifecycle in this plan.

## Need

| Field | Type | Rules |
|---|---|---|
| `id` | UUID text | Primary key. |
| `campaign_id` | UUID text | Required foreign key to Campaign; cascade behavior is restricted to future approved deletion flows. |
| `description` | Text | Required requested item/resource. |
| `quantity` | Numeric, nullable | Positive when provided; no currency semantics. |
| `unit` | Text, nullable | Optional unit such as item or kilogram. |
| `status` | Text enum | `open` or `attended`; new needs start `open`. |
| `created_at` | UTC timestamp | Required. |
| `attended_at` | UTC timestamp, nullable | Set only on the transition from `open` to `attended`. |

**Lifecycle**: `open -> attended`. The transition is one-way in the MVP and performed atomically by the campaign's valid management token. Public campaign details expose open needs only.

## Update

| Field | Type | Rules |
|---|---|---|
| `id` | UUID text | Primary key. |
| `campaign_id` | UUID text | Required foreign key to Campaign. |
| `content` | Text | Required informational post. |
| `created_at` | UTC timestamp | Required; public updates are ordered newest first. |

Updates can be created only with the matching campaign management token. Public comments are not part of the model.

## Management Token

- Created with each campaign from 32 random bytes; encoded base64url for delivery.
- Raw value is returned only in the successful campaign-creation response and must be copied by the organizer.
- Only the SHA-256 digest is persisted in `Campaign.management_token_hash`.
- For writes, the server hashes the supplied Bearer token and compares it in constant time with the digest belonging to the route's campaign ID.
- Tokens are never accepted in query strings, stored in analytics, returned by public reads, or used to authorize a different campaign.
- Recovery or rotation is not implemented; a lost token leaves the campaign without an administrative path in this MVP.

## Search Location (Transient)

The supporter submits either a city (and UF when needed to disambiguate) or a CEP. The server resolves coordinates, applies category/institution filters, orders every non-ended campaign by Haversine distance, returns the result, and discards the submitted locality and derived coordinates. Do not write search locations to SQLite, browser storage, application logs or analytics. A CEP lookup is sent to ViaCEP only after the supporter explicitly submits the search; the service receives the CEP.

## Geographic Reference Data

A local GeoNames Brazil gazetteer supplies city/state names and municipality-level coordinates. It is reference data, not user location or personal data. Include GeoNames attribution as required by CC BY 4.0. ViaCEP maps a user-submitted CEP to city/UF; no ViaCEP response is retained. Tests use fictional campaigns and a fake lookup adapter.

## Relationships and Integrity

- Campaign `1 -> many` Needs.
- Campaign `1 -> many` Updates.
- Campaign `1 -> 1` management-token digest.
- Enable SQLite foreign keys; parameterize all SQL; enforce category, date ordering, positive quantities, token-digest uniqueness and valid need states with constraints where supported.
- Keep write transactions short. Marking a need attended must condition on its current `open` status so duplicate/concurrent requests cannot create a second transition.
