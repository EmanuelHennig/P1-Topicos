# Tasks: Divulgação e descoberta de campanhas solidárias

**Input**: Design documents from `/specs/001-descoberta-campanhas/`

**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/openapi.yaml`, `quickstart.md`

**Tests**: Included because the constitution requires automated tests for business rules.

**Organization**: Tasks are grouped by the four prioritized user stories. Tests precede implementation within each story.

## Format: `[ID] [P?] [Story?] Description`

- **[P]**: Tasks in separate files that can run in parallel after their listed phase dependencies.
- **[Story]**: User story label matching `spec.md`.
- Every task names the exact file path(s) to create or change.

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Create the independent backend and frontend projects and repository-wide generated-file exclusions.

- [ ] T001 [P] Create the Node.js 24 ESM backend package and scripts for development, migrations, and Vitest in `backend/package.json`.
- [ ] T002 [P] Scaffold the React 19/Vite frontend and Vitest/Testing Library scripts in `frontend/package.json`, `frontend/index.html`, `frontend/src/main.jsx`, and `frontend/src/App.jsx`.
- [ ] T003 [P] Ignore `node_modules`, environment files, local SQLite databases/WAL files, build outputs, and test artifacts in `.gitignore`.

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Establish shared persistence, configuration, location reference data, Express middleware, and isolated test helpers before any story implementation.

- [ ] T004 Load validated environment configuration, local database path, ViaCEP base URL, and provider timeout without logging secrets or search input in `backend/src/config.js`.
- [ ] T005 Open SQLite with `better-sqlite3`, enable foreign keys and WAL, set a bounded busy timeout, and reject network-filesystem database paths in `backend/src/db/connection.js`.
- [ ] T006 Create the initial SQL schema in `backend/src/db/migrations/001_initial_schema.sql`: Campaign fields include required title/description/institution/location, category limited to clothing/food/animal adoption/other, required WGS84 municipality coordinates, `starts_on`/`ends_on` as `YYYY-MM-DD` with `ends_on >= starts_on`, and a unique 32-byte management-token digest; Need has `open`/`attended` status and positive quantity when present; Update has required content and campaign foreign key.
- [ ] T007 Apply ordered SQL migrations once at startup and expose migration status to tests in `backend/src/db/migrate.js`.
- [ ] T008 Configure JSON parsing, Helmet, route mounting, redacted request logging, and consistent error responses without recording request bodies or Authorization headers in `backend/src/app.js`, `backend/src/middleware/requestLogger.js`, and `backend/src/middleware/errorHandler.js`.
- [ ] T009 Add the GeoNames Brazil gazetteer source and required CC BY 4.0 attribution in `backend/data/geonames/BR.zip` and `backend/data/geonames/README.md`.
- [ ] T010 Build a reproducible importer and local municipality lookup from the GeoNames source, keyed by normalized city and UF and returning municipality-level coordinates, in `backend/scripts/import-geonames.js` and `backend/src/services/geonamesCatalog.js`.
- [ ] T011 Provide isolated temporary SQLite setup, fictional campaign fixtures, and a deterministic fake ViaCEP adapter for tests in `backend/tests/helpers/testDatabase.js` and `backend/tests/helpers/fakeViaCep.js`.

**Checkpoint**: Both projects install; migrations and local geography are available; test suites can run without a live CEP service.

---

## Phase 3: User Story 1 - Cadastrar e publicar campanha (Priority: P1)

**Goal**: An organizer registers a valid campaign, publishes it immediately, and receives its campaign-scoped management token.

**Independent Test**: Register with fictional values; verify a public campaign response and one raw token response, while SQLite stores only its digest. Invalid required fields or reversed dates must not create a public campaign.

### Tests for User Story 1

- [ ] T012 [P] [US1] Test campaign category, required fields, municipality location, and `starts_on`/`ends_on` constraints in `backend/tests/unit/campaignValidation.test.js`.
- [ ] T013 [P] [US1] Test `POST /api/campaigns` for valid creation, missing required fields, invalid category, reversed dates, and digest-only persistence in `backend/tests/integration/campaignRegistration.test.js`.

### Implementation for User Story 1

- [ ] T014 [US1] Validate Campaign input: category is clothing/food/animal adoption/other; title, description, institution name, location label, city, UF, starts_on, and ends_on are required; dates use `YYYY-MM-DD` and end date is inclusive with `ends_on >= starts_on` in `backend/src/domain/campaign.js`.
- [ ] T015 [US1] Persist campaign fields and only a unique 32-byte token digest with parameterized SQL in `backend/src/repositories/campaignRepository.js`.
- [ ] T016 [US1] Resolve campaign city/UF through the local GeoNames catalog, generate a 32-byte random base64url token, store its SHA-256 digest, and return the raw token only in the creation result in `backend/src/services/registerCampaign.js`.
- [ ] T017 [US1] Implement `POST /api/campaigns` and the public base campaign read needed to verify publication, then mount the routes in `backend/src/routes/campaignCreationRoutes.js`, `backend/src/routes/campaignReadRoutes.js`, and `backend/src/app.js`.
- [ ] T018 [P] [US1] Build the accessible campaign registration form with category, title, description, institution, venue, dates, and field-level validation in `frontend/src/pages/CampaignCreatePage.jsx`.
- [ ] T019 [P] [US1] Show the newly issued token with copy action and a clear save-now warning, without persisting it in browser storage, in `frontend/src/components/ManagementTokenReveal.jsx` and `frontend/src/pages/CampaignCreatedPage.jsx`.

**Checkpoint**: A valid campaign is public immediately; invalid records are rejected; the raw campaign token is not persisted or logged.

---

## Phase 4: User Story 2 - Encontrar campanhas próximas (Priority: P1)

**Goal**: A supporter searches manually by city/UF or CEP, filters by category and institution, and sees every non-ended campaign ordered by approximate distance.

**Independent Test**: Use fictional campaigns and a fake CEP resolver; verify city and CEP searches, combined filters, no radius cutoff, ended-campaign exclusion, deterministic ordering, and no persisted search location.

### Tests for User Story 2

- [ ] T020 [P] [US2] Test city/UF disambiguation, malformed/unrecognized CEP handling, Haversine ordering, and deterministic ties in `backend/tests/unit/campaignSearch.test.js`.
- [ ] T021 [P] [US2] Test `POST /api/campaign-search` with fake GeoNames/ViaCEP data for category and institution filters together, all non-ended results, ended exclusion, no distance cutoff, provider errors, and no location persistence/logging in `backend/tests/integration/campaignSearch.test.js`.

### Implementation for User Story 2

- [ ] T022 [US2] Implement the server-side ViaCEP adapter for one user-submitted eight-digit CEP at a time; validate input before the request, apply a timeout, and never log or cache the CEP in `backend/src/services/viaCepClient.js`.
- [ ] T023 [US2] Resolve city plus UF locally or resolve a CEP through ViaCEP to city/UF and then local coordinates; keep query text and coordinates transient in `backend/src/services/searchLocation.js`.
- [ ] T024 [US2] Implement date filtering in `America/Sao_Paulo`, AND-combined category/institution filters, Haversine distance, no radius limit, and tie order by start date then campaign ID in `backend/src/services/searchCampaigns.js`.
- [ ] T025 [US2] Implement public `POST /api/campaign-search` with the OpenAPI request/response shape and redacted request handling in `backend/src/routes/campaignSearchRoutes.js` and mount it in `backend/src/app.js`.
- [ ] T026 [P] [US2] Build the Portuguese search form for city plus UF when needed or CEP, disclose the external ViaCEP lookup before CEP submission, and avoid browser persistence in `frontend/src/pages/CampaignSearchPage.jsx`.
- [ ] T027 [P] [US2] Render nearest-first campaign results, combined filters, loading/error states, and a no-results state in `frontend/src/components/CampaignResults.jsx`.

**Checkpoint**: Public search works without device location or supporter accounts; city lookup remains local; CEP input is not stored by the application.

---

## Phase 5: User Story 3 - Consultar necessidades e atualizações (Priority: P2)

**Goal**: A supporter opens a public campaign page and reads campaign details, open needs, and published updates without a token.

**Independent Test**: Seed a fictional campaign with open/attended needs and updates; public detail returns campaign data, only open needs, and updates newest-first with no authorization header.

### Tests for User Story 3

- [ ] T028 [P] [US3] Test public campaign detail, open-needs filtering, newest-first updates, unknown campaign behavior, and anonymous access in `backend/tests/integration/campaignDetails.test.js`.

### Implementation for User Story 3

- [ ] T029 [US3] Query one public campaign with its open needs and updates ordered newest-first in `backend/src/repositories/campaignDetailsRepository.js`.
- [ ] T030 [US3] Extend `GET /api/campaigns/{campaignId}` to return Campaign, `openNeeds`, and `updates` according to `contracts/openapi.yaml` in `backend/src/routes/campaignReadRoutes.js`.
- [ ] T031 [P] [US3] Render open needs with description and optional quantity/unit in `frontend/src/components/NeedsList.jsx`.
- [ ] T032 [P] [US3] Render campaign updates newest-first with their publication time in `frontend/src/components/UpdatesFeed.jsx`.
- [ ] T033 [US3] Compose the public campaign page with event location, period, open needs, and updates, without requesting a management token in `frontend/src/pages/CampaignDetailPage.jsx`.

**Checkpoint**: Public campaign details remain anonymous and attended needs never appear as open.

---

## Phase 6: User Story 4 - Administrar necessidades e atualizações (Priority: P2)

**Goal**: The organizer presents the token for one campaign to create needs, mark them attended, and publish updates; other or missing tokens cannot mutate data.

**Independent Test**: Seed two fictional campaigns with distinct tokens; verify valid-token writes are confined to one campaign, and absent/invalid/cross-campaign tokens cause no mutation.

### Tests for User Story 4

- [ ] T034 [P] [US4] Test token digest validation, absent/invalid credentials, and campaign scoping in `backend/tests/unit/campaignTokenAuth.test.js`.
- [ ] T035 [P] [US4] Test need creation, atomic open-to-attended transition, update publication, and rejection without or with another campaign's token in `backend/tests/integration/campaignAdministration.test.js`.

### Implementation for User Story 4

- [ ] T036 [US4] Require an Authorization Bearer token, hash it with SHA-256, compare digests in constant time, and authorize only the campaign in the route in `backend/src/middleware/requireCampaignToken.js`.
- [ ] T037 [P] [US4] Create needs with required description, optional positive quantity/unit, initial `open` state, and atomic one-way `open` to `attended` transition in `backend/src/services/needService.js`.
- [ ] T038 [P] [US4] Create campaign-scoped informational updates with required non-empty content and UTC creation time in `backend/src/services/updateService.js`.
- [ ] T039 [US4] Add token-protected need creation, need attendance, and update routes with no mutation on authorization failure in `backend/src/routes/campaignAdminRoutes.js` and mount them in `backend/src/app.js`.
- [ ] T040 [P] [US4] Add a management-token entry component that keeps the token in memory only and never places it in a URL, localStorage, or analytics in `frontend/src/components/ManagementTokenForm.jsx`.
- [ ] T041 [US4] Build the campaign management page for need creation, attended status, and update publication using the entered campaign token in `frontend/src/pages/CampaignManagePage.jsx`.

**Checkpoint**: Public read remains anonymous; only a token whose digest belongs to the path campaign can write; raw tokens never enter persistent browser storage or logs.

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: Verify accessibility, privacy boundaries, attribution, and the runnable quickstart across the complete requested MVP.

- [ ] T042 [P] Add keyboard-only and narrow/desktop viewport interaction tests for campaign creation, search/filtering, public detail, and campaign management in `frontend/tests/keyboardAccessibility.test.jsx`.
- [ ] T043 [P] Add privacy regression tests proving search input is absent from SQLite/logs and management tokens are absent from URLs/logs/browser persistent storage in `backend/tests/integration/privacyBoundaries.test.js` and `frontend/tests/privacyBoundaries.test.jsx`.
- [ ] T044 Update `specs/001-descoberta-campanhas/quickstart.md` with actual scripts and data import steps, then run every listed scenario and both project test commands.

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies; T001-T003 can run in parallel.
- **Foundational (Phase 2)**: Depends on setup; blocks all user story work. Within it, T009 data acquisition and T010 importer can proceed in parallel after agreeing on GeoNames input/output format.
- **User Stories (Phase 3+)**: Depend on the completed foundation and are listed in priority order.
- **Polish (Phase 7)**: Depends on the desired stories being complete.

### User Story Dependencies

- **US1 (P1)**: Starts after Phase 2; establishes campaign creation and campaign-scoped token issuance.
- **US2 (P1)**: Starts after Phase 2 and can be tested with fixtures; the full create-then-discover flow integrates with US1.
- **US3 (P2)**: Starts after Phase 2 and can be tested with seeded campaign content; its public route builds on US1's campaign read route.
- **US4 (P2)**: Starts after Phase 2 with seeded token fixtures; the organizer's end-to-end flow uses the token returned by US1.

### Within Each User Story

- Automated business-rule and integration tests are written first, then domain/repository logic, then routes, then interface.
- Persistence/migrations and shared test helpers are completed in Phase 2.
- Campaign venue coordinates are resolved during US1; supporter search locations are resolved transiently during US2.
- US3 reads needs/updates from fixtures before US4 enables public creation of those records.

### Parallel Opportunities

- Setup tasks T001-T003 use different files and can run together.
- In US1, T012 and T013 are independent test files; T018 and T019 are separate frontend files.
- In US2, T020 and T021 are independent test files; T026 and T027 are separate frontend files.
- In US3, T031 and T032 are independent display components after the contract is stable.
- In US4, T034 and T035 are independent test files; T037 and T038 are separate domain services; T040 can be built separately from the page composition.
- US1 and US2 can be staffed in parallel after Phase 2 if US2 uses fixtures; combined acceptance is run after both finish.

## Parallel Example: User Story 2

```text
Task: T020 Unit tests for city/CEP resolution and distance ordering in backend/tests/unit/campaignSearch.test.js
Task: T021 Search API integration tests with fake geocoders in backend/tests/integration/campaignSearch.test.js
Task: T026 Search form with manual city/UF or CEP in frontend/src/pages/CampaignSearchPage.jsx
Task: T027 Nearest-first results and empty/error states in frontend/src/components/CampaignResults.jsx
```

## Implementation Strategy

### Core Discovery Increment

1. Complete Phase 1 and Phase 2.
2. Complete US1 and US2; use fictional fixtures and verify the full register-then-find flow.
3. **STOP and VALIDATE**: run US1 and US2 independently plus the discovery quickstart.
4. This is the minimum end-to-end increment required by the constitution; US3 and US4 remain part of the requested complete MVP.

### Incremental Delivery

1. Setup + foundation, then US1 campaign creation/publication.
2. US2 manual location search and filters; validate the core public discovery flow.
3. US3 public campaign details, needs and updates.
4. US4 token-scoped administration of needs and updates.
5. Polish and run all privacy, accessibility, and quickstart checks.
