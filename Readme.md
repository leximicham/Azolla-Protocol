# The Azolla Protocol

**License:** [CC-BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/) — Open source for non-commercial use. Attribution required. See [LICENSE](LICENSE) file for details.

## Contents

- [Why this exists](#why-this-exists)
- [How it works](#how-it-works)
- [Azolla types](#azolla-types)
- [One execution cycle](#one-execution-cycle)
- [Naming](#naming)
- [Design center](#design-center)
- [Reading guide](#reading-guide)
- [Repository contents](#repository-contents)
- [Contributing](#contributing)

## Why this exists

Long-running projects fail in predictable ways: momentum dies after interruption, complex state becomes impossible to reconstruct, and branching decisions compound into overwhelm. These risks intensify when executive function fluctuates.

The Azolla Protocol is a cognitive prosthesis — an externalized executive function system that converts structured intent into sustained execution under constraint. It is a blueprint library for constructing azolla deployments: each deployment is composed from shared schemas, reusable workers, and statically-defined diazotroph types, orchestrated through a durable, deterministic control plane that preserves progress across any interruption.

## How it works

Each deployment (an **azolla**) owns one objective at a time and comprises three internal components:

```
                          ┌─────────────────────────────────────────┐
                          │                azolla                   │
                          │                                         │
                          │  ┌──────────────┐    ┌───────────────┐  │
                          │  │ Control Plane│    │   Substrate   │  │
                          │  │              │    │               │  │
                          │  │  governance, │    │  Context      │  │
                          │  │  scheduling, │    │  Store        │  │
                          │  │  gating      │    │  ──────────   │  │
                          │  │              │    │  Yield        │  │
                          │  │              │    │  Store        │  │
                          │  └──────────────┘    └───────────────┘  │
                          │                                         │
                          │  ┌──────────────────────────────────┐   │
                          │  │        Exchange Membrane         │   │
                          │  │  validates, gates, transitions   │   │
                          │  └──┬────────────────┬──────────────┘   │
                          └─────┼────────────────┼──────────────────┘
                                │                │
               inter-azolla     │                │ trust boundary
               query interface  │                │
                                │    ┌───────────┼───────────┐
        ┌───────────┐           │    │           │           │
        │  other    │◄──────────┘    │           │           │
        │  azollas  │ substrate  ┌───┴───┐  ┌───┴───┐  ┌───┴───┐
        │  (read    │ query      │ diaz- │  │ diaz- │  │ diaz- │
        │   only)   │            │ otroph│  │ otroph│  │ otroph│
        └───────────┘            └───┬───┘  └───┬───┘  └───┬───┘
                                     │          │          │
                                external    external   external
                                LLM API     LLM API    LLM API
```

- The **Control Plane** governs scheduling, enforces gates, and applies pause semantics.
- The **Substrate** is the durable data plane: a Context Store (objectives, snapshots, workorders) and a Yield Store (output bundles, run records).
- The **Exchange Membrane** validates diazotroph output before it can mutate Substrate state. It also validates inbound inter-azolla queries.
- **Diazotrophs** are stateless workers external to the azolla. They operate across the deployment's trust boundary — currently by invoking external LLM provider APIs.

## Azolla types

The protocol defines two categories of azolla (see `docs/specs/core_protocol/azolla_taxonomy.md`):

- **Execution azollas** produce concrete deliverables (e.g., code patches, documents, analyses). They use `objective_type: TICKET`.
- **Memory-support azollas** maintain and surface state for the operator (e.g., orientation manifests, triage candidates). They use `objective_type: STANDING`.

Multiple azollas can operate concurrently. Each owns its own Substrate. Cross-azolla reads go through the target azolla's Exchange Membrane via a typed query interface (see `docs/specs/core_protocol/inter_azolla_protocol.md`).

## One execution cycle

The following example walks through a single execution cycle using a code patch azolla — one of many possible azolla types. Other execution azollas could produce documents, analyses, or other deliverables using the same lifecycle.

**Example: fixing a bug in a CLI tool.**

**1. Objective** — the operator creates a single objective:
```json
{
  "objective_id": "obj-0042",
  "azolla_id": "az-main",
  "title": "Fix --verbose flag being ignored in cli parse module",
  "acceptance_criteria": "cli parse respects --verbose; existing tests pass; one new test covers the flag",
  "objective_type": "TICKET",
  "status": "NEW",
  "importance": 2,
  "urgency": 1,
  "blocked_reason": null,
  "blocked_on": [],
  "metadata": {
    "repo_ref": "github.com/example/cli-tool",
    "base_branch": "main"
  },
  "created_at": "2025-06-01T10:00:00Z",
  "updated_at": "2025-06-01T10:00:00Z"
}
```

**2. Human review** promotes `NEW` to `TODO`, then the Control Plane emits a `TICKET_READY` event.

**3. Context Snapshot** — the scheduler captures everything needed for a self-contained run:
```json
{
  "snapshot_id": "snap-0042-1",
  "objective_id": "obj-0042",
  "metadata": {
    "commit_sha": "a1b2c3d",
    "base_branch": "main"
  },
  "related_yield_refs": [],
  "full_prompt_text": "Fix the --verbose flag in src/cli/parse.rs. The flag is accepted by the argument parser but never propagated to the logger configuration. Acceptance criteria: cli parse respects --verbose; existing tests pass; one new test covers the flag.",
  "created_at": "2025-06-01T10:05:00Z"
}
```

**4. Workorder** — bounded execution is dispatched:
```json
{
  "work_order_id": "wo-0042-1",
  "objective_id": "obj-0042",
  "diazotroph_type": "PATCH_DIAZOTROPH",
  "context_snapshot_id": "snap-0042-1",
  "branch_name": "azolla/obj-0042",
  "budget_ms": 120000,
  "status": "CREATED",
  "created_at": "2025-06-01T10:05:01Z"
}
```

**5. Diazotroph execution** — a stateless worker generates a candidate patch and produces an output bundle:
```json
{
  "output_id": "out-0042-1",
  "objective_id": "obj-0042",
  "title": "Fix --verbose flag propagation in parse_args",
  "metadata": {
    "branch_name": "azolla/obj-0042",
    "pr_description_draft": "Fixes #42: --verbose flag was not propagated from parse_args to the logger config.",
    "commit_sha": "d4e5f6a"
  },
  "content": "diff --git a/src/cli/parse.rs b/src/cli/parse.rs\n...",
  "notes": "Added verbose flag propagation in parse_args; new test in tests/cli_parse_test.rs.",
  "created_at": "2025-06-01T10:06:30Z"
}
```

**6. Gating** — the Gate Worker evaluates the output against Exchange Membrane gate criteria and persists a run record:
```json
{
  "run_id": "run-0042-1",
  "work_order_id": "wo-0042-1",
  "objective_id": "obj-0042",
  "output_id": "out-0042-1",
  "gate_result": "PASS",
  "gate_reason": "patch applies cleanly; acceptance criteria met",
  "created_at": "2025-06-01T10:06:35Z"
}
```

**7. Pause** — the system writes a pause state with prioritized next actions and stops:
```json
{
  "objective_id": "obj-0042",
  "azolla_id": "az-main",
  "reason": "RUN_COMPLETE",
  "prioritized_actions": ["Review PR on azolla/obj-0042", "Confirm completion"],
  "created_at": "2025-06-01T10:06:36Z"
}
```

The operator can return hours or days later and reconstruct exactly where things stand from persisted artifacts alone.

## Naming

The project borrows from symbiotic biology:

- **Azolla** is a genus of aquatic ferns that host nitrogen-fixing cyanobacteria in their leaf cavities. The fern provides structure and environment; the bacteria provide a specific metabolic capability. Neither functions the same way alone.
- **Diazotrophs** are the nitrogen-fixing organisms. In the protocol, they are the stateless workers that provide the generative capability (LLM-based generation) while the azolla provides governance, durability, and control.
- The boundary between fern tissue and cyanobacterial cavity is a biological membrane — hence **Exchange Membrane** for the validation layer between internal state and external execution.

The metaphor reinforces a key architectural principle: the generative capability is contained and governed, not autonomous.

## Design center

The protocol is optimized for:

- deterministic progress under constrained attention,
- reconstructable state across interruptions,
- explicit authority boundaries,
- bounded execution and explicit pause semantics.

## Reading guide

The repository is layered. Start from governance and work outward toward concrete azolla types.

**1. Governance and vocabulary** — Read these first to establish the invariants and terminology that the rest of the repo assumes.

- `docs/charter.md` — architectural principles, non-goals, and prosthesis guardrails. Every spec decision traces back to a charter section.
- `docs/glossary.md` — canonical definitions for every protocol term. If a word appears in backticks in a spec file, its definition is here.
- `docs/style/language_constraints.md` — the no-anthropomorphism policy and approved verb list. Useful context for understanding the prose style throughout.

**2. Core protocol** — The shared infrastructure that all azolla types build on.

- `docs/specs/core_protocol/azolla_taxonomy.md` — the two azolla categories (execution, memory-support) and the blueprint format. Read this before either azolla spec.
- `docs/specs/core_protocol/polling_workers.pseudo.md` — the generic worker polling model, claim/lease protocol, and idempotency rules. Both azolla specs reference this file as normative.
- `docs/specs/core_protocol/inter_azolla_protocol.md` — the cross-azolla read interface. Relevant for the task management spec (Track B: manifest build).
- `docs/specs/core_protocol/schemas/` — JSON schemas defining every artifact. `domain/` contains intra-azolla artifacts; `transport/` contains inter-azolla envelopes. Schemas are authoritative when prose diverges.

**3. Library components** — Reusable building blocks selected by azolla type blueprints.

- `docs/libraries/workers/*.md` — seven polling worker definitions with pseudocode flows and state machine diagrams. The four core workers (readiness, scheduler, runner, gate) are generic; the three specialized workers (capture readiness, manifest scheduler, deadline monitor) originated in the task management azolla but are available to all types.
- `docs/libraries/diazotroph_types/*.md` — three stateless diazotroph type definitions with input/output contracts and gate criteria.

**4. Azolla type specs** — Concrete azolla types that compose core protocol, library workers, and diazotroph types into a working lifecycle.

- `docs/specs/execution_azolla/patch_diazotroph/patch.md` — the v0.1 code patch azolla. Start here for a minimal end-to-end understanding of the objective lifecycle (NEW through DONE/BLOCKED to pause).
- `docs/specs/memory_support_azolla/task_mgmt/task_mgmt.md` — the v0.2 task management azolla. Introduces standing objectives, capture buffers, cross-azolla queries, and three concurrent tracks.
- Each spec has a `flows/` subdirectory with pseudocode for the end-to-end track flow and local worker topology.

**5. Diagrams** — PlantUML state machines in `docs/diagrams/`. Per-worker diagrams show generic worker state transitions; per-track diagrams show the full lifecycle of a specific azolla track including azolla-specific artifact states.

## Repository contents

This repository defines specification artifacts, protocol blueprints, and component libraries. It does not yet include a production runtime implementation.

- `docs/glossary.md` — canonical terms and concise definitions
- `docs/charter.md` — protocol purpose, invariants, non-goals, and guardrails
- `docs/specs/core_protocol/` — core protocol, shared schemas, and inter-azolla interface
  - `azolla_taxonomy.md` — azolla categories and type blueprints
  - `inter_azolla_protocol.md` — cross-azolla query interface
  - `polling_workers.pseudo.md` — shared worker polling model and claim/lease protocol
  - `schemas/domain/*.schema.json` — intra-azolla domain artifact schemas (objective, workorder, etc.)
  - `schemas/transport/*.schema.json` — inter-azolla transport schemas (substrate_query, substrate_query_response)
- `docs/specs/execution_azolla/patch_diazotroph/` — v0.1 minimal executable core (code patch azolla)
  - `patch.md` — patch diazotroph scope, artifacts, lifecycle, and acceptance criteria
  - `flows/patch_execution_flow.pseudo.md` — end-to-end execution track flow
  - `flows/patch_polling_workers.pseudo.md` — local worker topology
- `docs/specs/memory_support_azolla/task_mgmt/` — task management azolla specification
  - `task_mgmt.md` — task management azolla blueprint
  - `flows/*.pseudo.md` — track flows (triage, manifest, deadline) and local worker topology
- `docs/diagrams/*.puml` — PlantUML state machine diagrams (per-worker and per-track)
- `docs/style/language_constraints.md` — terminology and no-anthropomorphism constraints
- `docs/libraries/workers/*.md` — reusable polling worker definitions
- `docs/libraries/diazotroph_types/*.md` — diazotroph type definitions

## Contributing

When updating docs:

1. Keep terminology aligned with `docs/glossary.md`.
2. Preserve charter invariants in `docs/charter.md`.
3. Keep artifacts aligned with schemas in `docs/specs/core_protocol/schemas/domain/` and `docs/specs/core_protocol/schemas/transport/`.
4. Follow language constraints in `docs/style/language_constraints.md`.
