# Runner Worker

## Input
- `workorder` artifacts with `status: CREATED`

## Responsibility
- Claim one workorder via atomic claim acquisition.
- Dispatch execution to the diazotroph type specified in `workorder.diazotroph_type`.
- Persist the resulting `output_bundle`.
- Set workorder to `EXECUTED`.

## Flow

```
1. Pick the oldest eligible workorder with status=CREATED.
   - If none exists, do nothing for this tick.

2. Attempt atomic claim acquisition on the workorder.
   - On failure: skip, continue polling.

3. Load the workorder and referenced context_snapshot.
   - Start budget timer using workorder.budget_ms.

4. Dispatch execution to the diazotroph type specified
   in workorder.diazotroph_type.
   - The diazotroph produces an output_bundle.

5. Renew lease periodically during execution.
   - If renewal fails: abort without further shared-state writes.

6. If budget expires before completion:
   - Persist output_bundle with explanatory notes.
   - Set workorder status to EXECUTED.
   - Release claim.
   - Exit.

7. Persist terminal output_bundle.

8. Set workorder status to EXECUTED.

9. Release claim.
```

## Used By
- All azolla types.

## Claim/Lease
- Uses the claim/lease protocol defined in `docs/specs/core_protocol/polling_workers.pseudo.md`.
- Long-running executions must renew the lease periodically.
- If lease renewal fails, the runner must abort without further shared-state writes.
