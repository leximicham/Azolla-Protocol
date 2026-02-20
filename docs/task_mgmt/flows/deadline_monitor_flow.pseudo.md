# Deadline Monitor Flow

## Intent

Monitor objectives for urgency-derived deadline overruns and surface overdue items via local `pause_state`.

## Preconditions

- At least one objective exists with urgency values in the range 0-4.
- A deployment-configurable urgency-to-deadline mapping is available.

## Urgency-to-Deadline Mapping

Each deployment configures what urgency values (0-4) map to in terms of time-based deadlines. Example:

| Urgency | Example mapping |
|---------|----------------|
| 0 | Immediate action required; do not stop until addressed |
| 1 | Due within 24 hours |
| 2 | Due within 1 week |
| 3 | Due within 1 month |
| 4 | Address when spare cycles are available; no fixed deadline |

The mapping is deployment-specific and not enforced by the protocol schema.

## Flow

```
1. Poll objectives from the local Substrate.
   - Read all objectives with status not DONE.

2. For each objective:
   - Look up urgency value.
   - Apply deployment urgency-to-deadline mapping.
   - If urgency = 4 (no fixed deadline), skip.
   - Calculate deadline based on objective.updated_at + mapped duration.

3. For each objective where the calculated deadline is exceeded:
   - Write pause_state:
     - objective_id: the overdue objective's identifier
     - azolla_id: the task management azolla's identifier
     - reason: RUN_COMPLETE (deadline monitoring is control-plane logic, not a failed run)
     - prioritized_actions: e.g.,
       ["Review overdue objective <objective_id> (urgency <N>)",
        "Update objective urgency or address the item"]

4. No cross-azolla writes. If the overdue objective belongs to another azolla
   (discovered via cross-azolla query), the pause_state is still written
   locally. The operator acts manually to create blockers or update objectives
   in the target azolla.
```

## Notes

- This is control-plane logic, not a diazotroph execution. It does not create workorders, output_bundles, or run_records. It does not count toward the concurrency limit.
- The deadline monitor worker polls at an implementation-defined interval.
- Correctness does not depend on exact poll interval values.
