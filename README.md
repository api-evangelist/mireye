# Mireye

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Mireye is infrastructure for physical-world AI agents, turning any US latitude and longitude into sourced, structured geospatial ground truth. Its Mireye Earth API exposes two HTTP endpoints — POST /v1/fetch for deterministic per-field values and POST /v1/ask for natural-language, citation-backed Q&A — plus GET /v1/meta/fields for a self-describing catalog of 175+ fields across seven layers (terrain, land cover, built environment, utilities and energy, climate and resource, hazards, and parcels and boundaries) and 14 preset bundles. Every value carries provenance: the source identifier, upstream URL, dataset vintage, timestamp, and a confidence rating, sourced from federal agencies (USGS, NOAA, USDA, USFS, FEMA, EPA, EIA, NREL, US Census) and open and commercial datasets (Sentinel-2, Overture Maps, Regrid). Mireye also runs a hosted OAuth 2.1 MCP server (com.mireye/earth) and a local stdio adapter so agents in Claude, Cursor, and custom loops can query the earth as easily as the web. Founded 2026 by Ansh Chokshi and Shashwat Kapoor; Y Combinator S26.

**Areas:** Company · Geospatial · Geographic Information System · Location · AI Agents · MCP · Government Data · Risk Management · Insurance · Data

## Mireye on APIs.io

Everything profiled for Mireye is published on APIs.io rather than in this repository. Start at the provider page — **[apis.io/providers/mireye/](https://apis.io/providers/mireye/)** — or go straight to one of the APIs or properties below.

### APIs (17)

- [Mireye Ask API](https://apis.io/apis/mireye/mireye-ask-api/) — The Ask API from Mireye — 2 operation(s) for ask.
- [Mireye Ask Site API](https://apis.io/apis/mireye/mireye-ask-site-api/) — The Ask Site API from Mireye — 1 operation(s) for ask site.
- [Mireye Auth API](https://apis.io/apis/mireye/mireye-auth-api/) — The Auth API from Mireye — 1 operation(s) for auth.
- [Mireye Authorize API](https://apis.io/apis/mireye/mireye-authorize-api/) — The Authorize API from Mireye — 1 operation(s) for authorize.
- [Mireye Feature Requests API](https://apis.io/apis/mireye/mireye-feature-requests-api/) — The Feature Requests API from Mireye — 1 operation(s) for feature requests.
- [Mireye Fetch API](https://apis.io/apis/mireye/mireye-fetch-api/) — The Fetch API from Mireye — 1 operation(s) for fetch.
- [Mireye Healthz API](https://apis.io/apis/mireye/mireye-healthz-api/) — The Healthz API from Mireye — 1 operation(s) for healthz.
- [Mireye Mcp API](https://apis.io/apis/mireye/mireye-mcp-api/) — The Mcp API from Mireye — 3 operation(s) for mcp.
- [Mireye Meta API](https://apis.io/apis/mireye/mireye-meta-api/) — The Meta API from Mireye — 1 operation(s) for meta.
- [Mireye Readyz API](https://apis.io/apis/mireye/mireye-readyz-api/) — The Readyz API from Mireye — 1 operation(s) for readyz.
- [Mireye Register API](https://apis.io/apis/mireye/mireye-register-api/) — The Register API from Mireye — 1 operation(s) for register.
- [Mireye Revoke API](https://apis.io/apis/mireye/mireye-revoke-api/) — The Revoke API from Mireye — 1 operation(s) for revoke.
- [Mireye Sites API](https://apis.io/apis/mireye/mireye-sites-api/) — The Sites API from Mireye — 2 operation(s) for sites.
- [Mireye Token API](https://apis.io/apis/mireye/mireye-token-api/) — The Token API from Mireye — 1 operation(s) for token.
- [Mireye Users API](https://apis.io/apis/mireye/mireye-users-api/) — The Users API from Mireye — 5 operation(s) for users.
- [Mireye .well Known API](https://apis.io/apis/mireye/mireye-well-known-api/) — The .well Known API from Mireye — 1 operation(s) for .well known.
- [Mireye OAUTH API](https://apis.io/apis/mireye/mireye-oauth-api/) — The Oauth API from Mireye — 4 operation(s) for oauth.

### Published properties

Each property below is a page on APIs.io carrying the machine-readable artifact and what it says.

#### MCP servers (1)

Model Context Protocol servers that expose the API to agents.

- [com.mireye/earth](https://apis.io/servers/mireye/com-mireye-earth/)

#### Agentic access (1)

Which operations an agent can call, which act, and which need a human in the loop.

- [Mireye Agentic Access](https://apis.io/agentic-access/mireye/mireye-agentic-access/)

#### Collections (17)

Postman and Open Collection request collections.

- [Mireye Earth Ask API](https://apis.io/collections/mireye/open-mireye-ask-api/)
- [Mireye Earth Ask Ask Site API](https://apis.io/collections/mireye/open-mireye-ask-site-api/)
- [Mireye Earth Ask Auth API](https://apis.io/collections/mireye/open-mireye-auth-api/)
- [Mireye Earth Ask Authorize API](https://apis.io/collections/mireye/open-mireye-authorize-api/)
- [Mireye Earth Ask Feature Requests API](https://apis.io/collections/mireye/open-mireye-feature-requests-api/)
- [Mireye Earth Ask Fetch API](https://apis.io/collections/mireye/open-mireye-fetch-api/)
- [Mireye Earth Ask Healthz API](https://apis.io/collections/mireye/open-mireye-healthz-api/)
- [Mireye Earth Ask Mcp API](https://apis.io/collections/mireye/open-mireye-mcp-api/)
- [Mireye Earth Ask Meta API](https://apis.io/collections/mireye/open-mireye-meta-api/)
- [Mireye Earth Ask Oauth API](https://apis.io/collections/mireye/open-mireye-oauth-api/)
- [Mireye Earth Ask Readyz API](https://apis.io/collections/mireye/open-mireye-readyz-api/)
- [Mireye Earth Ask Register API](https://apis.io/collections/mireye/open-mireye-register-api/)
- [Mireye Earth Ask Revoke API](https://apis.io/collections/mireye/open-mireye-revoke-api/)
- [Mireye Earth Ask Sites API](https://apis.io/collections/mireye/open-mireye-sites-api/)
- [Mireye Earth Ask Token API](https://apis.io/collections/mireye/open-mireye-token-api/)
- [Mireye Earth Ask Users API](https://apis.io/collections/mireye/open-mireye-users-api/)
- [Mireye Earth Ask .well Known API](https://apis.io/collections/mireye/open-mireye-well-known-api/)

#### Overlays (1)

OpenAPI overlays, including conversational phrasing for each operation.

- [Mireye Openapi Overlay](https://apis.io/overlays/mireye/mireye-openapi-overlay/)

#### Security (2)

Authentication, trust centre, and vulnerability disclosure.

- [Mireye Authentication](https://apis.io/security/mireye/mireye-authentication/)
- [Mireye Domain Security](https://apis.io/security/mireye/mireye-domain-security/)

#### OAuth scopes (1)

The permissions the API asks for.

- [Mireye Scopes](https://apis.io/scopes/mireye/mireye-scopes/)

## Where to find this API

Links to the company's own pages. This repository publishes no copy of their specifications — the canonical document is theirs and lives at the URLs below.

- [Website](https://mireye.ai)
- [Documentation](https://docs.mireye.ai)
- [Getting started](https://docs.mireye.ai/quickstart)
- [API reference](https://docs.mireye.ai/api-reference)
- [Pricing](https://www.mireye.com/pricing)
- [Sign up](https://www.mireye.com)
- [Support](https://docs.mireye.ai/mcp/troubleshooting)
- [Blog](https://www.mireye.com/blog)
- [Terms of service](https://www.mireye.com/terms)
- [Privacy policy](https://www.mireye.com/privacy)
- [GitHub](https://github.com/Mireye-Labs)
- [Status](https://status.mireye.com)

## API Evangelist rating

**Kin Score 45.1** — developing · rubric 0.25.0 · scored 2026-10-23

| Facet | Score |
|---|---|
| Discoverability | 75.0 |
| Contract quality | 35.4 |
| Contract governance | 4.5 |
| Operational transparency | 10.5 |
| Developer ergonomics | 71.4 |
| Access clarity | 34.2 |

**Agent readiness 52.5** — agent-ready (gated down from agent-native)

| Agent-readiness dimension | Result |
|---|---|
| Spec Presence | yes |
| Agentic Access | derived (partial credit) |
| Reversibility Documented | — |
| MCP Server | verified |
| Auth Clarity | served (partial credit) |
| Idempotency | — |
| Error Semantics | verified |
| OpenAPI Examples | — |
| Rate Limit Signal | — |
| Event Surface Described | — |
| Agent Skills | yes |
| Well Known Catalog | — |
| Consent Identity | — |
| Agent Card | — |
| Dry Run Mode | — |
| Delegated Identity | served (partial credit) |
| Protected Resource Metadata | verified |
| Dynamic Client Registration | yes |
| Agentic Commerce | — |
| Pagination Ratio | na |
| Operation Distinctiveness | distinct |

**Regulatory alignment 33.3** — Insurance

The rubric is published at [github.com/api-evangelist/kin-score](https://github.com/api-evangelist/kin-score). The per-check breakdown behind these numbers — which checks were earned, partial or missed — along with score history and how this provider compares to its peers, is available at [apis.io/providers/mireye/](https://apis.io/providers/mireye/).

## How this profile is made

API Evangelist profiles a company's public API surface the way a developer would find it: the website, developer portal and documentation, the specifications it publishes, its public repositories, and its pricing, status and changelog pages. From that it derives the properties above — contracts split per API, workflows, schemas, collections, governance rules, plans and rate limits — and scores them against the published [Kin Score rubric](https://github.com/api-evangelist/kin-score).

The artifacts are published on [APIs.io](https://apis.io), not in this repository. This repository is the public record for Mireye: the place to say something is wrong, missing or out of date, and to ask for a re-score or a removal.

## Open an issue

- **Something about this listing** — a wrong description, a missing API, a stale artifact or a question about the rating: [open an issue on this repository](https://github.com/api-evangelist/mireye/issues/new).
- **A new API, or one that is not in the catalog yet** — [submit it on APIs.io](https://apis.io/add/).
- **Anything else** — [the APIs.io inbox](https://github.com/api-search/inbox/issues/new) or [info@apievangelist.com](mailto:info@apievangelist.com).
