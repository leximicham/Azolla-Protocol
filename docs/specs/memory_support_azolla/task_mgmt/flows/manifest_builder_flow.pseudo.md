# Manifest Builder Flow

## Intent

Produce an orientation manifest from aggregated cross-azolla state for operator session resumption.

## Preconditions

- A `MANIFEST_REQUESTED` event has been emitted and is unprocessed.
- At least one other azolla is registered and reachable via the inter-azolla query interface.

## Claim/Lease Protocol

Claim/lease protocol details are defined in `docs/specs/core_protocol/polling_workers.pseudo.md` and are normative for this flow.

## Flow

```
1. Pick oldest unprocessed MANIFEST_REQUESTED event.

2. Attempt atomic claim acquisition on event.
   - On failure: skip, continue polling.

3. Issue cross-azolla queries:
   - For each azolla in the blueprint registry:
     - Construct substrate_query targeting pause_state artifacts.
     - Submit query to target azolla's Exchange Membrane.
     - Collect substrate_query_response.
   - Aggregate all responses. Note any DENIED or TARGET_UNAVAILABLE statuses.

4. Read local state:
   - Count capture_buffer artifacts with status: PENDING.
   - Read objective urgency data from local and queried objectives.

5. Build context_snapshot:
   - objective_id: standing objective identifier
   - full_prompt_text: manifest format requirements + serialized aggregated data
   - related_yield_refs: []

6. Create workorder:
   - diazotroph_type: MANIFEST_BUILDER_DIAZOTROPH
   - context_snapshot_id: from step 5
   - branch_name: null
   - budget_ms: deployment-configured manifest budget
   - status: CREATED

7. Mark event processed (reason: SCHEDULED).

8. Release claim.

9. Runner Worker claims workorder:
   - Dispatch to MANIFEST_BUILDER_DIAZOTROPH.
   - MANIFEST_BUILDER_DIAZOTROPH produces output_bundle:
     - metadata: { queried_azolla_ids: [...] }
     - content: orientation manifest text with prioritized actions
     - title: manifest summary
     - notes: data freshness, unavailable azollas, query failures
   - Persist output_bundle.
   - Set workorder status to EXECUTED.

10. Gate Worker evaluates output:
    - Validate output_bundle against schema.
    - Check metadata.queried_azolla_ids is non-empty.
    - Check content contains actionable orientation text.

11. Gate Worker writes run_record:
    - gate_result: PASS or FAIL
    - gate_reason: validation result description

12. On PASS:
    - Write pause_state (reason: RUN_COMPLETE) with prioritized actions
      derived from the manifest's orientation items (up to 3).

13. On FAIL:
    - Write pause_state (reason: GATE_FAILED) with prioritized actions:
      e.g., ["Review gate failure reason", "Retry manifest build"]
```

## Idempotency

- Reprocessing an already processed event is a no-op.
- Re-running scheduler for the same event must not create duplicate workorders.
- Re-running runner for an already executed workorder is a no-op.
- Gate worker must upsert by work_order_id so one workorder maps to one terminal run_record.
