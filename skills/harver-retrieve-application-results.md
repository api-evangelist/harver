---
name: Retrieve an application's results and reports
description: Fetch a completed candidate's matching score, module results and report links from Harver.
api: openapi/harver-openapi-original.json
operations:
- POST /oauth/token
- GET /applications/{applicationId}
- GET /applications/{applicationId}/reports/reports-hub
---

# Retrieve an application's results and reports

Use this once a candidate has completed the Harver Journey (status `new`) to pull
scores, module results and report links.

## Steps

1. **Authenticate.** `POST /oauth/token` (client_credentials) and send the
   Bearer token on all calls.
2. **Get the application with includes.** `GET /applications/{applicationId}?include=report,matching-results,personal-info,additional-info,ats,matching-indicators`.
   The base payload always carries `status`, `progress`, `matchingScore` (0–100)
   and `matchingProfile.label` (configured score band).
3. **Read the modules you need:**
   - `matching-results` / `matching-indicators` → per-module and KMI scores.
   - `report` → PDF report + fact-sheet links (**expire in 15 minutes**) and a
     non-expiring Candidate Detail Page link (requires a Harver account / SSO).
   - `ats` → the key/value attributes you passed at creation.
4. **Reports Hub (optional).** `GET /applications/{applicationId}/reports/reports-hub`
   returns a Reports Hub magic link.

## Rules

- Report/fact-sheet links are short-lived (15 min) — fetch them just-in-time.
- `401` → token expired, re-auth. `404` → check the applicationId.
- Only request the `include` modules configured for the account.
