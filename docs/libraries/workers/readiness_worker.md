# Readiness Worker

## Input
- `objective` artifacts with `status: NEW`

## Responsibility
- Enforce `NEW` vs `TODO` semantics.
- Require human validation before `NEW` -> `TODO` transition.
- Emit `TICKET_READY` event for executable `TODO` objectives.

## Flow

```
1. Poll objectives from the local Substrate.
   - Read objectives eligible for status promotion.

2. For each eligible objective:
   - Verify that the required status transition prerequisites are met
     (e.g., human review for NEW -> TODO).
   - If prerequisites are not met, skip.

3. Emit a readiness event for the objective.
   - Check for existing unprocessed readiness events to avoid duplicates.

4. No claim/lease required — readiness evaluation is idempotent
   and does not mutate lifecycle state beyond event emission.
```

## Used By
- All azolla types.

## Diagram

See `docs/diagrams/readiness_worker.puml`.

## Notes
- Autonomous diazotroph-based review may replace the human promotion step in a later version without changing downstream scheduler/runner/gate responsibilities.
