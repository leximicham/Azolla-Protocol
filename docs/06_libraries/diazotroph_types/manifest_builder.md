# MANIFEST_BUILDER_DIAZOTROPH

## Description
A stateless worker that produces an orientation manifest from aggregated cross-azolla state data.

## Input Contract
Receives a `context_snapshot` containing:
- Aggregated `pause_state` data from cross-azolla queries (serialized substrate_query_response results)
- Local capture_buffer status counts
- Objective urgency data
- Full prompt text with manifest format requirements and orientation generation instructions

## Output Contract
Produces one `output_bundle` with:
- `metadata`: `{ "queried_azolla_ids": ["<string>", ...] }`
- `content`: orientation manifest text with prioritized operator actions
- `title`: manifest summary
- `notes`: notes on data freshness, unavailable azollas, or query failures

## Execution
1. Parse aggregated cross-azolla state from the context_snapshot.
2. Invoke external LLM provider API with bounded budget.
3. Synthesize prioritized orientation actions from pause_states, pending captures, and urgency data.
4. Emit `output_bundle`.

## Gate Criteria
The Exchange Membrane evaluates:
- Output validates against the `output_bundle` schema.
- `metadata.queried_azolla_ids` is non-empty.
- `content` contains actionable orientation text.

## Available To
- Task management azolla (available to all azolla types).
