# Diazotroph Runner Flow (Pseudo)

## Intent

Describe, in plain language, how a `PATCH_DIAZOTROPH` workorder is executed and how a diazotroph yields an `output_bundle`.

## Narrative Flow

1. **Load required inputs**
   - Read the `workorder` and confirm it targets `PATCH_DIAZOTROPH`.
   - Read the referenced `context_snapshot` and `ticket`.
   - Start a budget timer using `workorder.budget_ms`.

2. **Prepare repository state**
   - Checkout the repository at the ticket's target base reference.
   - Create (or reset) the work branch named by the workorder.

3. **Generate a candidate patch**
   - Build generation input from:
     - ticket objective and acceptance criteria,
     - full snapshot prompt,
     - execution budget.
   - Ask the runner model to produce a candidate patch.

4. **Handle budget exhaustion before finalization**
   - If the elapsed time already exceeds the budget:
     - return runner status `BUDGET_EXHAUSTED`,
     - emit an `output_bundle` with an empty patch and explanatory notes.

5. **Attempt to apply the patch**
   - Try applying the candidate patch to the checked-out branch.
   - If apply fails:
     - return runner status `PATCH_APPLY_FAILED`,
     - emit an `output_bundle` containing the attempted patch and failure notes.

6. **Normalize and summarize successful output**
   - If apply succeeds:
     - compute normalized git diff from repository state,
     - generate concise notes describing what changed,
     - draft PR description text aligned to acceptance criteria.

7. **Return terminal success artifact**
   - Return runner status `COMPLETED`.
   - Emit an `output_bundle` containing:
     - ticket id,
     - branch name,
     - normalized patch,
     - summary notes,
     - PR description draft.

## Deterministic Guarantees

- One runner invocation emits exactly one terminal status.
- One runner invocation emits exactly one `output_bundle`.
- The persisted patch content is the normalized repository diff, not raw model text, when apply succeeds.
