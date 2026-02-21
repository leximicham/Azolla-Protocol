# Capture Readiness Worker

## Input
- `capture_buffer` artifacts with `status: PENDING`

## Responsibility
- Emit `CAPTURE_READY` event for pending capture_buffers.
- Avoid duplicate event emissions for the same capture_buffer.

## Flow

```
1. Poll capture_buffer artifacts from the local Substrate.
   - Read capture_buffers with status=PENDING.

2. For each pending capture_buffer:
   - Check for an existing unprocessed CAPTURE_READY event
     referencing this capture_buffer.
   - If an unprocessed event already exists, skip (avoid duplicate emission).

3. Emit a CAPTURE_READY event for the capture_buffer.

4. No claim/lease required — readiness evaluation is idempotent
   and does not mutate lifecycle state beyond event emission.
```

## Diagram

See `docs/diagrams/capture_readiness_worker.puml`.

## Used By
- Task management azolla (available to all azolla types).
