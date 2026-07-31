---
name: Assess flood risk for a US coordinate
description: Determine whether a US location is at flood risk using Mireye Earth's flood_risk preset and a cited natural-language answer, with FEMA/federal provenance on every value.
api: openapi/mireye-openapi-original.json
operations: [meta_fields_v1_meta_fields_get, fetch_v1_fetch_post, ask_v1_ask_post]
generated: '2026-07-20'
method: generated
source: derived from openapi/mireye-openapi-original.json + https://docs.mireye.ai
---

# Assess flood risk for a US coordinate

Use Mireye Earth to answer "is this location at flood risk?" with authoritative,
citation-backed federal data. All operations are read-only and idempotent.

## Auth

Send `Authorization: Bearer <MIREYE_API_TOKEN>` on every data call (a dashboard-minted
JWT). `GET /v1/meta/fields` is public and needs no token. See
`authentication/mireye-authentication.yml`.

## Steps

1. **(Optional) Discover the catalog** — call `meta_fields_v1_meta_fields_get`
   (`GET /v1/meta/fields`, no auth) once to confirm the `flood_risk` preset and the
   `us_envelope` bounds. It is ETag-cached; re-fetch with `If-None-Match` is free.

2. **Validate the coordinate** — ensure `lat ∈ [18, 72]` and `lng ∈ [-180, -65]`.
   Out-of-bounds calls return `400 coord_out_of_bounds`.

3. **Fetch the deterministic flood fields** — call `fetch_v1_fetch_post`
   (`POST /v1/fetch`) with `{"lat": <lat>, "lng": <lng>, "preset": "flood_risk"}`.
   Each returned field carries `value`, `unit`, `source`, `source_url`, `confidence`,
   `dataset_vintage`, and `ttl_seconds`. Keep `within_floodplain_polygon`,
   `elevation`, and `coast_distance_m` for the verdict. A 200 response with a populated
   `partial_failures[]` array is normal — report which sources were degraded rather than
   failing the whole check.

4. **Get a cited prose verdict** — call `ask_v1_ask_post` (`POST /v1/ask`) with
   `{"lat": <lat>, "lng": <lng>, "question": "Is this property at flood risk?"}`.
   Use `citations[]` as the audit trail and `fields_used[]` to re-verify any claim
   deterministically via step 3. A `confidence: "low"` answer is still a 200 — surface the
   confidence bucket to the user.

## Error handling

Retry only when the error body's `retryable` is true (e.g. `ask_upstream_rate_limited`
429, `ask_upstream_unreachable` 502) with backoff. Refresh the catalog on `fields_unknown`.
See `errors/mireye-error-codes.yml`. Correlate failures by sending your own `X-Request-ID`.
