# Polling Workers Topology

## Intent

Define the generic worker polling model and claim/lease protocol so objective lifecycle transitions are deterministic under interval polling. All azolla types use this model.

## Worker Roles

The standard polling topology comprises four worker roles. Each azolla type selects and configures these roles in its local topology file.

1. **Readiness Worker**
   - Input: `objective` artifacts
   - Responsibility:
     - enforce status transition semantics,
     - emit a readiness event for executable objectives,
     - avoid duplicate emissions for the same objective state transition.
   - Library definition: `docs/libraries/workers/readiness_worker.md`

2. **Scheduler Worker**
   - Input: unprocessed readiness events
   - Responsibility:
     - claim one event,
     - create `context_snapshot` + `workorder`,
     - set objective to `IN_PROGRESS`,
     - mark event `processed=true` with terminal reason.
   - Library definition: `docs/libraries/workers/scheduler_worker.md`

3. **Runner Worker**
   - Input: `workorder.status=CREATED`
   - Responsibility:
     - claim one workorder,
     - dispatch to the diazotroph type specified in `workorder.diazotroph_type`,
     - persist `output_bundle`,
     - set workorder to `EXECUTED`.
   - Library definition: `docs/libraries/workers/runner_worker.md`

4. **Gate Worker**
   - Input: executed workorder + `output_bundle`
   - Responsibility:
     - evaluate gates,
     - persist `run_record`,
     - transition objective status,
     - emit terminal `pause_state`.
   - Library definition: `docs/libraries/workers/gate_worker.md`

Azolla types may define additional workers (e.g., Capture Readiness Worker, Manifest Scheduler Worker, Deadline Monitor Worker) in their local topology files.

## Claim/Lease Protocol (Normative)

Workers maintain claim metadata in implementation storage (for example, a lock table keyed by `event_id` or `work_order_id`) and must validate each record against `docs/specs/core_protocol/schemas/domain/claim_record.schema.json`.

### Required claim fields

For both event and workorder claims, store at least (as specified by `claim_record.schema.json`):

- `resource_type`: `EVENT` or `WORKORDER`
- `resource_id`: `event_id` or `work_order_id`
- `owner_id`: unique worker/process id
- `lease_expires_at`: absolute UTC expiration timestamp
- `heartbeat_at`: last successful renewal timestamp
- `claim_version`: monotonic counter for compare-and-swap updates

### Claim acquisition

- Worker reads candidate resource (oldest first).
- Worker attempts atomic **create-or-update-if-expired** on claim record.
- Acquisition succeeds only if:
  - no active claim exists, or
  - existing claim is expired (`lease_expires_at <= now`).
- On acquisition failure, worker must skip resource and continue polling.

### Lease duration and renewal

- Lease duration must be short and bounded (recommended 30-120s).
- Long-running work must renew lease periodically (recommended every 10-30s).
- Renewal uses compare-and-swap on `claim_version` to prevent split brain.
- If renewal fails, worker must stop mutating shared state and abort safely.

### Completion and release

- Worker writes terminal artifacts/state transition first.
- Worker releases claim after durable write success.
- If worker crashes, lease expiry enables takeover by another worker.

## Idempotency Rules (Normative)

- Reprocessing an already processed event must be a no-op.
- Re-running scheduler for the same event must not create duplicate workorders.
- Re-running runner for an already executed workorder must be a no-op.
- Gate worker must upsert by `work_order_id` so one workorder maps to one terminal `run_record`.

## Coordination Rules

- Claims/leasing are required for event/workorder processing to avoid double execution.
- Each worker must be idempotent for retries after partial failures.
- Poll interval is implementation-defined; correctness does not depend on exact interval values.

## Human-in-the-Loop Boundary

- Human performs requirement validation and approval (e.g., `NEW -> TODO`).
- Autonomous diazotroph-based review can replace this boundary in a later version without changing downstream scheduler/runner/gate responsibilities.
