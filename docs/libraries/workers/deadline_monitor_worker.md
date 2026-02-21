# Deadline Monitor Worker

## Input
- `objective` artifacts with urgency values and deployment-configurable urgency-to-deadline mappings

## Responsibility
- Poll objectives and apply the deployment's urgency-to-deadline configuration to determine overdue items.
- Write local `pause_state` for overdue items with recommended operator actions.
- No cross-azolla writes. The operator acts manually to create blockers or update objectives in target azollas.

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
     - azolla_id: local azolla identifier
     - reason: RUN_COMPLETE
     - prioritized_actions: e.g.,
       ["Review overdue objective <objective_id> (urgency <N>)",
        "Update objective urgency or address the item"]

4. No cross-azolla writes. If the overdue objective belongs to
   another azolla (discovered via cross-azolla query), the
   pause_state is still written locally.
```

## Diagram

See `docs/diagrams/deadline_monitor_worker.puml`.

## Used By
- Task management azolla (available to all azolla types).

## Notes
- This is control-plane logic, not a diazotroph execution. It does not count toward the concurrency limit.
- Urgency values (0-4) are defined on the objective. Each deployment configures what these values map to in terms of time-based deadlines (e.g., urgency 0 = immediate action required, urgency 4 = address when spare cycles are available).
