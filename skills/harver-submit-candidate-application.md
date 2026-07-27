---
name: Submit a candidate and create an application
description: Invite a candidate into a Harver vacancy by creating a Candidate + Application and returning a magic-link.
api: openapi/harver-openapi-original.json
operations:
- POST /oauth/token
- POST /vacancies/{vacancyId}/applications
---

# Submit a candidate and create an application

Use this to push a candidate from your ATS into a Harver vacancy so they can start the Harver Journey.

## Steps

1. **Authenticate.** `POST /oauth/token` on the environment host
   (`https://api.harver-test.com` for test, `https://api.harver.com` for prod)
   with `Content-Type: application/x-www-form-urlencoded` and body
   `grant_type=client_credentials`, `client_id`, `client_secret` (all provided
   by Harver). Keep the returned `access_token` (valid 1 hour) and send it as
   `Authorization: Bearer {access_token}` on every following call.
2. **Create the application.** `POST /vacancies/{vacancyId}/applications` with a
   JSON:API body: `data.type = "applications"`, optional `data.attributes`
   key/value pairs (your ATS ids, `return_url`, etc.), and
   `data.relationships.candidate` carrying the candidate's `emailAddress`,
   `firstName`, `lastName` (all mandatory).
3. **Use the response.** The response returns `applicationId` and `magicLink`.
   Email the magic-link to the candidate or redirect them to it (valid 1 hour).
   Re-posting the same email returns the same `applicationId` with a renewed link.

## Rules

- Custom `attributes` are retrievable later via the `ats` include on
  `GET /applications/{applicationId}`.
- Handle `400` (validation), `401` (expired token — re-auth), `429` (rate limit;
  back off using `Ratelimit-Reset`). Errors use `{statusCode,status,code,message,name}`.
- No idempotency-key header exists; idempotency is by candidate email natural key.
