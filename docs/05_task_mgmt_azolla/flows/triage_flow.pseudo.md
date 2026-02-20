# Triage Flow

## Intent

Convert pending `capture_buffer` artifacts into structured candidate objectives via `TRIAGE_DIAZOTROPH` execution.

## Preconditions

- One or more `capture_buffer` artifacts exist with `status: PENDING`.
- A `CAPTURE_READY` event has been emitted and is unprocessed.

## Flow

```
1. Pick oldest unprocessed CAPTURE_READY event.

2. Attempt atomic claim acquisition on event.
   - On failure: skip, continue polling.

3. Load all capture_buffer artifacts with status=PENDING
   (up to batch limit N, implementation-defined).

4. If no PENDING capture_buffers exist:
   - Mark event processed (reason: NO_PENDING_CAPTURES).
   - Release claim.
   - Exit.

5. Build context_snapshot:
   - objective_id: standing objective identifier
   - repo_commit_sha: null or current state reference
   - full_prompt_text: triage instructions + serialized capture_buffer contents
   - related_yield_refs: []

6. Create workorder:
   - diazotroph_type: TRIAGE_DIAZOTROPH
   - context_snapshot_id: from step 5
   - branch_name: null
   - budget_ms: deployment-configured triage budget
   - status: CREATED

7. Mark event processed (reason: SCHEDULED).

8. Release claim.

9. Runner Worker claims workorder:
   - Dispatch to TRIAGE_DIAZOTROPH.
   - TRIAGE_DIAZOTROPH produces output_bundle:
     - metadata: { capture_ids: [...], suggested_target_azolla_type: "..." }
     - content: structured candidate objective description
     - title: triage result summary
     - notes: confidence notes, ambiguities
   - Persist output_bundle.
   - Set workorder status to EXECUTED.

10. Gate Worker evaluates output:
    - Validate output_bundle against schema.
    - Check metadata.capture_ids references valid capture_buffers.
    - Check content contains non-empty candidate title and acceptance criteria.

11. Gate Worker writes run_record:
    - gate_result: PASS or FAIL
    - gate_reason: validation result description

12. On PASS:
    - Transition source capture_buffers to status: PROCESSED.
    - Write pause_state (reason: RUN_COMPLETE) with prioritized actions:
      e.g., ["Review triage output for obj-ctx-continuity", "Promote candidate to objective or discard"]

13. On FAIL:
    - Write pause_state (reason: GATE_FAILED) with prioritized actions:
      e.g., ["Review gate failure reason", "Retry triage or manually process captures"]
```

## Idempotency

- Reprocessing an already processed event is a no-op.
- Re-running scheduler for the same event must not create duplicate workorders.
- Re-running runner for an already executed workorder is a no-op.
- Gate worker must upsert by work_order_id so one workorder maps to one terminal run_record.
