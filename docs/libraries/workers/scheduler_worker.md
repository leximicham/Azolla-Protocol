# Scheduler Worker

## Input
- Unprocessed events (e.g., `TICKET_READY`, `CAPTURE_READY`)

## Responsibility
- Claim one event via atomic claim acquisition.
- Build `context_snapshot` from the objective and current state.
- Create `workorder` with bounded budget, branch target (if applicable), and diazotroph type.
- Set objective to `IN_PROGRESS`.
- Mark event `processed=true` with terminal reason.

## Used By
- All azolla types.

## Claim/Lease
- Uses the claim/lease protocol defined in `docs/specs/execution/patch/flows/polling_workers_v0_1.pseudo.md`.
