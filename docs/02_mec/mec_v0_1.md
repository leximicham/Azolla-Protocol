# MEC v0.1 — Minimal Executable Core

## 1. Purpose

MEC v0.1 defines the smallest deterministic implementation slice of the Azolla Protocol that can:

1. accept a single-objective ticket,
2. create bounded execution work,
3. process one diazotroph run,
4. gate and persist outputs,
5. pause with prioritized next actions.

This document is normative for lifecycle behavior and references JSON schemas as data contracts.

## 2. Scope

Included in v0.1:

- Single azolla objective lifecycle via `ticket`.
- Work preparation via `context_snapshot` and `workorder`.
- One runner type: `PATCH_DIAZOTROPH`.
- Output persistence via `output_bundle` and `run_record`.
- Event trigger support for `TICKET_READY`.
- Paused-state emission via `pause_state`.

Out of scope in v0.1:

- Multi-repo coordination.
- Dynamic runner type registration.
- Recursive worker spawning.
- Autonomous completion without explicit user confirmation.

## 3. Artifact Contracts

Normative schemas:

- `docs/02_mec/schemas/ticket.schema.json`
- `docs/02_mec/schemas/blocker_ref.schema.json`
- `docs/02_mec/schemas/context_snapshot.schema.json`
- `docs/02_mec/schemas/workorder.schema.json`
- `docs/02_mec/schemas/output_bundle.schema.json`
- `docs/02_mec/schemas/run_record.schema.json`
- `docs/02_mec/schemas/event.schema.json`
- `docs/02_mec/schemas/pause_state.schema.json`

Schema constraints are authoritative if prose and examples diverge.

## 4. Lifecycle

### 4.1 Ticket Intake

- Create `ticket` in `NEW` or `TODO` status.
- Validate objective clarity and acceptance criteria.
- Attach unresolved blockers as `blocked_on` entries when required.

### 4.2 Readiness Transition

- Emit `event` of type `TICKET_READY` when a ticket is executable.
- Control plane marks event as `processed` when handling reaches a terminal outcome:
  successful scheduling, `MISSING_TICKET`, `NON_EXECUTABLE_STATUS`, or `BLOCKED`.

### 4.3 Workorder Creation

- Build immutable `context_snapshot` for the current repo and objective state.
- Create `workorder` with bounded `budget_ms`, branch target, and runner type.

### 4.4 Execution

- Runner executes workorder with snapshot context.
- Runner produces one `output_bundle`.
- Control plane writes one `run_record` with gate decision.

### 4.5 Gating and Pause

- If gates pass: update ticket status toward `DONE` and persist commit linkage.
- If gates fail: preserve failure reason and optional blockers.
- In all terminal run outcomes, write `pause_state` with 1–3 prioritized actions.

## 5. Determinism Requirements

- Every run must be reconstructable from `ticket` + `context_snapshot` + `workorder` + `output_bundle` + `run_record`.
- Historical artifacts are immutable after write.
- No hidden background progression outside declared budget windows.

## 6. Acceptance Criteria (v0.1)

MEC v0.1 is complete when the implementation can demonstrate:

1. Creation and validation of all v0.1 schema artifacts.
2. Deterministic processing of a `TICKET_READY` event into one executed workorder.
3. Gate outcomes represented in `run_record` (`PASS` or `FAIL`).
4. Emission of `pause_state` with bounded prioritized actions.
5. Rehydration of run context from persisted artifacts without external memory.

## 7. Failure Semantics

The control plane must halt and write explicit state when:

- required schema validation fails,
- gate checks fail,
- budget expires before completion,
- unresolved blockers prevent continuation.

Failure handling must be represented in durable artifacts, not implicit logs.

## 8. Traceability

Each work cycle must provide trace links:

- `ticket.ticket_id` -> `workorder.ticket_id`
- `workorder.context_snapshot_id` -> `context_snapshot.snapshot_id`
- `run_record.work_order_id` -> `workorder.work_order_id`
- `run_record.output_id` -> `output_bundle.output_id`
- `pause_state.ticket_id` -> `ticket.ticket_id`
