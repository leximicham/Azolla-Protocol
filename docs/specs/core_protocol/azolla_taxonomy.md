# Azolla Taxonomy

## 1. Purpose

This document defines the classification system for azolla deployment types. Each azolla type is a composition of protocol components selected from shared schemas, worker libraries, and diazotroph type libraries.

## 2. Azolla Categories

### 2.1 Execution Azolla

An azolla whose objective is to produce a concrete deliverable through bounded diazotroph execution. The deliverable is a durable output artifact (e.g., a git patch, a document, an analysis).

- Uses `objective_type: TICKET`.
- Diazotrophs produce output_bundles representing direct progress toward the objective.
- Lifecycle follows the standard ticket flow: `NEW` -> `TODO` -> `IN_PROGRESS` -> `DONE` | `BLOCKED`.

### 2.2 Memory-Support Azolla

An azolla whose objective is to maintain, synthesize, or surface state information for the operator. Memory-support azollas read from other azollas' substrates (via the inter-azolla query interface) and produce orientation artifacts.

- Uses `objective_type: STANDING`.
- Diazotrophs produce synthesis artifacts (triage candidates, orientation manifests) rather than execution deliverables.
- Does not directly advance any execution azolla's objective.
- May read from other azollas' substrates through the inter-azolla query interface defined in `docs/specs/core_protocol/inter_azolla_protocol.md`.

## 3. Shared Structural Contract

All azollas, regardless of category, share:

- **Internal architecture**: Azolla Control Plane, Azolla Substrate (Context Store + Yield Store), Exchange Membrane.
- **One active objective** (Charter Section 3.3).
- **Single active work stream** with a maximum of three concurrent diazotroph executions (Charter Section 3.4).
- **Stateless diazotrophs** operating across the azolla's trust boundary (Charter Section 3.2).
- **Immutable historical artifacts** (Charter Section 3.5).
- **Gated execution** with explicit pause semantics (Charter Section 3.6).
- **Common schemas** from `docs/specs/core_protocol/schemas/`.
- **Workers** selected from `docs/libraries/workers/`.
- **Diazotroph types** selected from `docs/libraries/diazotroph_types/`.

Each deployment instance is identified by a unique `azolla_id`.

## 4. Azolla Type Blueprints

A blueprint is a static, versioned declaration of an azolla type. It specifies which protocol components the type uses. Blueprints are documentation artifacts, not runtime discovery mechanisms.

### CODE_PATCH_AZOLLA (v0.1)

```
category:          EXECUTION
objective_type:    TICKET
diazotroph_types:  [PATCH_DIAZOTROPH]
workers:           [Readiness, Scheduler, Runner, Gate]
spec:              docs/specs/execution_azolla/patch_diazotroph/patch.md
```

### TASK_MGMT_AZOLLA (v0.2)

```
category:          MEMORY_SUPPORT
objective_type:    STANDING
diazotroph_types:  [TRIAGE_DIAZOTROPH, MANIFEST_BUILDER_DIAZOTROPH]
workers:           [Readiness, Scheduler, Runner, Gate,
                    Capture Readiness, Manifest Scheduler, Deadline Monitor]
cross_azolla_reads: [pause_state, objective]
spec:              docs/specs/memory_support_azolla/task_mgmt/task_mgmt.md
```

## 5. Modularity

Protocol components (schemas, workers, diazotroph types) are designed for reuse across azolla types. Creating library items usable by any azolla type is encouraged. An azolla type blueprint selects the subset of components relevant to its category and objective.

Prompt template libraries (deferred to v0.3) will extend this modularity with many-to-many associations between templates and diazotroph types, plus per-deployment customization.
