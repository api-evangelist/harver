---
name: Poll a vacancy for newly completed candidates
description: Periodically pull candidates who completed the Harver Journey in a vacancy using status + since filters.
api: openapi/harver-openapi-original.json
operations:
- POST /oauth/token
- GET /vacancies/{vacancyId}/candidates
- GET /applications/{applicationId}
---

# Poll a vacancy for newly completed candidates

Use this to keep your ATS in sync with candidates who just finished assessments.

## Steps

1. **Authenticate.** `POST /oauth/token` (client_credentials); send the Bearer token.
2. **List new candidates.** `GET /vacancies/{vacancyId}/candidates?filter[status]=new&filter[status-updated-at][since]={epoch}`.
   Use the epoch timestamp of your previous poll as `[since]` to fetch only
   newly-updated candidates. Statuses: `in-progress`, `new`, `hired`, `rejected`
   — filter for `new` (assessment completed).
3. **Follow the relationship.** Each candidate lists its application under
   `relationships`; take that application id and call
   `GET /applications/{applicationId}` (with includes) to pull results.
4. **Advance the cursor.** Store the max `status-updated-at` you saw as the next
   `[since]` value.

## Rules

- Combine filters: `filter[locations]`, `filter[region]`, `filter[job_function]`,
  `filter[external_location_id]` narrow the set.
- Respect rate limits (429 + `Ratelimit-Reset`); poll on an interval, not tightly.
- `401` → re-auth (tokens last 1 hour).
