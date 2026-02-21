# Patch Diazotroph — Polling Workers Topology

## Intent

Define the worker topology for the code patch azolla, referencing shared workers from the component library and specifying patch-specific coordination rules.

## Workers

### 1. Readiness Worker

- Library definition: `docs/libraries/workers/readiness_worker.md`
- Input: `objective` artifacts with `status: NEW`
- Responsibility: enforce `NEW` vs `TODO` semantics, require human validation before `NEW -> TODO`, emit `TICKET_READY` for executable `TODO` objectives
- Must avoid duplicate event emissions for the same objective state transition

### 2. Scheduler Worker

- Library definition: `docs/libraries/workers/scheduler_worker.md`
- Input: unprocessed `TICKET_READY` events
- Responsibility: claim event, build context_snapshot from objective and repo state, create workorder targeting `PATCH_DIAZOTROPH`
- Mark event `processed=true` with terminal reason

### 3. Runner Worker

- Library definition: `docs/libraries/workers/runner_worker.md`
- Input: `workorder.status=CREATED`
- Responsibility: claim workorder, dispatch to `PATCH_DIAZOTROPH`, persist `output_bundle`, set workorder to `EXECUTED`

### 4. Gate Worker

- Library definition: `docs/libraries/workers/gate_worker.md`
- Input: executed workorder + output_bundle
- Responsibility: evaluate gates, extract `commit_sha` from output_bundle metadata, write `run_record`, transition objective status, emit terminal `pause_state`

## Claim/Lease Protocol

All workers use the claim/lease protocol defined in `docs/specs/core_protocol/polling_workers.pseudo.md`.

Required claim fields are specified by `docs/specs/core_protocol/schemas/domain/claim_record.schema.json`.

## Idempotency Rules

- Reprocessing an already processed event is a no-op.
- Re-running scheduler for the same event must not create duplicate workorders.
- Re-running runner for an already executed workorder is a no-op.
- Gate worker must upsert by `work_order_id` so one workorder maps to one terminal `run_record`.

## Coordination Rules

- Claims/leasing are required for event/workorder processing to avoid double execution.
- Each worker must be idempotent for retries after partial failures.
- Poll interval is implementation-defined; correctness does not depend on exact interval values.

## Human-in-the-Loop Boundary

- Human performs requirement validation and approval (`NEW -> TODO`).
- Autonomous diazotroph-based review can replace this boundary in a later version without changing downstream scheduler/runner/gate responsibilities.
