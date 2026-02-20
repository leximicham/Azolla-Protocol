# Azolla Protocol Glossary

## A

### azolla
A single deployment instance running under the Azolla Protocol. An azolla comprises three named internal components — the Azolla Control Plane, the Azolla Substrate, and the Exchange Membrane — and owns one active objective at a time. Diazotrophs are external to the azolla; they operate across its trust boundary.

### azolla_category
A classification of azolla deployment types. v0.2 defines two categories: `EXECUTION` (azollas that produce deliverables through bounded diazotroph execution) and `MEMORY_SUPPORT` (azollas that maintain, synthesize, or surface state information for the operator).

### Azolla Control Plane
An internal component of an azolla. The governance and orchestration layer that schedules work, enforces gates, updates durable state, and applies pause semantics.

### Azolla Protocol
The architecture, constraints, blueprint library, and documentation set that defines how azollas, diazotrophs, and substrate artifacts interact.

## B

### blocker_ref
A typed dependency blocker attached to an objective. Used to encode why an objective cannot progress and whether the blocker is resolved.

### blueprint
A static, versioned composition of shared schemas, library workers, and diazotroph types that defines an azolla type. Blueprints are documentation artifacts, not runtime constructs.

## C

### capture_buffer
A universal intake artifact for raw external inputs (operator notes, feature requests, requirement lists, etc.) awaiting processing. Available to any azolla type. Transitions from `PENDING` to `PROCESSED`.

### claim_record
An atomic lease record used by polling workers to prevent double-processing of events and workorders. Contains a worker identifier, resource reference, lease expiry, and terminal status.

### context_snapshot
An immutable, versioned capture of all run-critical context for a single workorder execution.

## D

### diazotroph
A stateless worker process that performs bounded generation/execution work from a workorder and context snapshot. v0.2 defines three diazotroph types: `PATCH_DIAZOTROPH`, `TRIAGE_DIAZOTROPH`, and `MANIFEST_BUILDER_DIAZOTROPH`.

## E

### event
A control-plane signal used to trigger deterministic workflow transitions (for example, `TICKET_READY`, `CAPTURE_READY`, `MANIFEST_REQUESTED`).

### Exchange Membrane
The policy boundary between worker output and substrate mutation. It validates output, applies gates, determines state transitions, and validates inbound inter-azolla queries.

### execution azolla
An azolla whose objective is to produce a concrete deliverable through bounded diazotroph execution. Uses `objective_type: TICKET`. See `azolla_category`.

## I

### importance
An integer value (0-4) on an objective indicating how much the objective matters. Interpretation is deployment-specific.

### inter-azolla query
A read-only request issued by one azolla's control plane to another azolla's Exchange Membrane through the substrate query interface.

## M

### MANIFEST_BUILDER_DIAZOTROPH
A stateless diazotroph type that produces orientation manifests from aggregated cross-azolla state data packaged into its context_snapshot.

### memory-support azolla
An azolla whose objective is to maintain, synthesize, or surface state information for the operator rather than produce execution deliverables. Uses `objective_type: STANDING`. See `azolla_category`.

## O

### objective
The protocol's core unit: a single bounded goal owned by one azolla. Includes acceptance criteria, status, importance, urgency, and a metadata property for type-specific data. Replaces `ticket` as the canonical artifact name in v0.2. Objectives with `objective_type: TICKET` carry repository and branch metadata; objectives with `objective_type: STANDING` represent persistent operational goals.

### output_bundle
An immutable output artifact from one diazotroph execution. Contains a title, metadata (type-specific structured data), content (primary output payload), and execution notes.

## P

### PATCH_DIAZOTROPH
A stateless diazotroph type that produces code patches from an objective's repository state and prompt data packaged into its context_snapshot. Used by execution azollas.

### pause_state
A durable paused-state artifact written when execution stops due to completion, gate failure, or budget exhaustion. Contains a short prioritized action list.

## R

### run_record
An immutable audit artifact describing one executed workorder and gate outcome.

## S

### session_manifest
A universal orientation artifact synthesized from cross-azolla state for operator session resumption. Contains pause state summaries from queried azollas and a prioritized list of recommended operator actions. Available to any azolla type.

### Substrate
The durable storage layer for objectives, snapshots, bundles, run records, events, and pause states.

### substrate_query
A typed, read-only request issued by one azolla's control plane to another azolla's Exchange Membrane. Contains source and target azolla identifiers, an artifact type filter, and query parameters.

### substrate_query_response
The response to a substrate_query. Contains the query outcome status (`OK`, `DENIED`, or `TARGET_UNAVAILABLE`) and an array of matching artifacts.

## T

### task management azolla
A memory-support azolla operating under the standing objective "maintain operator context continuity." Captures raw operator inputs, triages them into candidate objectives, and synthesizes orientation manifests from cross-azolla state.

### ticket
A specialization of `objective` with `objective_type: TICKET`. Carries repository and branch metadata in the objective's `metadata` property. Used by execution azollas.

### TRIAGE_DIAZOTROPH
A stateless diazotroph type that converts raw `capture_buffer` artifacts into structured candidate objectives packaged as `output_bundle` artifacts.

## U

### urgency
An integer value (0-4) on an objective indicating time-sensitivity. Each deployment configures what urgency values map to in terms of time-based deadlines (e.g., 0 = immediate action required, 4 = address when spare cycles are available).

## W

### workorder
A concrete executable unit created from an objective, with an assigned diazotroph type, budget, branch target (if applicable), and snapshot reference.
