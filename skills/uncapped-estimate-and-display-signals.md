---
name: Generate a pre-offer estimation and display funding Signals
description: Authenticate as a partner, submit applicant performance data for a pre-offer estimation, and fetch display-ready Signals for the applicant.
api: openapi/uncapped-partners-openapi-original.json
operations: [issueAccessToken, generatePreOfferEstimation, getSignals]
generated: '2026-07-21'
method: generated
---

# Generate a pre-offer estimation and display funding Signals

Use the sandbox server `https://dev.weareuncapped.com/api/partners` while testing; production is `https://portal.weareuncapped.com/api/partners` (server-side only — its CORS allowlist blocks browsers by design).

## Steps

1. **Authenticate** — `POST /partner/authentication/token` (`issueAccessToken`) with body `{"token": "<signed partner JWT containing your client id>"}`. The response `accessToken` goes in `Authorization: Bearer {accessToken}` on every subsequent call. Tokens are short-lived; re-issue on 401 `AUTHENTICATION_REQUIRED`.
2. **Submit performance data** — `POST /estimations/pre-offers` (`generatePreOfferEstimation`) with `partnerApplicantId` (required) plus `applicantData` (legalStructure `LLC | Sole Proprietorship | Unknown`, ISO 3166-1 alpha-3 `country`, ISO 3166-2 `state`) and `applicantPerformance.revenueFigures[]` grouped by `source` (`Amazon | Shopify | Walmart | Unknown`), each figure `{date, amount, currency}` — ISO 8601 dates, ISO 4217 currencies. Response currency is determined from the country code.
3. **Fetch Signals** — `GET /applicants/{partnerApplicantId}/signals` (`getSignals`) with required query `partnerApplicantUserId`, optional `locale` (default `en_US`) and `highlights=true` for highlight blocks. Render each Signal's `title`, `description`, `highlights[]`, and `action[]` (text + URL) in your UI; `productType` is `MCA | TERM_LOAN | LOC`.

## Rules

- Errors arrive as `{message, code, details[{code, message, fieldName}]}` — surface `details[].fieldName` for 400 validation failures (see `errors/uncapped-problem-types.yml`).
- No idempotency-key mechanism exists: do not blind-retry `POST /estimations/pre-offers` on timeouts without checking `GET /applicants/{partnerApplicantId}/estimations/pre-offers` (`getPreOfferEstimations`) first.
- Conventions detail: `conventions/uncapped-conventions.yml`.
