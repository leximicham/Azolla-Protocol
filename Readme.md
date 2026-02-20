# Azolla Protocol

## Why this exists
Azolla Protocol is a durability-first control-plane architecture for executing bounded software tasks with stateless workers (`diazotrophs`).

This repository currently contains the foundational protocol documentation, including:

- Charter and invariants
- Core terminology
- Minimal Executable Core (MEC) model
- JSON schemas for persisted artifacts
- Pseudo-flows for control-plane and runner behavior
- Language constraints for documentation and prompts

## Documentation Index

- `docs/01_charter.md` — protocol purpose, invariants, non-goals, and guardrails.
- `docs/00_glossary.md` — canonical terms and concise definitions.
- `docs/02_mec/mec_v0_1.md` — MEC v0.1 scope, artifacts, lifecycle, and acceptance criteria.
- `docs/02_mec/schemas/*.schema.json` — normative artifact schemas.
- `docs/02_mec/flows/*.pseudo.md` — deterministic control loop and runner flow pseudocode.
- `docs/03_style/language_constraints.md` — terminology and no-anthropomorphism constraints.

## Design Center

The protocol is optimized for:

- deterministic progress under constrained attention,
- reconstructable state across interruptions,
- explicit authority boundaries,
- bounded execution and explicit pause semantics.

## Current Status

This repository defines specification artifacts and protocol blueprints. It does not yet include a production runtime implementation.

## Contributing

When updating docs:

1. Keep terminology aligned with `docs/00_glossary.md`.
2. Preserve charter invariants in `docs/01_charter.md`.
3. Keep MEC artifacts aligned with schemas in `docs/02_mec/schemas/`.
4. Follow language constraints in `docs/03_style/language_constraints.md`.

**License:** [CC-BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/) — Open source for non-commercial use. Attribution required. See [LICENSE](LICENSE) file for details.