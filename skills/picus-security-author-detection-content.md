---
generated: '2026-08-02'
method: generated
name: Author custom detection content for a SIEM or EDR
description: Browse Picus detection content sources and rules, then create custom detection content bound to real actions and MITRE ATT&CK techniques.
api: openapi/picus-security-mitigation-openapi.yml
operations: [contentSources, contentSourceNameUriParam, detectionContentRulesParams, contentSourceNameAndRuleIdUriParam, getDetectionDevicesParams, getMitreTacticsParams, getMitreTechniquesParams, getMitreSubTechniquesParams, getCustomDetectionContentParams, createCustomDetectionContentParams, getCustomDetectionContentByIdParams, updateCustomDetectionContentParams, deleteCustomDetectionContentParams]
source: >-
  Grounded in openapi/picus-security-mitigation-openapi.yml (source in openapi/_original/). Every operationId
  verified verbatim in the contract; entity graph (DetectionContentSource → LogSource → DetectionRule) per
  data-model/picus-security-data-model.yml.
---

# Author custom detection content for a SIEM or EDR

Picus ships validated detection rules per content source (Splunk, QRadar, CrowdStrike and others) and lets you author your own, bound to the actions they should catch and the MITRE ATT&CK techniques they cover.

## Auth
`Authorization: Bearer <access token>` — see `skills/picus-security-authenticate.md`.

## Explore the existing content
1. **Sources** — `contentSources` (`GET /v1/mitigation/detection-content/sources`) returns the detection content sources on the account, with rule counts.
2. **Log sources** — `contentSourceNameUriParam` (`GET /v1/mitigation/detection-content/{source}/log-sources`) for one source.
3. **Rules** — `detectionContentRulesParams` (`GET /v1/mitigation/detection-content/{source}/rules`). Note this endpoint uses **capitalised** `Limit`/`Offset` query parameters (plus `Search`, `RuleSeverity`, `OrderBy`, `IsAscending`) — an inconsistency with the rest of the API, which uses lowercase `limit`/`offset`. See `conventions/picus-security-conventions.yml`.
4. **One rule in full** — `contentSourceNameAndRuleIdUriParam` (`GET /v1/mitigation/detection-content/{source}/rules/{ruleId}`) returns the rule with its mapped actions and MITRE techniques, plus integration configuration.

## Steps to author custom content
1. **Get the device vocabulary** — `getDetectionDevicesParams` (`GET /v1/mitigation/detection-content/devices`) returns the available detection content devices (e.g. Splunk, QRadar). This exists specifically to populate the device selection.
2. **Get the ATT&CK vocabulary** — `getMitreTacticsParams` (`/mitre/tactics`), `getMitreTechniquesParams` (`/mitre/techniques`), `getMitreSubTechniquesParams` (`/mitre/sub-techniques`). Sub-techniques carry their parent technique reference. Use the returned identifiers verbatim — do not construct ATT&CK ids by hand.
3. **Pick the actions to detect** — from `skills/picus-security-author-a-custom-threat.md` or from the not-blocked actions in `skills/picus-security-close-mitigation-gaps.md`. Custom content is bound by `action_ids`.
4. **Create the content** — `createCustomDetectionContentParams` (`POST /v1/mitigation/detection-content/custom`) with the selected device/source, `action_ids` and technique selections. Capture the returned `content_id`.
5. **Manage it** — `getCustomDetectionContentParams` (`GET .../custom`) to list, `getCustomDetectionContentByIdParams` (`GET .../custom/{content_id}`) to read, `updateCustomDetectionContentParams` (`PUT .../custom/{content_id}`) to edit, `deleteCustomDetectionContentParams` (`DELETE .../custom/{content_id}`) to remove.
6. **Validate it** — re-run the simulation containing those actions and read the detection side of the result (`latestRunParams` / `latestActionAlertsParams`) to confirm the content now logs and alerts.

## Rules
- Always populate device, tactic, technique and sub-technique selections from the enumeration endpoints in steps 1–2. They exist to be the source of truth; hand-typed values will fail validation.
- **No idempotency.** Re-list with `getCustomDetectionContentParams` before retrying a timed-out create.
- Detection content is vendor-shaped (the contract carries CrowdStrike IOA rule structures, Splunk/QRadar log-source shapes), not vendor-neutral Sigma. See `conformance/picus-security-conformance.yml`.

## Errors
- `422` — validation error naming the offending field (commonly an unknown technique or action id).
- `404` — unknown `source`, `ruleId` or `content_id`.
- `403` — token scope/role does not permit detection-content writes.
See `errors/picus-security-problem-types.yml`.
