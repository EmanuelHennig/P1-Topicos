# Quickstart: Validation Guide

This is the planned validation workflow after the application scaffold and scripts are implemented.

## Prerequisites

- Node.js 24 LTS and npm.
- Network access for a live CEP search through ViaCEP. Automated tests use a fake and do not require network access.
- The GeoNames Brazil municipality catalog installed locally with the required CC BY attribution.

## Start the API

```powershell
cd backend
npm install
npm run db:migrate
npm test
npm run dev
```

Expected: migrations create a local SQLite file; the API starts with campaign routes available; all domain and HTTP tests pass.

## Start the web client

In another terminal:

```powershell
cd frontend
npm install
npm test
npm run dev
```

Expected: the web client opens in a browser and communicates with the local API.

## Business-Rule Scenarios

1. Register a fictional campaign with a valid category, venue, institution and inclusive date range. Expect a public campaign and a management token in the creation response. Confirm the database contains only the token digest.
2. Submit a campaign with missing category, location or dates, and one whose end precedes its start. Expect validation errors and no public campaign.
3. Search by a fictional city and by a valid CEP fixture. Expect all campaigns ending today or later, sorted by municipality-level distance, with category/institution filters applied together. Confirm the search input and derived search coordinates are absent from SQLite and logs.
4. Use an invalid or unknown city/CEP. Expect a correctable validation/provider error and no device-location prompt.
5. Open campaign details without any token. Expect public campaign data, open needs and updates.
6. Create a need, mark it attended and publish an update with the matching campaign token. Expect the need to disappear from open needs and the update to appear publicly.
7. Repeat administrative requests with no token, an invalid token and a token from another campaign. Expect refusal and no data mutation.
8. Navigate campaign creation, search, filters and administration using only the keyboard at narrow and desktop viewport widths. Expect visible focus and no horizontal overflow that blocks content.

## Checks

- Backend: `npm test` runs unit and Supertest integration tests against isolated temporary/in-memory SQLite databases.
- Frontend: `npm test` runs React interaction tests with Vitest and Testing Library.
- Before deployment: verify HTTPS, confirm the user-facing notice that CEP lookup is sent to ViaCEP, confirm logs omit request bodies and tokens, and review ViaCEP/GeoNames attribution and terms.
