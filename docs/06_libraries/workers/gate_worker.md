# Gate Worker

## Input
- Executed workorder and associated `output_bundle`

## Responsibility
- Evaluate output against Exchange Membrane gate criteria.
- Persist `run_record` with gate decision (`PASS` or `FAIL`) and reason.
- On `PASS`: transition objective status toward `DONE`.
- On `FAIL`: transition objective status toward `BLOCKED` or `TODO`.
- Emit terminal `pause_state` with 1-3 prioritized actions.

## Used By
- All azolla types.
