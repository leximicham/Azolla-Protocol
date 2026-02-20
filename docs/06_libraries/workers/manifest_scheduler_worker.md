# Manifest Scheduler Worker

## Input
- Unprocessed `MANIFEST_REQUESTED` events

## Responsibility
- Claim one event via atomic claim acquisition.
- Issue `substrate_query` requests to registered azollas for `pause_state` artifacts via the inter-azolla query interface.
- Read local capture_buffer counts and objective urgency data.
- Build `context_snapshot` containing aggregated cross-azolla data.
- Create `workorder` targeting `MANIFEST_BUILDER`.
- Mark event `processed=true` with terminal reason.

## Used By
- Task management azolla (available to all azolla types).

## Cross-Azolla Query
- Queries are issued through target azollas' Exchange Membranes.
- Query responses are packaged into the context_snapshot for the MANIFEST_BUILDER diazotroph.
- The diazotroph does not issue queries directly; all cross-azolla reads occur at scheduling time.
