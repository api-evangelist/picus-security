---
generated: '2026-08-02'
method: generated
name: Read simulation results with ATT&CK and detection evidence
description: Walk a Picus simulation run from summary scores down to per-action attack detail and the SIEM/EDR logs and alerts that prove detection.
api: openapi/picus-security-simulation-latest-result-openapi.yml
operations: [latestRunParams, latestRunDetailParams, latestRunThreatParams, latestActionDetailsListParams, latestActionDetailsParams, latestActionAlertsParams, latestActionLogsParams, simulationRunParams, runDetailParams, runThreatParams, actionDetailsListParams, actionDetailsParams, rawLogsParams, integrationList]
source: >-
  Grounded in openapi/picus-security-simulation-latest-result-openapi.yml,
  openapi/picus-security-simulation-result-openapi.yml and openapi/picus-security-integrations-openapi.yml
  (source in openapi/_original/). Every operationId verified verbatim; entity graph per
  data-model/picus-security-data-model.yml; pagination caps per conventions/picus-security-conventions.yml.
---

# Read simulation results with ATT&CK and detection evidence

Picus scores every run on two axes — **prevention** (blocked / not blocked, objectives achieved / unachieved) and **detection** (logged / not logged, alerted / not alerted) — and maps everything to MITRE ATT&CK and the Unified Kill Chain. All of this is read-only.

## The two parallel families
The contract exposes the same drill-down twice:
- **Latest run** — `/v1/simulations/{Id}/run/latest/...` (no `RunId` needed).
- **A specific run** — `/v1/simulations/{Id}/run/{RunId}/...`.
Use the latest family for monitoring, the `{RunId}` family for history and diffing.

## Auth
`Authorization: Bearer <access token>` — see `skills/picus-security-authenticate.md`.

## Steps (latest run)
1. **Run summary** — `latestRunParams` (`GET /v1/simulations/{Id}/run/latest`). Prevention: total threats, blocked/not-blocked, attacker objectives achieved/unachieved. Detection: logged/not-logged, alerted/not-alerted.
2. **Framework mapping** — `latestRunDetailParams` (`GET /v1/simulations/{Id}/run/latest/frameworks`). Returns the run mapped to MITRE ATT&CK (tactic / technique / sub-technique) and the Unified Kill Chain.
3. **Threats, objectives, actions** — `latestRunThreatParams` (`GET /v1/simulations/{Id}/run/latest/threats`). Default `limit` 10, `offset` 0, **max limit 50**. Capture each `ThreatId`.
4. **Actions inside a threat** — `latestActionDetailsListParams` (`GET .../threats/{ThreatId}/actions`). Attack start/end time, log/alert time, and attack-module detail (payloads, terminal log, file name, sha256, md5, sha1). Protocol-based results included.
5. **One action in full** — `latestActionDetailsParams` (`GET .../threats/{ThreatId}/actions/{ActionId}`).
6. **Detection evidence** — first `integrationList` (`GET /v1/integrations`) to get an `IntegrationId`, then:
   - `latestActionAlertsParams` (`GET .../actions/{ActionId}/integrations/{IntegrationId}/alerts`)
   - `latestActionLogsParams` (`GET .../actions/{ActionId}/integrations/{IntegrationId}/logs`)
   Both: default `limit` 100, `offset` 0, **max limit 1000**.

## Steps (a specific run)
Same shape with `{RunId}`: `simulationRunParams` → `runDetailParams` → `runThreatParams` → `actionDetailsListParams` → `actionDetailsParams`. Raw log files for a peer are at `rawLogsParams` (`GET /v1/simulations/{SimulationId}/run/{RunId}/threats/{ThreatId}/peers/{PeerId}/integrations/{IntegrationId}/raw-logs`) — note this one path uses `{SimulationId}`, not `{Id}`.

## Rules
- **`node_id` disambiguates repeated actions.** The same action can appear several times inside one threat; the alert and log endpoints accept an optional `node_id` query parameter and **default to the first node when it is omitted**. If you are reconciling counts, pass `node_id` explicitly or you will under-count.
- Actions carry two identifiers: the internal `action_id` and the customer-facing `action_display_id`/`display_id`. Report the display id to humans; use the internal id in paths.
- Everything here is read-only and safe for an agent to call unattended, subject to rate limits.
- Exceeding a `limit` cap returns `422` naming the bound.

## Errors
- `422` — limit above the endpoint cap, or an unknown id in the path.
- `404` — no such simulation, run, threat, action or integration.
- `429` — throttled; back off until `X-Ratelimit-Reset`.
See `errors/picus-security-problem-types.yml` and `rate-limits/picus-security-rate-limits.yml`.
