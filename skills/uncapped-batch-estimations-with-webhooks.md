---
name: Pre-load Signals for a cohort with batch estimations and webhooks
description: Subscribe to the batch-completed webhook, submit a batch of pre-offer estimation requests, and list the resulting estimations per applicant.
api: openapi/uncapped-partners-openapi-original.json
operations: [issueAccessToken, createSubscription, handlePreOfferEstimationBatch, getPreOfferEstimations]
generated: '2026-07-21'
method: generated
---

# Pre-load Signals for a cohort with batch estimations and webhooks

Batch estimations let you pre-load Signals for a cohort of applicants before they reach your UI, so the funding experience renders without a loading state.

## Steps

1. **Authenticate** — `POST /partner/authentication/token` (`issueAccessToken`); use the returned `accessToken` as a Bearer token.
2. **Subscribe to completion events** — `POST /webhooks/subscriptions` (`createSubscription`) with `{"eventType": "ESTIMATION_BATCH_COMPLETED", "url": "https://your.host/webhooks"}`. One subscription per event type per partner: a 409 means it already exists — update the URL with `PATCH /webhooks/subscriptions/{eventType}` (`updateSubscription`) instead.
3. **Submit the batch** — `POST /estimations/pre-offers/batch` (`handlePreOfferEstimationBatch`) with `estimationRequests[]`, each shaped exactly like a single `generatePreOfferEstimation` request. The response returns a `batchId` (UUID); processing is asynchronous.
4. **On webhook delivery** — when `ESTIMATION_BATCH_COMPLETED` fires, list results per applicant with `GET /applicants/{partnerApplicantId}/estimations/pre-offers` (`getPreOfferEstimations`); filter by `status`, `productType` (`MCA | TERM_LOAN | LOC`), `stage` (`ACTIVE | EXPIRED | SUPERSEDED | CONVERTED`), or `createdDateFrom`/`createdDateTo`. Pagination is zero-based `page`/`size` (default size 50, sort `createdAt,DESC`).

## Rules

- Also consider subscribing to `ESTIMATION_EXPIRED` to refresh stale Signals.
- Poll `getPreOfferEstimations` only as a fallback — the webhook is the intended completion signal.
- Errors: `{message, code, details[]}` envelope (see `errors/uncapped-problem-types.yml`); webhook catalog: `asyncapi/uncapped-webhooks.yml`.
