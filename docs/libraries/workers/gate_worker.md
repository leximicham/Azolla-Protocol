# Gate Worker

## Input
- Executed workorder and associated `output_bundle`

## Responsibility
- Evaluate output against Exchange Membrane gate criteria.
- Extract `commit_sha` from `output_bundle.metadata` if present (for diazotrophs that produce repository state such as `PATCH_DIAZOTROPH`).
- Persist `run_record` with gate decision (`PASS` or `FAIL`), reason, and optional `commit_sha`.
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
   - Extract metadata fields (e.g., commit_sha) if present.

6. Write run_record:
   - gate_result: PASS or FAIL
   - gate_reason: validation result description
   - Optional metadata (e.g., commit_sha)

7. On PASS: transition objective status toward DONE.

8. On FAIL: transition objective status toward BLOCKED or TODO.

9. Write terminal pause_state with 1-3 prioritized actions.

10. Release claim.
```

## Used By
- All azolla types.

## Claim/Lease
- Uses the claim/lease protocol defined in `docs/specs/core_protocol/polling_workers.pseudo.md`.
