# Diazotroph Runner Flow (Pseudo)

## Intent

Describe, in plain language, how a `PATCH_DIAZOTROPH` workorder is executed and how a diazotroph yields an `output_bundle`.

## Worker Model (Polling)

v0.1 assumes a dedicated interval-based **runner worker**:

- polls for executable `workorder.status=CREATED`,
- claims one workorder at a time,
- invokes the diazotroph,
- persists `output_bundle`,
- returns control-plane-readable execution result for gate evaluation.

This keeps scheduling and execution responsibilities separated, while still using simple polling.

Claim/lease protocol details are defined in `polling_workers_v0_1.pseudo.md` and are normative for this flow.

## Narrative Flow

1. **Pick and claim work**
   - Select oldest eligible `workorder.status=CREATED`.
   - Attempt atomic claim acquisition for `work_order_id`.
   - If claim fails, skip this workorder.

2. **Load required inputs**
   - Read the `workorder` and confirm it targets `PATCH_DIAZOTROPH`.
   - Read the referenced `context_snapshot` and `ticket`.
   - Start a budget timer using `workorder.budget_ms`.

3. **Prepare repository state**
   - Checkout the repository at the ticket's target base reference.
   - Create (or reset) the work branch named by the workorder.

4. **Generate a candidate patch**
   - Build generation input from:
     - ticket objective and acceptance criteria,
     - full snapshot prompt,
     - execution budget.
   - Ask the runner model to produce a candidate patch.

5. **Renew lease while executing**
   - If execution exceeds one renewal interval, heartbeat lease.
   - If renewal fails, stop and return without further shared-state mutation.

6. **Handle budget exhaustion before finalization**
   - If the elapsed time already exceeds the budget:
     - return runner status `BUDGET_EXHAUSTED`,
     - emit an `output_bundle` with an empty patch and explanatory notes.

7. **Attempt to apply the patch**
   - Try applying the candidate patch to the checked-out branch.
   - If apply fails:
     - return runner status `PATCH_APPLY_FAILED`,
     - emit an `output_bundle` containing the attempted patch and failure notes.

8. **Normalize and summarize successful output**
   - If apply succeeds:
     - compute normalized git diff from repository state,
     - generate concise notes describing what changed,
     - draft PR description text aligned to acceptance criteria.

9. **Persist and finalize workorder**
   - Emit one terminal `output_bundle`.
   - Set `workorder.status=EXECUTED`.
   - Release claim.
   - Return runner status `COMPLETED|PATCH_APPLY_FAILED|BUDGET_EXHAUSTED` and identifiers needed by control-plane gating.

## Deterministic Guarantees

- One claimed workorder invocation emits exactly one terminal status.
- One claimed workorder invocation emits exactly one `output_bundle`.
- The persisted patch content is the normalized repository diff, not raw model text, when apply succeeds.
