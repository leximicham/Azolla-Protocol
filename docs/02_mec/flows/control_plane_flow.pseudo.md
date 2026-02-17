# Control Plane Flow (Pseudo)

## Intent

Describe, in plain language, how one `TICKET_READY` event becomes one evaluated run.

## Narrative Flow

1. **Pick the next ready event**
   - The control plane looks for the oldest unprocessed `TICKET_READY` event.
   - If none exists, it does nothing for this tick.

2. **Load and validate the ticket**
   - Read the ticket referenced by the event.
   - If the ticket cannot be found, mark the event as processed with reason `MISSING_TICKET`.
   - If the ticket is already terminal (`DONE` or `BLOCKED`), mark the event as processed with reason `NON_EXECUTABLE_STATUS`.

3. **Check blockers before scheduling work**
   - If unresolved blockers are present:
     - write a `pause_state` explaining that gates are not clear,
     - include prioritized actions (for example: resolve blockers, then re-emit `TICKET_READY`),
     - set the ticket to `BLOCKED`,
     - mark the event as processed with reason `BLOCKED`.

4. **Prepare bounded execution context**
   - Build a `context_snapshot` for the current ticket state.
   - Create one `workorder` that specifies:
     - runner type (`PATCH_DIAZOTROPH`),
     - target branch name,
     - execution budget.
   - Set ticket status to `IN_PROGRESS`.

5. **Run the assigned diazotroph**
   - Invoke the runner with the workorder.
   - Persist exactly one `output_bundle` from runner output.

6. **Evaluate gates and record the run**
   - Evaluate gates against the ticket and persisted `output_bundle`.
   - Write a `run_record` containing the gate result, reason, and commit linkage when available.

7. **Publish terminal ticket state and next actions**
   - If gate result is `PASS`:
     - set ticket status to `DONE`,
     - write `pause_state` with completion-oriented follow-ups (review patch, confirm objective completion).
   - If gate result is `FAIL`:
     - set ticket status to `BLOCKED`,
     - write `pause_state` with recovery-oriented follow-ups (address gate failures, re-run when green).

8. **Finalize bookkeeping**
   - Mark the workorder as executed.
   - Mark the source event as processed with reason `EXECUTED`.
   - Return the `run_id` for traceability.

## Deterministic Guarantees

- One processed `TICKET_READY` event results in at most one executed workorder.
- One executed workorder produces one persisted `output_bundle` and one `run_record`.
- Every terminal outcome writes a `pause_state` with explicit next actions.
