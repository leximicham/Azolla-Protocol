# Task Management Azolla — v0.2

## 1. Purpose

The task management azolla operates under the standing objective "maintain operator context continuity." It provides three capabilities:

1. Capture and triage raw operator inputs into structured candidate objectives.
2. Synthesize cross-azolla state into orientation manifests for session resumption.
3. Monitor urgency-derived deadlines and surface overdue items via local pause_state.

This document is normative for lifecycle behavior within this azolla type and references shared JSON schemas as data contracts.

## 2. Scope

Included in v0.2:

- Capture buffer intake and lifecycle.
- Triage processing via `TRIAGE_DIAZOTROPH`.
- Orientation manifest generation via `MANIFEST_BUILDER_DIAZOTROPH`.
- Review of urgency-derived deadlines with local pause_state emission.
- Cross-azolla reads via the substrate query interface (`docs/specs/core_protocol/inter_azolla_protocol.md`).

Out of scope in v0.2:

- Automated objective creation in execution azollas (operator promotes triage output manually).
- Priority scoring or ranking of candidate objectives.
- Cross-azolla writes.
- Dynamic azolla discovery (blueprint registry is static and versioned).

## 3. Objective

The task management azolla uses `objective_type: STANDING` with a single persistent objective:

```json
{
  "objective_id": "obj-ctx-continuity",
  "azolla_id": "az-taskmgmt",
  "title": "Maintain operator context continuity",
  "acceptance_criteria": "Operator can resume any session with a current orientation manifest; all raw captures are triaged; urgency-derived deadlines are monitored",
  "objective_type": "STANDING",
  "status": "IN_PROGRESS",
  "importance": 0,
  "urgency": 0,
  "blocked_reason": null,
  "blocked_on": [],
  "metadata": {
    "scope_description": "Cross-azolla state synthesis and operator intake triage"
  },
  "created_at": "2025-06-01T10:00:00Z",
  "updated_at": "2025-06-01T10:00:00Z"
}
```

## 4. Artifact Contracts

Shared schemas (from `docs/specs/core_protocol/schemas/domain/`):

- `objective.schema.json`
- `capture_buffer.schema.json`
- `session_manifest.schema.json`
- `output_bundle.schema.json`
- `context_snapshot.schema.json`
- `workorder.schema.json`
- `event.schema.json`
- `pause_state.schema.json`
- `run_record.schema.json`
- `claim_record.schema.json`
- `blocker_ref.schema.json`

Schema constraints are authoritative if prose and examples diverge.

## 5. Lifecycle

### 5.1 Track A — Triage

1. Operator creates `capture_buffer` with `status: PENDING`.
2. Capture Readiness Worker emits `CAPTURE_READY` event.
3. Scheduler Worker claims event, builds `context_snapshot` from pending capture_buffers (batch of up to N, implementation-defined).
4. Creates `workorder` targeting `TRIAGE_DIAZOTROPH` with bounded `budget_ms` and `branch_name: null`.
5. Runner Worker executes TRIAGE_DIAZOTROPH, which produces an `output_bundle` containing structured candidate objectives.
6. Gate Worker validates output_bundle against schema, writes `run_record` with gate decision.
7. Source capture_buffers transition to `status: PROCESSED`.
8. Gate Worker writes `pause_state` with prioritized actions for the operator (e.g., "Review triage output, promote to objective or discard").

### 5.2 Track B — Manifest Build

1. Operator requests orientation, or a periodic trigger fires.
2. Control plane emits `MANIFEST_REQUESTED` event.
3. Manifest Scheduler Worker claims event, issues `substrate_query` requests to registered azollas for `pause_state` artifacts.
4. Manifest Scheduler Worker reads local capture_buffer counts and objective urgency data.
5. Builds `context_snapshot` containing aggregated cross-azolla data.
6. Creates `workorder` targeting `MANIFEST_BUILDER_DIAZOTROPH` with bounded `budget_ms` and `branch_name: null`.
7. Runner Worker executes MANIFEST_BUILDER_DIAZOTROPH, which produces an `output_bundle` containing orientation manifest text.
8. Gate Worker validates output, writes `run_record` with gate decision.
9. Gate Worker writes `pause_state` with orientation actions from the manifest.

### 5.3 Track C — Deadline Monitoring

1. Deadline Monitor Worker polls objectives (local and cross-azolla-queried) by urgency values.
2. Applies deployment-configurable urgency-to-deadline mapping to determine overdue items.
3. For overdue items: writes local `pause_state` noting the overdue objective and recommended operator action.
4. No cross-azolla writes. The operator acts manually to create blockers or update objectives in target azollas.

### 5.4 Track Serialization

Tracks A and B are serialized: only one workorder is active at a time within this azolla (Charter Section 3.4, single active work stream). Priority ordering when both tracks have pending work: triage before manifest. The manifest reads triage state and should reflect the most current triage results.

Track C is control-plane logic (not a diazotroph execution) and does not count toward the concurrency limit.

## 6. Worker Topology

Six polling workers, using the claim/lease protocol from `docs/specs/core_protocol/polling_workers.pseudo.md`:

1. **Capture Readiness Worker** — `docs/libraries/workers/capture_readiness_worker.md`
2. **Scheduler Worker** — `docs/libraries/workers/scheduler_worker.md`
3. **Runner Worker** — `docs/libraries/workers/runner_worker.md`
4. **Gate Worker** — `docs/libraries/workers/gate_worker.md`
5. **Manifest Scheduler Worker** — `docs/libraries/workers/manifest_scheduler_worker.md`
6. **Deadline Monitor Worker** — `docs/libraries/workers/deadline_monitor_worker.md`

Idempotency rules from `docs/specs/core_protocol/polling_workers.pseudo.md` apply to all workers.

## 7. Diazotroph Types

Two diazotroph types, statically defined:

1. **TRIAGE_DIAZOTROPH** — `docs/libraries/diazotroph_types/triage_diazotroph.md`
2. **MANIFEST_BUILDER_DIAZOTROPH** — `docs/libraries/diazotroph_types/manifest_builder.md`

Both are stateless. All input arrives through `context_snapshot`. Neither issues cross-azolla queries directly.

## 8. Traceability

Each work cycle provides trace links:

- `capture_buffer.capture_id` -> `output_bundle.metadata.capture_ids`
- `workorder.context_snapshot_id` -> `context_snapshot.snapshot_id`
- `run_record.work_order_id` -> `workorder.work_order_id`
- `run_record.output_id` -> `output_bundle.output_id`
- `pause_state.objective_id` -> `objective.objective_id`

## 9. Determinism Requirements

- Every run must be reconstructable from `objective` + `context_snapshot` + `workorder` + `output_bundle` + `run_record`.
- Historical artifacts are immutable after write.
- No hidden background progression outside declared budget windows.

## 10. Acceptance Criteria (v0.2)

The task management azolla specification is complete when the implementation can demonstrate:

1. Creation and validation of all v0.2 schema artifacts (`capture_buffer`, `session_manifest`, `output_bundle` with triage/manifest metadata).
2. Deterministic processing of a `CAPTURE_READY` event into one executed triage workorder producing a valid `output_bundle`.
3. Deterministic processing of a `MANIFEST_REQUESTED` event into one executed manifest workorder producing a valid `output_bundle` with cross-azolla `pause_state` data included.
4. Urgency-based deadline monitoring producing local `pause_state` for overdue items.
5. Gate outcomes represented in `run_record` (`PASS` or `FAIL`).
6. Emission of `pause_state` with bounded prioritized actions after each work cycle.
7. Rehydration of work context from persisted artifacts without external memory.

## 11. UML State Machine

See `docs/specs/task_mgmt/task_mgmt_state_machine.puml`.
