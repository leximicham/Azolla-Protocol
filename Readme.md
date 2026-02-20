# Azolla Protocol

**License:** [CC-BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/) — Open source for non-commercial use. Attribution required. See [LICENSE](LICENSE) file for details.

## Why this exists

Long-running code projects fail in predictable ways: momentum dies after interruption, complex state becomes impossible to reconstruct, and branching decisions compound into overwhelm. These risks intensify when executive function fluctuates.

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
                     │  └───────────────┬──────────────────┘   │
                     └──────────────────┼──────────────────────┘
                                        │ trust boundary
                        ┌───────────────┼───────────────┐
                        │               │               │
                   ┌────┴────┐     ┌────┴────┐     ┌────┴────┐
                   │  diaz-  │     │  diaz-  │     │  diaz-  │
                   │  otroph │     │  otroph │     │  otroph │
                   └────┬────┘     └────┬────┘     └────┬────┘
                        │               │               │
                   external LLM    external LLM    external LLM
                   provider API    provider API    provider API
```

- The **Control Plane** governs scheduling, enforces gates, and applies pause semantics.
- The **Substrate** is the durable data plane: a Context Store (objectives, snapshots, workorders) and a Yield Store (output bundles, run records).
- The **Exchange Membrane** validates diazotroph output before it can mutate Substrate state. It also validates inbound inter-azolla queries.
- **Diazotrophs** are stateless workers external to the azolla. They operate across the deployment's trust boundary — currently by invoking external LLM provider APIs.

## Azolla types

The protocol defines two categories of azolla (see `docs/04_taxonomy/azolla_taxonomy.md`):

- **Execution azollas** produce concrete deliverables (e.g., code patches). They use `objective_type: TICKET`.
- **Memory-support azollas** maintain and surface state for the operator (e.g., orientation manifests, triage candidates). They use `objective_type: STANDING`.

Multiple azollas can operate concurrently. Each owns its own Substrate. Cross-azolla reads go through the target azolla's Exchange Membrane via a typed query interface (see `docs/04_taxonomy/inter_azolla_protocol.md`).

## One execution cycle

A concrete example: fixing a bug in a CLI tool using a code patch azolla.

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

**6. Gating** — the Exchange Membrane evaluates the output and persists a run record:
```json
{
  "run_id": "run-0042-1",
  "work_order_id": "wo-0042-1",
  "objective_id": "obj-0042",
  "output_id": "out-0042-1",
  "gate_result": "PASS",
  "gate_reason": "patch applies cleanly; acceptance criteria met",
  "commit_sha": "d4e5f6a",
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
- **Diazotrophs** are the nitrogen-fixing organisms. In the protocol, they are the stateless workers that provide the generative capability (LLM-based code generation) while the azolla provides governance, durability, and control.
- The boundary between fern tissue and cyanobacterial cavity is a biological membrane — hence **Exchange Membrane** for the validation layer between internal state and external execution.

The metaphor reinforces a key architectural principle: the generative capability is contained and governed, not autonomous.

## Design center

The protocol is optimized for:

- deterministic progress under constrained attention,
- reconstructable state across interruptions,
- explicit authority boundaries,
- bounded execution and explicit pause semantics.

## Repository contents

This repository defines specification artifacts, protocol blueprints, and component libraries. It does not yet include a production runtime implementation.

- `docs/glossary.md` — canonical terms and concise definitions
- `docs/charter.md` — protocol purpose, invariants, non-goals, and guardrails
- `docs/mec/mec_v0_1.md` — MEC v0.1 scope, artifacts, lifecycle, and acceptance criteria
- `docs/mec/schemas/*.schema.json` — normative artifact schemas (shared across azolla types)
- `docs/mec/flows/*.pseudo.md` — v0.1 deterministic control loop and runner flow pseudocode
- `docs/style/language_constraints.md` — terminology and no-anthropomorphism constraints
- `docs/taxonomy/azolla_taxonomy.md` — azolla categories and type blueprints
- `docs/taxonomy/inter_azolla_protocol.md` — cross-azolla query interface
- `docs/taxonomy/*.schema.json` — inter-azolla query schemas
- `docs/task_mgmt/task_mgmt_v0_2.md` — task management azolla blueprint (v0.2)
- `docs/task_mgmt/flows/*.pseudo.md` — task management flow pseudocode
- `docs/libraries/workers/*.md` — reusable polling worker definitions
- `docs/libraries/diazotroph_types/*.md` — diazotroph type definitions

## Contributing

When updating docs:

1. Keep terminology aligned with `docs/glossary.md`.
2. Preserve charter invariants in `docs/charter.md`.
3. Keep artifacts aligned with schemas in `docs/mec/schemas/`.
4. Follow language constraints in `docs/style/language_constraints.md`.
