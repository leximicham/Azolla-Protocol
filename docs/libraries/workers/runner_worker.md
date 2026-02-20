# Runner Worker

## Input
- `workorder` artifacts with `status: CREATED`

## Responsibility
- Claim one workorder via atomic claim acquisition.
- Dispatch execution to the diazotroph type specified in `workorder.diazotroph_type`.
- Persist the resulting `output_bundle`.
- Set workorder to `EXECUTED`.

## Used By
- All azolla types.

## Claim/Lease
- Uses the claim/lease protocol defined in `docs/specs/execution_azolla/patch_diazotroph/flows/polling_workers.pseudo.md`.
- Long-running executions must renew the lease periodically.
- If lease renewal fails, the runner must abort without further shared-state writes.
