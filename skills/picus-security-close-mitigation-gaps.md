---
generated: '2026-08-02'
method: generated
name: Close prevention gaps with vendor mitigation signatures
description: Find the actions a simulation did not block, pull the vendor-specific prevention signatures Picus recommends, and score each mitigation device.
api: openapi/picus-security-mitigation-openapi.yml
operations: [deviceListParamsV2, deviceStatsByIdParams, signatureListParams, genericNotBlockedActionsParams, latestRunParams, agentParams]
source: >-
  Grounded in openapi/picus-security-mitigation-openapi.yml, openapi/picus-security-agents-openapi.yml and
  openapi/picus-security-simulation-latest-result-openapi.yml (source in openapi/_original/). Every
  operationId verified verbatim; the v1→v2 device migration is recorded in
  lifecycle/picus-security-lifecycle.yml.
---

# Close prevention gaps with vendor mitigation signatures

After a simulation runs, Picus tells you which attack actions were **not blocked** and which vendor signature would have blocked them. This is the remediation loop.

## Auth
`Authorization: Bearer <access token>` — see `skills/picus-security-authenticate.md`.

## Steps
1. **Get the device list** — `deviceListParamsV2` (`GET /v2/mitigation/devices`). Returns the mitigation devices (firewall / IPS / WAF) attached to the account.
2. **Score each device** — `deviceStatsByIdParams` (`GET /v2/mitigation/devices/{DeviceId}`). Returns Blocked Count, Not Blocked Count, Total Count and Score for that device. This is a **two-call workflow by design** — list first, then fetch stats per device you care about.
3. **Pull recommended signatures** — `signatureListParams` (`GET /v1/mitigation/devices/{DeviceId}/signatures`). Returns signature id, name, vendor severity, category, version, description, reference, product platform and product version — i.e. what to deploy on that device.
4. **Cover the generic gaps** — `genericNotBlockedActionsParams` (`GET /v1/mitigation/generic/not-blocked-actions`). Lists not-blocked actions for generic mitigations with their associated signatures and recommendations, including action detail, attack module, category and mitigation info.
5. **Re-validate** — deploy the signatures, then re-run the simulation (`skills/picus-security-run-a-simulation.md`) and compare the prevention score from `latestRunParams`. That before/after delta is the point of the whole exercise.

## Deprecation you must respect
`deviceListParams` (`GET /v1/mitigation/devices`) is **deprecated and no longer available** — it returns `410 Gone`, the only 410 in the contract. Its combined list+stats response was split into the two v2 calls in steps 1 and 2. Never write new code against the v1 path. See `lifecycle/picus-security-lifecycle.yml`.

## Rules
- Devices are also visible per agent: `agentParams` (`GET /v1/agents/{Id}`) returns the agent's mitigation devices and attack modules — useful for attributing a gap to a specific deployment.
- All operations here are read-only and safe to run unattended within rate limits.
- Signature listings are large; page with `limit`/`offset` and respect the endpoint's max.

## Errors
- `410` — you called the deprecated v1 device endpoint. Migrate to v2.
- `404` — unknown `DeviceId`.
- `422` — limit above the endpoint cap.
See `errors/picus-security-problem-types.yml`.
