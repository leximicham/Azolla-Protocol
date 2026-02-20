# Task Management Azolla — Polling Workers Topology

## Intent

Define the worker topology for the task management azolla, referencing shared workers from the component library and specifying task-management-specific coordination rules.

## Workers

### 1. Capture Readiness Worker

- Library definition: `docs/libraries/workers/capture_readiness_worker.md`
- Input: `capture_buffer` artifacts with `status: PENDING`
- Responsibility: emit `CAPTURE_READY` event for pending capture_buffers
- Must avoid duplicate event emissions for the same capture_buffer

### 2. Scheduler Worker

- Library definition: `docs/libraries/workers/scheduler_worker.md`
- Input: unprocessed `CAPTURE_READY` events
- Responsibility: claim event, build context_snapshot from pending captures, create triage workorder
- Mark event `processed=true` with terminal reason

### 3. Manifest Scheduler Worker

- Library definition: `docs/libraries/workers/manifest_scheduler_worker.md`
- Input: unprocessed `MANIFEST_REQUESTED` events
- Responsibility: claim event, issue cross-azolla substrate queries, build context_snapshot, create manifest workorder
- Mark event `processed=true` with terminal reason

### 4. Runner Worker

- Library definition: `docs/libraries/workers/runner_worker.md`
- Input: `workorder.status=CREATED`
- Responsibility: claim workorder, dispatch to diazotroph by `workorder.diazotroph_type`, persist `output_bundle`, set workorder to `EXECUTED`

### 5. Gate Worker

- Library definition: `docs/libraries/workers/gate_worker.md`
- Input: executed workorder + output_bundle
- Responsibility: evaluate gates, write `run_record`, transition artifact states, emit terminal `pause_state`

### 6. Deadline Monitor Worker

- Library definition: `docs/libraries/workers/deadline_monitor_worker.md`
- Input: objectives with urgency values
- Responsibility: poll urgency-based deadlines, write local `pause_state` for overdue items
- Control-plane logic; does not count toward concurrency limit

## Claim/Lease Protocol

All workers (except the Deadline Monitor Worker) use the claim/lease protocol defined in `docs/mec/flows/polling_workers_v0_1.pseudo.md`.

Required claim fields are specified by `docs/mec/schemas/claim_record.schema.json`.

## Track Serialization

- Tracks A (triage) and B (manifest) are serialized: only one workorder is active at a time.
- Priority ordering: triage before manifest.
- Track C (deadline monitoring) is independent control-plane logic.

## Idempotency Rules

- Reprocessing an already processed event is a no-op.
- Re-running scheduler for the same event must not create duplicate workorders.
- Re-running runner for an already executed workorder is a no-op.
- Gate worker must upsert by `work_order_id` so one workorder maps to one terminal `run_record`.

## Coordination Rules

- Claims/leasing are required for event/workorder processing to avoid double execution.
- Each worker must be idempotent for retries after partial failures.
- Poll interval is implementation-defined; correctness does not depend on exact interval values.
