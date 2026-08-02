---
generated: '2026-08-02'
method: generated
name: Create and run a security validation simulation
description: Pick a template and agent, create a Picus simulation, run it on demand, and watch it to completion.
api: openapi/picus-security-simulations-openapi.yml
operations: [templateListParams, Result, createSimulationParams, simulationListParams, simulationParams, simulateNowParams, simulationCancelParams, updateSimulationParams, simulationDeleteParams]
source: >-
  Grounded in openapi/picus-security-simulations-openapi.yml, openapi/picus-security-templates-openapi.yml and
  openapi/picus-security-agents-openapi.yml (source in openapi/_original/). Every operationId verified
  verbatim in the contract; filter grammar and pagination caps per
  conventions/picus-security-conventions.yml.
---

# Create and run a security validation simulation

A Picus simulation is a **template** (which threats to run) bound to an **agent** (where to run them) plus a schedule. Running it executes real adversary techniques against live security controls.

> **Consequence gate.** `createSimulationParams` and `simulateNowParams` are not read operations. They launch attack simulations against production defences. Require explicit human approval before either call, and never run them speculatively or in a retry loop.

## Auth
`Authorization: Bearer <access token>` — see `skills/picus-security-authenticate.md`. Base URL `https://api.picussecurity.com/`.

## Steps
1. **Choose a template** — `templateListParams` (`GET /v1/templates`); fetch one with `templateParams` (`GET /v1/templates/{Id}`) to confirm its threats, rules and supported agent types.
2. **Choose an agent** — `Result` (`GET /v1/agents`) lists agents with name, status, type, platform. Confirm the agent is alive and its type matches the template's `agent_types` before binding. `agentParams` (`GET /v1/agents/{Id}`) gives the agent's mitigation devices and attack modules.
3. **Create the simulation** — `createSimulationParams` (`POST /v1/simulations`) with the template id, agent id, name and schedule in the JSON body. Capture the returned simulation `Id`.
4. **Run it now** — `simulateNowParams` (`POST /v1/simulations/{Id}/simulate-now`) re-runs the simulation immediately.
5. **Poll for completion** — `simulationParams` (`GET /v1/simulations/{Id}`) returns the detail including `status_details` and the `simulation_run` list. Statuses seen in the filter vocabulary: `RUNNING`, `STOPPED`, `WAITING FOR THE FIRST RUN`, `NOT STARTED`, `COMPLETED`. Poll on a backoff that respects `X-Ratelimit-Remaining`.
6. **Read the results** — once `COMPLETED`, hand off to `skills/picus-security-read-simulation-results.md`.

## Managing existing simulations
- **List/filter** — `simulationListParams` (`GET /v1/simulations`). Default `limit` 25, `offset` 0, **max limit 50**. Filters: `status` (comma-separated multi-value), `simulation_name`, `agent_name`, `template_name` (all contains-style free text), `agent_id`, `template_id`, and numeric ranges `prevention_result_gte`/`_lte`, `detection_result_gte`/`_lte`.
- **Update** — `updateSimulationParams` (`PUT /v1/simulations/{Id}`).
- **Cancel a running simulation** — `simulationCancelParams` (`PUT /v1/simulations/{Id}/cancel`). Only meaningful while `RUNNING`.
- **Delete** — `simulationDeleteParams` (`DELETE /v1/simulations/{Id}`). Destructive; results go with it.

## Rules
- **No idempotency.** Picus publishes no `Idempotency-Key` mechanism. If `POST /v1/simulations` or `simulate-now` times out, **do not blind-retry** — call `simulationListParams` (or `simulationParams`) first and confirm whether the create/run already landed. See `conventions/picus-security-conventions.yml`.
- Multi-value filters are comma-separated: `?status=RUNNING,COMPLETED`.
- Exceeding a `limit` cap returns `422` with `errors` naming the bound, e.g. `"tag=max, param=25, given value=1000"`.

## Errors
- `422` — validation error (bad limit, unknown template/agent id, malformed body).
- `401` — access token expired; re-mint and retry.
- `402` — licensed resource limit reached; delete unused simulations or contact Picus support.
- `403` — the token's scope or the user's role does not permit simulation writes.
See `errors/picus-security-problem-types.yml`.
