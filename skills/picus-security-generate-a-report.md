---
generated: '2026-08-02'
method: generated
name: Generate and download a simulation result report
description: Kick off a Picus simulation result report, poll it, and fetch it either by signed URL or as a direct binary download.
api: openapi/picus-security-simulation-result-reports-openapi.yml
operations: [generateSimulationReportParams, reportParams, reportDownloadUrlParams, reportDownloadParams, summaryOverallParams, activityLogsFilterParams]
source: >-
  Grounded in openapi/picus-security-simulation-result-reports-openapi.yml,
  openapi/picus-security-summary-openapi.yml and openapi/picus-security-activity-logs-openapi.yml
  (source in openapi/_original/). Every operationId verified verbatim in the contract.
---

# Generate and download a simulation result report

Reporting is a v2 surface: you request a report, then retrieve it. The download has two paths depending on whether the deployment can reach S3.

## Auth
`Authorization: Bearer <access token>` — see `skills/picus-security-authenticate.md`.

## Steps
1. **Request the report** — `generateSimulationReportParams` (`POST /v2/simulations/{Id}/results/reports`). The body selects either a simulation **overview** or a specific **run**. Capture the returned `ReportId`.
2. **Poll for readiness** — `reportParams` (`GET /v2/simulations/{Id}/results/reports/{ReportId}`) returns the report's detail/status. Poll on a backoff that respects `X-Ratelimit-Remaining`.
3. **Fetch it — pick the right path:**
   - **SaaS / S3-reachable clients:** `reportDownloadUrlParams` (`GET .../reports/{ReportId}/download-url`) returns a download URL.
   - **On-prem clients that cannot reach S3 directly:** `reportDownloadParams` (`GET .../reports/{ReportId}/download`) streams the file itself — **binary body with a `Content-Disposition` header**, not JSON. Do not parse it as JSON.

## Related reporting reads
- **Account-wide posture** — `summaryOverallParams` (`POST /v1/summary/overall`). Note this is a **POST** even though it is a read; it takes a filter body.
- **Audit trail** — `activityLogsFilterParams` (`GET /v1/activity-logs`), filterable by `date_start` / `date_end`.
- **Exposure scores** — `listInstanceScoresParams` (`POST /v1/exposures/instances/scores`) returns exposure scores for host+exposure pairs (`asset_id` + `cve_id`). Duplicate pairs are deduplicated server-side and **unmatched pairs are silently omitted from the response** — reconcile what you sent against what came back rather than assuming positional alignment.

## Rules
- **No idempotency.** A retried `generateSimulationReportParams` produces a second report. If the call times out, list the simulation's reports (or re-read the known `ReportId`) before retrying.
- Report generation is asynchronous — never assume the response to step 1 contains the file.
- Two of the three read-shaped operations here are `POST` (`summaryOverallParams`, `listInstanceScoresParams`) because they take filter bodies. Do not treat them as idempotent GETs on cache layers.

## Errors
- `422` — validation error on the report body or filter body.
- `404` — unknown simulation `Id` or `ReportId`.
- `429` — throttled; report generation is a heavier endpoint, so expect a lower per-minute limit than reads.
See `errors/picus-security-problem-types.yml` and `rate-limits/picus-security-rate-limits.yml`.
