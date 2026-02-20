# Readiness Worker

## Input
- `objective` artifacts with `status: NEW`

## Responsibility
- Enforce `NEW` vs `TODO` semantics.
- Require human validation before `NEW` -> `TODO` transition.
- Emit `TICKET_READY` event for executable `TODO` objectives.

## Used By
- All azolla types.

## Notes
- Autonomous diazotroph-based review may replace the human promotion step in a later version without changing downstream scheduler/runner/gate responsibilities.
