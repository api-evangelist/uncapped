---
name: Track funding applications end to end
description: Subscribe to application status webhooks, enrich applicant details, and reconcile application state for an applicant.
api: openapi/uncapped-partners-openapi-original.json
operations: [issueAccessToken, createSubscription, enrichApplicantDetails, getApplications, getApplication]
generated: '2026-07-21'
method: generated
---

# Track funding applications end to end

## Steps

1. **Authenticate** — `POST /partner/authentication/token` (`issueAccessToken`); use the returned `accessToken` as a Bearer token.
2. **Subscribe to status changes** — `POST /webhooks/subscriptions` (`createSubscription`) with `{"eventType": "APPLICATION_STATUS_CHANGED", "url": "https://your.host/webhooks"}`. Application statuses: `CREATED, STARTED, APPROVED, REJECTED, ACTIVE, CLOSED, ABANDONED`.
3. **Enrich applicant data (optional but recommended)** — `PUT /applicants/{partnerApplicantId}/enrichment/details` (`enrichApplicantDetails`) to pre-fill onboarding: `legalName`, `companyRegistrationId`, `mainRevenueSource` (`Amazon | Shopify | Walmart | Unknown`), `mainStoreUrl`, company `address`, and `applicantUsers[]`. An applicant user `id` is required whenever any user detail field (email, name, phone, date of birth, address) is provided. Use ISO 8601 dates, ISO 3166-1 alpha-3 countries/nationalities, ISO 3166-2 states.
4. **Reconcile state** — `GET /applicants/{partnerApplicantId}/applications` (`getApplications`) lists every application with `status[]` filtering and zero-based `page`/`size` pagination (default size 50, sort `createdAt,DESC`); `GET /applications/{applicationId}` (`getApplication`) returns one application with its `estimationId` and `applicantUsers[]`.

## Rules

- 404 on enrichment or listing means the applicant is unknown for your partner id — create the applicant journey first (estimation via `generatePreOfferEstimation` establishes the `partnerApplicantId`).
- 403 on `getApplication` means the application belongs to another partner.
- Errors: `{message, code, details[]}` envelope (see `errors/uncapped-problem-types.yml`); data model: `data-model/uncapped-data-model.yml`.
