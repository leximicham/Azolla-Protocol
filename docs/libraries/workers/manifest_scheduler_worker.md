# Manifest Scheduler Worker

## Input
- Unprocessed `MANIFEST_REQUESTED` events

## Responsibility
- Claim one event via atomic claim acquisition.
- Issue `substrate_query` requests to registered azollas for `pause_state` artifacts via the inter-azolla query interface.
- Read local capture_buffer counts and objective urgency data.
- Build `context_snapshot` containing aggregated cross-azolla data.
- Create `workorder` targeting `MANIFEST_BUILDER_DIAZOTROPH`.
- Mark event `processed=true` with terminal reason.

## Flow

```
1. Pick the oldest unprocessed MANIFEST_REQUESTED event.
   - If none exists, do nothing for this tick.

2. Attempt atomic claim acquisition on the event.
   - On failure: skip, continue polling.

3. Issue cross-azolla queries:
   - For each azolla in the blueprint registry:
     - Construct substrate_query targeting pause_state artifacts.
     - Submit query to the target azolla's Exchange Membrane.
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
```

## Diagram

See `docs/diagrams/manifest_scheduler_worker.puml`.

## Used By
- Task management azolla (available to all azolla types).

## Claim/Lease
- Uses the claim/lease protocol defined in `docs/specs/core_protocol/polling_workers.pseudo.md`.

## Cross-Azolla Query
- Queries are issued through target azollas' Exchange Membranes.
- Query responses are packaged into the context_snapshot for the MANIFEST_BUILDER_DIAZOTROPH diazotroph.
- The diazotroph does not issue queries directly; all cross-azolla reads occur at scheduling time.
