# Research: Divulgação e descoberta de campanhas solidárias

**Date**: 2026-10-10

## Decisions

### Runtime and application boundaries

- **Decision**: Node.js 24 LTS with JavaScript ESM, Express 5, React 19 and a compatible stable Vite release. Use npm with separate `backend/` and `frontend/` projects.
- **Rationale**: Node 24 is an LTS line and meets current Vite/Vitest runtime requirements. Separate packages keep the client/server boundary clear without introducing a workspace framework.
- **Alternatives considered**: Node 26 Current was not selected because production guidance recommends an LTS release; TypeScript and a shared package were not required by the user and add setup surface to this MVP.

### Persistence

- **Decision**: SQLite through `better-sqlite3`, SQL migrations and parameterized statements. Keep the database on the API host; enable foreign keys and WAL.
- **Rationale**: Direct SQL and a local file fit a small single-instance MVP. WAL allows readers alongside a writer but still serializes writers and depends on same-host shared memory; it is not a network-filesystem deployment strategy.
- **Alternatives considered**: A client/server database or ORM would add deployment or abstraction overhead without a stated scale requirement. A native Node SQLite API was not selected to avoid tying the plan to a runtime experimental surface.

### Business-rule tests

- **Decision**: Vitest for backend unit/integration tests and frontend interaction tests, Supertest for Express HTTP assertions, and Testing Library for React.
- **Rationale**: Vitest works with Vite and runs in Node; Supertest exercises the real Express middleware and routes without opening a production port. Location integration is replaced with a deterministic fake in tests.
- **Alternatives considered**: Separate test runners for API and UI would add configuration. Browser end-to-end infrastructure is not required to test the specified domain rules in this phase.

### Management credential

- **Decision**: Generate a 32-byte cryptographically secure random token per campaign; return it only after successful campaign registration, persist only its SHA-256 digest, and require it in an Authorization Bearer header for campaign-scoped writes.
- **Rationale**: High-entropy random credentials are not user passwords and can be compared by digest without a slow password hash. A campaign-specific digest prevents one campaign's token from authorizing another campaign. Excluding tokens from URLs and logs reduces accidental disclosure.
- **Alternatives considered**: Email/password accounts conflict with the latest user instruction. A shared administrator account or global token would weaken campaign isolation. Token recovery/rotation is not included because no recovery identity or flow was specified; losing the displayed token means the campaign cannot be administered in this MVP.

### Manual locality resolution

- **Decision**: Resolve city + state from a locally imported GeoNames Brazil gazetteer. For a manually entered CEP, send a single user-triggered lookup from the backend to ViaCEP and use only the returned city/state to select the same local city coordinates. Do not persist the input or coordinates of a search, cache them, or log request bodies. Disclose that CEP lookup is sent to ViaCEP. Keep campaign venue coordinates as public campaign data.
- **Rationale**: City lookup remains local. ViaCEP documents individual CEP requests, rejects malformed/nonexistent codes, and warns against bulk harvesting. A local gazetteer means the app need not send city searches to a geocoder. Municipality-level coordinates are an intentionally coarse MVP approximation.
- **Alternatives considered**: The public Nominatim service permits moderate end-user-triggered searches but has a strict 1 request/second limit, requires attribution and an identifying User-Agent, and asks clients not to submit personal/confidential data. It is not used for supporter locations. GeoNames' separate Brazil postal-code dump contains only major postal codes, so it is not sufficient for arbitrary CEP lookup.
- **Operational caveat**: ViaCEP receives the manually submitted CEP; the application's no-retention guarantee does not control the provider's own logs. If that disclosure is unacceptable, a licensed offline CEP dataset or an approved provider with suitable retention terms must be selected before implementation.

### Distance and lifecycle

- **Decision**: Use municipality-level WGS84 coordinates and the Haversine formula for straight-line ordering. Include published campaigns ending today or later; exclude campaigns whose end date has passed. Sort ties by start date then campaign ID.
- **Rationale**: This implements the clarified no-radius search and yields deterministic results without a spatial database extension.
- **Alternatives considered**: Road distance, street-level geocoding, maximum-radius filters and map directions are not in the requested MVP.

## Source Notes

- Node release status: https://nodejs.org/en/about/previous-releases
- Express production security: https://expressjs.com/en/advanced/best-practice-security.html
- SQLite WAL behavior and limitations: https://www.sqlite.org/wal.html
- Vite requirements and setup: https://vite.dev/guide/
- Vitest requirements and Vite integration: https://vitest.dev/guide/
- ViaCEP request and response behavior: https://viacep.com.br/
- GeoNames data fields and license (CC BY 4.0): https://download.geonames.org/export/dump/
- GeoNames postal data caveat for Brazil: https://www.geonames.org/export/zip/
- Nominatim usage and privacy policy: https://operations.osmfoundation.org/policies/nominatim/
