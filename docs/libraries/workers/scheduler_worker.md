# Scheduler Worker

## Input
- Unprocessed events (e.g., `TICKET_READY`, `CAPTURE_READY`)

## Responsibility
- Claim one event via atomic claim acquisition.
- Build `context_snapshot` from the objective and current state.
- Create `workorder` with bounded budget, branch target (if applicable), and diazotroph type.
- Set objective to `IN_PROGRESS`.
- Mark event `processed=true` with terminal reason.

## Flow

```
1. Pick the oldest unprocessed readiness event.
   - If none exists, do nothing for this tick.

2. Attempt atomic claim acquisition on the event.
   - On failure: skip, continue polling.

3. Load and validate the objective referenced by the event.
   - If the objective is not found: mark event processed
     with terminal reason, release claim, exit.
   - If the objective status is not executable: mark event
     processed with terminal reason, release claim, exit.

4. Check for unresolved blockers.
   - If blockers are present: write pause_state, update
     objective status, mark event processed, release claim, exit.

5. Build context_snapshot from the objective and current state.

6. Create workorder with bounded budget, diazotroph type,
   and branch target (if applicable). Set status to CREATED.

7. Set objective status to IN_PROGRESS.

8. Mark event processed (reason: SCHEDULED).

9. Release claim.
```

## Used By
- All azolla types.

## Diagram

See `docs/diagrams/scheduler_worker.puml`.

## Claim/Lease
- Uses the claim/lease protocol defined in `docs/specs/core_protocol/polling_workers.pseudo.md`.
