# Gate Worker

## Input
- Executed workorder and associated `output_bundle`

## Responsibility
- Evaluate output against Exchange Membrane gate criteria.
- Persist `run_record` with gate decision (`PASS` or `FAIL`) and reason.
- On `PASS`: transition objective status toward `DONE`.
- On `FAIL`: transition objective status toward `BLOCKED` or `TODO`.
- Emit terminal `pause_state` with 1-3 prioritized actions.

## Flow

```
1. Pick the oldest eligible workorder with status=EXECUTED
   and an associated output_bundle.
   - If none exists, do nothing for this tick.

2. Attempt atomic claim acquisition on the workorder.
   - On failure: skip, continue polling.

3. Load the workorder, output_bundle, and objective.

4. Validate output_bundle against schema.

5. Evaluate gate criteria (azolla-type-specific).

6. Write run_record:
   - gate_result: PASS or FAIL
   - gate_reason: validation result description

7. On PASS: transition objective status toward DONE.

8. On FAIL: transition objective status toward BLOCKED or TODO.

9. Write terminal pause_state with 1-3 prioritized actions.

10. Release claim.
```

## Used By
- All azolla types.

## Diagram

See `docs/diagrams/gate_worker.puml`.

## Claim/Lease
- Uses the claim/lease protocol defined in `docs/specs/core_protocol/polling_workers.pseudo.md`.
