# Picus Security

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

Picus Security pioneered Breach and Attack Simulation (BAS) in 2013 and now ships the Picus Security Validation Platform for Adversarial Exposure Validation (AEV) and Continuous Threat Exposure Management (CTEM). The platform continuously simulates real-world adversary techniques against network, endpoint, email and cloud controls, scores prevention and detection effectiveness, maps results to MITRE ATT&CK and the Unified Kill Chain, and returns vendor-specific mitigation signatures and validated detection rules.

- Website: https://www.picussecurity.com/
- API documentation: https://apidocs.picussecurity.com/
- API host: https://api.picussecurity.com/
- Status: https://status.picussecurity.com/
- Trust Center: https://www.picussecurity.com/trust-center

## The Picus Customer API

A public REST API (Swagger 2.0, 71 paths / 84 operations) served from `api.picussecurity.com`. The machine-readable contract is published at **https://api.picussecurity.com/swagger.json** — on the API host root, not on the docs host — and is captured verbatim in `openapi/_original/`, then split one-per-tag under `openapi/`.

Authorization is a Picus-proprietary refresh-token exchange: a scoped refresh token (6 months, generated in the console) is exchanged at `POST /v1/auth/token` for a bearer access token (2 hours).

## Artifacts in this repo

| Directory | What it holds |
|---|---|
| `openapi/` | 14 per-tag specs split from the harvested Swagger 2.0 contract (`_original/`) |
| `overlays/` | One OpenAPI Overlay 1.0.0 per spec capturing API Evangelist enhancements |
| `authentication/` | The refresh-token exchange, token TTLs, scoped tokens, role model |
| `conventions/` | Pagination caps, filter grammar, error envelope, versioning, rate-limit signalling — and the finding that **idempotency is not supported** |
| `rate-limits/` | Tiered per-endpoint limits and the `X-Ratelimit-*` headers |
| `errors/` | The three error envelopes plus the published status-code reference |
| `lifecycle/` | Versioning + deprecation policy, the deprecated `/v1/mitigation/devices` operation, status page |
| `changelog/` | The (sparse) ReadMe changelog |
| `data-model/` | 34 entities and 71 relationships derived from the contract |
| `conformance/` | Standards posture, including MITRE ATT&CK / Unified Kill Chain as first-class response structures |
| `security/` | Domain security probe, the Vulnerability Disclosure Program, and the Trust Center certifications |
| `packages/` | The registry sweep — Picus ships **no** first-party SDK, CLI or GitHub org |
| `skills/` | Seven packaged Agent Skills, every step grounded in a verified `operationId` |
| `mcp/` | A **candidate** MCP tool surface derived from the contract — Picus operates no MCP server |
| `agentic-access/` | Recommended `x-agentic-access` execution contracts per operation |
| `llms/` | The provider's own `llms.txt` files, saved verbatim |
| `well-known/` | The `/.well-known/` probe record (all misses) |

## Safety note

Picus is an offensive-security control plane. Twenty-five of its 84 operations create, run, cancel or delete adversary simulations, author executable threat content, upload payload files, or manage users and roles. Any agent integration must gate that write set behind explicit human approval.
