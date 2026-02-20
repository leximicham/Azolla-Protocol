# Capture Readiness Worker

## Input
- `capture_buffer` artifacts with `status: PENDING`

## Responsibility
- Emit `CAPTURE_READY` event for pending capture_buffers.
- Avoid duplicate event emissions for the same capture_buffer.

## Used By
- Task management azolla (available to all azolla types).
