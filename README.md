# Picus Security

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
