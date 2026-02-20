# Deadline Monitor Worker

## Input
- `objective` artifacts with urgency values and deployment-configurable urgency-to-deadline mappings

## Responsibility
- Poll objectives and apply the deployment's urgency-to-deadline configuration to determine overdue items.
- Write local `pause_state` for overdue items with recommended operator actions.
- No cross-azolla writes. The operator acts manually to create blockers or update objectives in target azollas.

## Used By
- Task management azolla (available to all azolla types).

## Notes
- This is control-plane logic, not a diazotroph execution. It does not count toward the concurrency limit.
- Urgency values (0-4) are defined on the objective. Each deployment configures what these values map to in terms of time-based deadlines (e.g., urgency 0 = immediate action required, urgency 4 = address when spare cycles are available).
