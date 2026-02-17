# Control Plane Flow (Pseudo)

## Intent

Describe, in plain language, how one `TICKET_READY` event becomes one evaluated run.

## Worker Model (Polling)

v0.1 assumes interval-based ACP workers, not long-lived stream consumers:

- **Readiness worker** (poll interval: implementation-defined) is responsible for:
  - scanning tickets that can advance from `NEW -> TODO` only after explicit human review,
  - emitting `TICKET_READY` only for executable `TODO` tickets,
  - avoiding duplicate emissions for the same ticket state transition.
- **Control-plane scheduler worker** (poll interval: implementation-defined) is responsible for:
  - claiming unprocessed `TICKET_READY` events,
  - creating bounded work (`context_snapshot` + `workorder`),
  - recording terminal processing outcomes and setting `event.processed=true`.

Claim/lease protocol details are defined in `polling_workers_v0_1.pseudo.md` and are normative for this flow.

## v0.1 Status Semantics

- `NEW` means intake-complete but not execution-approved.
- `TODO` means human-reviewed and execution-approved.
- Only `TODO` tickets should have `TICKET_READY` emitted in v0.1.

## Narrative Flow

1. **Pick the next ready event**
   - The scheduler worker looks for the oldest unprocessed `TICKET_READY` event.
   - If none exists, it does nothing for this tick.

2. **Claim event for this tick**
   - Attempt atomic claim acquisition for `event_id`.
   - If claim fails (another worker holds active lease), skip and continue.

3. **Load and validate the ticket**
   - Read the ticket referenced by the event.
   - If the ticket cannot be found, mark the event as processed with reason `MISSING_TICKET`.
   - If the ticket is not executable (`NEW`, `DONE`, or `BLOCKED`), mark the event as processed with reason `NON_EXECUTABLE_STATUS`.

4. **Check blockers before scheduling work**
   - If unresolved blockers are present:
     - write a `pause_state` explaining that gates are not clear,
     - include prioritized actions (for example: resolve blockers, then re-emit `TICKET_READY`),
     - set the ticket to `BLOCKED`,
     - mark the event as processed with reason `BLOCKED`.

5. **Prepare bounded execution context**
   - Build a `context_snapshot` for the current ticket state.
   - Create one `workorder` that specifies:
     - runner type (`PATCH_DIAZOTROPH`),
     - target branch name,
     - execution budget.
   - Set ticket status to `IN_PROGRESS`.

6. **Finalize scheduler bookkeeping**
   - Mark the source event as processed with reason `SCHEDULED`.
   - Release claim.
   - Return the `work_order_id` for traceability.

## Deterministic Guarantees

- One processed `TICKET_READY` event results in at most one created workorder.
- Every event terminal path sets `event.processed=true` exactly once.
- Every terminal outcome writes explicit state (`workorder` or terminal reason artifacts), not implicit logs.
