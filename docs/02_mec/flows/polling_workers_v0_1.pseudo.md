# Polling Workers Topology (v0.1)

## Intent

Make worker ownership explicit for MEC v0.1 so ticket lifecycle transitions are deterministic under interval polling.

## Workers

1. **Readiness Worker**
   - Input: `ticket`
   - Responsibility:
     - enforce `NEW` vs `TODO` semantics,
     - require human validation before `NEW -> TODO`,
     - emit `TICKET_READY` for executable `TODO` tickets.

2. **Scheduler Worker**
   - Input: unprocessed `TICKET_READY` events
   - Responsibility:
     - claim one event,
     - create `context_snapshot` + `workorder`,
     - set ticket to `IN_PROGRESS`,
     - mark event `processed=true` with terminal reason.

3. **Runner Worker**
   - Input: `workorder.status=CREATED`
   - Responsibility:
     - claim one workorder,
     - execute diazotroph,
     - persist `output_bundle`,
     - set workorder to `EXECUTED`.

4. **Gate Worker**
   - Input: executed workorder + output
   - Responsibility:
     - evaluate gates,
     - persist `run_record`,
     - move ticket to `DONE` or `BLOCKED|TODO`,
     - emit terminal `pause_state`.

## Claim/Lease Protocol (Normative v0.1 Behavior)

v0.1 workers maintain claim metadata in implementation storage (for example, a lock table keyed by `event_id` or `work_order_id`) and must validate each record against `docs/02_mec/schemas/claim_record.schema.json`.

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

## Idempotency Rules (Normative v0.1 Behavior)

- Reprocessing an already processed event must be a no-op.
- Re-running scheduler for the same event must not create duplicate workorders.
- Re-running runner for an already executed workorder must be a no-op.
- Gate worker must upsert by `work_order_id` so one workorder maps to one terminal `run_record`.

## Coordination Rules

- Claims/leasing are required for event/workorder processing to avoid double execution.
- Each worker must be idempotent for retries after partial failures.
- Poll interval is implementation-defined; correctness does not depend on exact interval values.

## v0.1 Human-in-the-Loop Boundary

- Human performs requirement validation and approval (`NEW -> TODO`).
- Autonomous review diazotroph can replace this boundary in a later version without changing downstream scheduler/runner/gate responsibilities.
