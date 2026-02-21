# The Azolla Protocol Charter (v1.0)

## 1. Purpose

The Azolla Protocol defines a control-plane architecture for orchestrating stateless LLM workers in a way that:

- Preserves executive continuity across energy variability
- Converts intent into structured, bounded execution
- Maintains durable, reconstructable state
- Prevents autonomy drift and architectural sprawl

The Azolla Protocol is structured to function as a cognitive prosthesis - an externalized executive function designed to sustain long-running code projects when motivation and activation fluctuate.

The system is designed for execution under constraint, not simulation of collaboration.

---

## 2. Problem Statement

Executive dysfunction introduces three systemic risks in long-term technical work:

- Loss of momentum after interruption
- Inability to rehydrate complex project state
- Overwhelm caused by branching decisions and open loops

The Azolla Protocol addresses these risks by:

- Centralizing state in a durable substrate
- Constraining execution to bounded, stateless workers
- Enforcing deterministic control loops
- Limiting concurrency and objective scope
- Surfacing prioritized next actions
- Automatically pausing when complexity escalates

---

## 3. Architectural Principles

### 3.1 Authority Separation

An azolla comprises three internal components:
 
- Governance resides in the Azolla Control Plane
- Durable state resides in the Azolla Substrate
- Boundary enforcement resides in the Exchange Membrane
 
Diazotrophs are external to the azolla and operate across its trust boundary:
 
- Execution resides in diazotrophs
- Diazotrophs possess no structural authority

### 3.2 Stateless Execution

- Diazotrophs are stateless
- No persistent diazotroph memory is permitted
- All execution must be reconstructable from Substrate state and versioned prompts

### 3.3 Objective Discipline

- Each azolla operates on exactly one concrete objective
- Multi-objective optimization within a single azolla is prohibited
- Re-planning is permitted only within the scope of the defined objective
- Objective completion requires explicit user confirmation

### 3.4 Concurrency Limits

- An azolla maintains a single active work stream
- No more than three diazotroph executions may run concurrently
- These limits exist to prevent cognitive and architectural overload

### 3.5 Substrate Integrity

- All historical artifacts are immutable
- Context and output are versioned and durable
- Schema mutation is not permitted at runtime
- State is tiered but never silently discarded

### 3.6 Gated Execution

- No direct writes to production branches without explicit gating
- No unbounded tool access
- No invisible background progress beyond declared budget windows
- All autonomous runs must declare explicit stop conditions

---

## 4. Non-Goals

The Azolla Protocol does not support:

- Recursive diazotroph spawning
- Control-plane modification by diazotroph output
- Dynamic diazotroph type creation
- Multi-objective optimization within a single azolla
- Social agent interaction models
- Persistent diazotroph memory
- Mutable historical artifacts
- High-energy interaction requirements for resumption
- Unclear stop conditions
- Unbounded or hidden autonomous execution

---

## 5. Prosthesis Guardrails

To function as a cognitive prosthesis, deployments under the Azolla Protocol must:

- Assume the user cannot maintain high-detail context between sessions
- Surface prioritized action lists rather than branching trees
- Prevent runaway analysis or combinatorial decision expansion
- Enter a paused state when overwhelm signals are detected
- Guarantee deterministic resume semantics

Failure to maintain these guardrails invalidates the prosthesis function of the system.

---

## 6. Overwhelm Detection and Pause Semantics

The Azolla Control Plane must monitor for:

- Excessive re-planning cycles
- Lack of output progress across bounded iterations
- Retry storms
- Context expansion beyond defined thresholds
- Budget exhaustion

Upon detection, the deployment must:

- Enter a paused state
- Surface a concise summary
- Present a single prioritized next action

---

## 7. Completion Semantics

An azolla is considered complete only when:

- The objective is marked done
- The user explicitly confirms completion

Completion is never inferred autonomously.

---

## 8. Versioning and Incremental Evolution

The Azolla Protocol may evolve through MVP and staged releases.

However, v1.0 must enforce:

- All invariants listed in this Charter
- All non-goals as hard constraints
- All prosthesis guardrails as operational requirements

Any future relaxation must be explicit and versioned.

---

## 9. Documentation and Language Constraints

### 9.1 No Anthropomorphism

Azollas and diazotrophs must not be described using language that implies:

- Intent
- Personality
- Desire or preference
- Emotional or cognitive states
- Independent agency beyond defined execution scope

Permitted language must describe structural roles, deterministic execution, and constrained system behavior.

### 9.2 Deployment Neutrality

An azolla is:

- A deployment instance comprising the Control Plane, Substrate, and Exchange Membrane
- A configuration of infrastructure
- A bounded execution system

An azolla is not:

- A collaborator
- An assistant
- A partner
- A personality

### 9.3 `diazotroph` Language Discipline

diazotrophs must be described as:

- Stateless workers
- Execution units
- Bounded generators

They must not be described as:

- Agents with goals
- Actors with strategy
- Participants in dialogue

Execution occurs through control logic and bounded invocation, not negotiation.

---

## 10. Design Position

The Azolla Protocol is:

- A deterministic progress engine
- A bounded orchestration framework
- A durability-first architecture
- A cognitive prosthesis

It is not:

- An agent society
- A creative companion
- A self-modifying system
- An autonomous goal generator

The system exists to convert structured intent into sustained execution under constraint.
