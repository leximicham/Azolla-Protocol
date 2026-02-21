# Inter-Azolla Protocol

## 1. Purpose

This document defines how independently deployed azollas coordinate through read-only queries. Each azolla owns its own Substrate; no shared substrate exists.

## 2. Principles

- **Substrate isolation**: Each azolla owns its own Substrate. No shared storage across azollas.
- **Read-only cross-azolla access**: Cross-azolla reads go through the target azolla's Exchange Membrane. No cross-azolla writes are permitted in v0.2.
- **Control-plane-mediated queries**: Diazotrophs never issue cross-azolla queries directly. The requesting azolla's control plane issues queries at scheduling time and packages results into the context_snapshot for the diazotroph.
- **No cross-azolla event propagation**: Events are control-plane-internal signals. They do not cross azolla boundaries.

## 3. Substrate Query Interface

The substrate query interface is a typed request/response contract for cross-azolla reads.

### 3.1 Query Flow

1. Requesting azolla's control plane constructs a `substrate_query` specifying the target azolla, artifact type, and filter criteria.
2. The query is submitted to the target azolla's Exchange Membrane.
3. The Exchange Membrane validates the query (Section 4).
4. On validation success, the Exchange Membrane reads matching artifacts from the target Substrate and returns a `substrate_query_response` with status `OK`.
5. On validation failure, the response status is `DENIED`.
6. If the target azolla is unreachable, the response status is `TARGET_UNAVAILABLE`.

### 3.2 Schemas

- `docs/specs/core_protocol/schemas/transport/substrate_query.schema.json`
- `docs/specs/core_protocol/schemas/transport/substrate_query_response.schema.json`

## 4. Exchange Membrane Query Validation

The target azolla's Exchange Membrane validates inbound queries by checking:

1. The `source_azolla_id` is a known azolla identifier.
2. The requesting azolla type declares read access to the requested `artifact_type` in its blueprint (`cross_azolla_reads` field).
3. The `artifact_type` is an artifact the target azolla actually stores in its Substrate.

If any check fails, the response status is `DENIED`.

## 5. Context Preparation

When a requesting azolla's scheduler builds a context_snapshot for a diazotroph that requires cross-azolla data:

1. The scheduler iterates through the relevant azolla identifiers.
2. For each, it issues a `substrate_query` to the target Exchange Membrane.
3. It collects and aggregates all responses.
4. It packages the aggregated data into the `full_prompt_text` or related fields of the context_snapshot.
5. The diazotroph receives the context_snapshot and operates statelessly on the packaged data.

This ensures all cross-azolla reads happen at scheduling time (control plane responsibility) rather than execution time (diazotroph responsibility), preserving stateless execution (Charter Section 3.2).

## 6. Cross-Azolla Writes

No cross-azolla writes are permitted in v0.2. When a memory-support azolla identifies an action that affects an execution azolla (e.g., a lapsed deadline), it writes a local `pause_state` with a recommended operator action. The operator acts manually to create blockers or update objectives in the target azolla.
