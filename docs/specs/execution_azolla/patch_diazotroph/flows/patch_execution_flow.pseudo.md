# Patch Execution Flow

## Intent

Convert a `TICKET_READY` event for an objective into one evaluated PATCH_DIAZOTROPH run, producing a gated code patch.

## Preconditions

- An `objective` exists with `status: TODO` and `objective_type: TICKET`.
- A `TICKET_READY` event has been emitted and is unprocessed.

## Claim/Lease Protocol

Claim/lease protocol details are defined in `docs/specs/core_protocol/polling_workers.pseudo.md` and are normative for this flow.

## Flow

```
1. Pick oldest unprocessed TICKET_READY event.

2. Attempt atomic claim acquisition on event.
   - On failure: skip, continue polling.

3. Load and validate the objective referenced by the event.
   - If the objective is not found:
     - Mark event processed (reason: MISSING_TICKET).
     - Release claim.
     - Exit.
   - If the objective status is not executable (NEW, DONE, or BLOCKED):
     - Mark event processed (reason: NON_EXECUTABLE_STATUS).
     - Release claim.
     - Exit.

4. Check blockers before scheduling work.
   - If unresolved blockers are present:
     - Write pause_state (reason: BLOCKED) with prioritized actions:
       e.g., ["Resolve blockers for <objective_id>", "Re-emit TICKET_READY after resolution"]
     - Set objective status to BLOCKED.
     - Mark event processed (reason: BLOCKED).
     - Release claim.
     - Exit.

5. Build context_snapshot:
   - objective_id: from event
   - full_prompt_text: objective title, acceptance criteria, repo state
   - metadata: { commit_sha: current HEAD, base_branch: target branch }
   - related_yield_refs: []

6. Create workorder:
   - diazotroph_type: PATCH_DIAZOTROPH
   - context_snapshot_id: from step 5
   - branch_name: azolla/<objective_id>
   - budget_ms: deployment-configured execution budget
   - status: CREATED

7. Set objective status to IN_PROGRESS.

8. Mark event processed (reason: SCHEDULED).

9. Release claim.

10. Runner Worker claims workorder:
    - Load workorder and confirm diazotroph_type is PATCH_DIAZOTROPH.
    - Load referenced context_snapshot and objective.
    - Start budget timer using workorder.budget_ms.
    - Checkout repository at the target base reference.
    - Create (or reset) work branch named by workorder.branch_name.
    - Build generation input from objective, acceptance criteria, and full snapshot prompt.
    - Invoke PATCH_DIAZOTROPH to produce a candidate patch.
    - Renew lease periodically during execution; abort safely if renewal fails.
    - If budget expires before completion:
      - Persist output_bundle with empty patch and explanatory notes.
      - Set workorder status to EXECUTED.
      - Exit to gate evaluation.
    - Attempt to apply the candidate patch to the work branch.
    - If apply fails:
      - Persist output_bundle containing the attempted patch and failure notes.
      - Set workorder status to EXECUTED.
      - Exit to gate evaluation.
    - If apply succeeds:
      - Compute normalized git diff from repository state.
      - Generate concise notes describing the changes.
      - Draft PR description text aligned to acceptance criteria.
    - Persist output_bundle:
      - metadata: { branch_name, pr_description_draft, commit_sha }
      - content: normalized diff
      - title: patch summary
      - notes: change description
    - Set workorder status to EXECUTED.

11. Gate Worker evaluates output:
    - Validate output_bundle against schema.
    - Check that content contains a non-empty patch (when runner status is not BUDGET_EXHAUSTED).
    - Extract commit_sha from output_bundle.metadata if present.

12. Gate Worker writes run_record:
    - gate_result: PASS or FAIL
    - gate_reason: validation result description
    - commit_sha: from output_bundle.metadata (if present)

13. On PASS:
    - Transition objective status to DONE.
    - Write pause_state (reason: RUN_COMPLETE) with prioritized actions:
      e.g., ["Review PR on azolla/<objective_id>", "Confirm completion"]

14. On FAIL:
    - Transition objective status to BLOCKED or TODO with blocker_ref.
    - Write pause_state (reason: GATE_FAILED) with prioritized actions:
      e.g., ["Review gate failure reason", "Refine objective and retry"]
```

## Deterministic Guarantees

- One processed `TICKET_READY` event results in at most one created workorder.
- One claimed workorder invocation emits exactly one `output_bundle`.
- Every event terminal path sets `event.processed=true` exactly once.
- Every terminal outcome writes explicit state, not implicit logs.
- The persisted patch content is the normalized repository diff, not raw model text, when apply succeeds.

## Idempotency

- Reprocessing an already processed event is a no-op.
- Re-running scheduler for the same event must not create duplicate workorders.
- Re-running runner for an already executed workorder is a no-op.
- Gate worker must upsert by work_order_id so one workorder maps to one terminal run_record.
