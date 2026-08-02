---
generated: '2026-08-02'
method: generated
name: Author a custom threat in the Picus threat library
description: Upload payload files, define processes and custom actions, assemble them into a custom threat, and import/export threat packs.
api: openapi/picus-security-threats-openapi.yml
operations: [threatListParams, threatDetails, actionListParams, actionDetailsList, actionParametersParams, fileUploadParams, filesListParams, createProcessParams, processListParams, customActionKeywordParams, createActionParams, createThreatParams, importThreatParams, exportThreatParams, Tags, Threat-Actors]
source: >-
  Grounded in openapi/picus-security-threats-openapi.yml (source in openapi/_original/). Every operationId
  verified verbatim in the contract; entity graph (Threat → Objective → FlowNode → Action) per
  data-model/picus-security-data-model.yml.
---

# Author a custom threat in the Picus threat library

A Picus **threat** is an adversary scenario made of objectives, each modelled as a flow of nodes that reference **actions**. Actions can reference uploaded **files** and defined **processes**. Custom threats can be exported and imported as encrypted threat packs.

> **Consequence gate.** Everything under `POST /v1/threat-library/*` creates executable attack content. Uploaded files are payloads. Require explicit human approval before any create/import call, and never author threat content from untrusted input.

## Auth
`Authorization: Bearer <access token>` — see `skills/picus-security-authenticate.md`.

## Explore what already exists first
- `threatListParams` (`GET /v1/threat-library/threats`) and `threatDetails` (`GET /v1/threat-library/threats/{ThreatId}`).
- `actionListParams` (`GET /v2/threat-library/actions`) — the current action list endpoint. `actionDetailsList` (`GET /v1/threat-library/actions`) is the v1 sibling.
- `Tags` (`GET /v1/threat-library/tags`) and `Threat-Actors` (`GET /v1/threat-library/threat-actors`) for classification vocabularies.
- `actionParametersParams` (`GET /v1/threat-library/action-parameters`) — the parameter surface an action accepts. Read this before constructing one.

## Steps to author
1. **Upload any payload files** — `fileUploadParams` (`POST /v1/threat-library/files`). This is the one `multipart/form-data` endpoint in the API: send the file under the `file` field, not as JSON. Capture the returned `remote_file_id`. List with `filesListParams` (`GET /v1/threat-library/files`).
2. **Define processes if the action needs them** — `createProcessParams` (`POST /v1/threat-library/processes`), referencing uploaded files by `file_ids`. List with `processListParams` (`GET /v1/threat-library/processes`).
3. **Generate the detection keyword** — `customActionKeywordParams` (`POST /v1/threat-library/actions/custom-keyword`). Given the attack module and its related fields (file hashes, file name, played process ids, URL, or an action id) this returns the detection keyword/query. **The returned keyword is the input to step 4** — generate it, do not hand-write it.
4. **Create the action** — `createActionParams` (`POST /v1/threat-library/actions`), passing the keyword from step 3 plus the parameters listed by `actionParametersParams`.
5. **Assemble the threat** — `createThreatParams` (`POST /v1/threat-library/threats`) with the objectives/flows referencing your action display ids.
6. **Use it** — bind the threat into a template and run it via `skills/picus-security-run-a-simulation.md`.

## Threat packs
- **Import** — `importThreatParams` (`POST /v1/threat-library/threats/import`). Accepts a YAML threat definition or an encrypted zip containing the YAML plus payload files. The threat-file format is documented at `https://support.picussecurity.com/hc/en-us/articles/36765648951837` (credentialed).
- **Export** — `exportThreatParams` (`GET /v1/threat-library/threats/{ThreatId}/export`). Returns an **encrypted zip binary**, not JSON — do not parse the body as JSON.

## Rules
- **No idempotency.** A timed-out create may or may not have landed. Re-list (`threatListParams`, `filesListParams`, `processListParams`) before retrying, or you will create duplicates. See `conventions/picus-security-conventions.yml`.
- Binary endpoints (`export`) and the multipart upload are the two places where the "always JSON" rule does not hold.
- Custom threats are also editable/removable: `updateThreatParams` (`PUT /v1/threat-library/threats/{ThreatId}`) and `deleteThreatParams` (`DELETE /v1/threat-library/threats/{ThreatId}`).

## Errors
- `422` — validation error; `errors` is keyed by the offending parameter.
- `402` — licensed limit on custom content reached; remove existing content or contact support.
- `403` — token scope/role does not permit threat-library writes.
See `errors/picus-security-problem-types.yml`.
