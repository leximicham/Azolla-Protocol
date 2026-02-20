# Azolla Protocol Glossary

## A

### azolla
A single deployment instance running under the Azolla Protocol. An azolla owns one active objective at a time and is governed by a control plane.

### Azolla Control Plane
The governance and orchestration layer that schedules work, enforces gates, updates durable state, and applies pause semantics.

### Azolla Protocol
The architecture, constraints, and documentation set that defines how azollas, diazotrophs, and substrate artifacts interact.

## B

### blocker_ref
A typed dependency blocker attached to a ticket. Used to encode why a ticket cannot progress and whether the blocker is resolved.

## C

### context_snapshot
An immutable, versioned capture of all run-critical context for a single workorder execution.

## D

### diazotroph
A stateless worker process that performs bounded generation/execution work from a workorder and context snapshot.

## E

### event
A control-plane signal used to trigger deterministic workflow transitions (for example, `TICKET_READY`).

### Exchange Membrane
The policy boundary between worker output and substrate mutation. It validates output, applies gates, and determines state transitions.

## O

### output_bundle
An immutable artifact package produced by one diazotroph execution, including patch content and draft run notes.

## P

### pause_state
A durable paused-state artifact written when execution stops due to completion, gate failure, or budget exhaustion. Contains a short prioritized action list.

## R

### run_record
An immutable audit artifact describing one executed workorder and gate outcome.

## S

### Substrate
The durable storage layer for tickets, snapshots, bundles, run records, events, and pause states.

## T

### ticket
The canonical representation of a single objective, including acceptance criteria, status, blockers, and repository reference.

## W

### workorder
A concrete executable unit created from a ticket, with an assigned runner type, budget, branch target, and snapshot reference.
