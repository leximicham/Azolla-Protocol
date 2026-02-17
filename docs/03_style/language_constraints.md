## Purpose

This file documents language constraints for Azolla Protocol documentation and artifacts. It enforces a strict no-anthropomorphism policy and prescribes terminology so readers correctly understand roles, responsibilities, and control flows.

## Scope

- Applies to all docs in `/docs`, schema descriptions, pseudo-flows, PR text, and Codex task prompts used to materialize MEC artifacts.
- Internal developer notes may use metaphors for brainstorming, but public-facing text and repository artifacts must follow these constraints.

## Core Rules

- Do not ascribe human traits, intentions, beliefs, or agency to systems, azollas, or diazotrophs. Avoid verbs like "decide", "want", "believe", "intend", "choose", "refuse", and "feel" when describing automated components.
- Describe observable behavior, inputs, outputs, constraints, and control-plane actions. Use neutral operational verbs such as "produce", "emit", "return", "fail", "retry", "throttle", "quarantine", and "notify".
- Attribute control and governance to the control plane or Exchange Membrane rather than to diazotrophs.
- Be explicit about human vs. system responsibility. Use "operator", "platform team", or "human reviewer" for people; use "Exchange Membrane" or "control plane" for governance logic.

## Terminology (Preferred Forms)

- `Azolla Protocol` (title case): the overall architecture and set of blueprints.
- `azolla` (lowercase): a specific deployment instance running under the Azolla Protocol.
- `diazotroph` (lowercase): a stateless worker process that performs bounded generation work for an azolla.
- `Exchange Membrane` (title case): the control-plane boundary that mediates between diazotroph execution and azolla state.
- `workorder`, `ticket`, `run_record`, `output_bundle`, `blocker_ref`, `pause_state`: use schema names as-is when referring to data artifacts.

## Capitalization and Synonyms

- Title case and sentence case capitalization are allowed in prose where grammatically appropriate.
- Lowercase canonical forms remain preferred for precision in technical descriptions and schema-oriented examples.
- `worker` is a preferred synonym for diazotroph in neutral operational context.
- `agent` is allowed only in neutral operational context and must not imply goals, strategy, dialogue, or personality.

## Practical Phrasing: Forbidden -> Preferred

- Forbidden: "The diazotroph decided to stop because it was confused."
- Preferred: "The diazotroph produced an invalid output; the Exchange Membrane paused the run and recorded a `blocker_ref`."
- Forbidden: "Azolla wants to prioritize this ticket."
- Preferred: "The control plane scheduled the ticket with higher priority based on configuration."
- Forbidden: "The system learned that..."
- Preferred: "The evaluation observed X in the outputs, and the control plane updated metadata accordingly."

## Prompting and Examples for Codex/Agents

- Prompts and tasks must avoid anthropomorphic framing. When asking Codex to generate text or code, specify constraints and expected outputs (schemas, examples, tests), and include transformation rules for rephrasing where needed.
- Example instruction: "Generate a `blocker_ref` JSON example describing a permission error; do not attribute intent to the diazotroph. Describe observed error, affected `azolla_id`, and `exchange_membrane_action`."

## PR Checklist (Language Constraints)

- Use schema names and canonical terminology from this file and `00_glossary.md`.
- Replace anthropomorphic verbs with operational descriptions.
- Ensure statements of responsibility clearly indicate human actors or control-plane actors.
- Allow title/sentence-case capitalization where appropriate; do not treat capitalization alone as a violation.
- If the term `agent` is used, ensure the surrounding text stays strictly operational and non-anthropomorphic.
- Run a quick pass of the examples above and fix violations.

## Notes and Rationale

These constraints improve clarity, reduce accidental anthropomorphism in system descriptions, and make it easier for reviewers and automated tools to reason about authority, responsibility, and failure modes.

If you need help rewriting text to conform, add a comment to the PR and request a language-constraints pass in Codex review.

