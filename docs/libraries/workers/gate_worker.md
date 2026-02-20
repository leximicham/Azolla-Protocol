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

## Used By
- All azolla types.
