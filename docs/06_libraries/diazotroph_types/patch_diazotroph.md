# PATCH_DIAZOTROPH

## Description
A stateless worker that generates candidate code patches from an objective and repository context.

## Input Contract
Receives a `context_snapshot` containing:
- Objective title and acceptance criteria (from the `objective` artifact)
- Repository state (`repo_commit_sha`, base branch)
- Full prompt text with generation instructions

## Output Contract
Produces one `output_bundle` with:
- `metadata`: `{ "branch_name": "<string>", "pr_description_draft": "<string>", "commit_sha": "<string>" }`
- `content`: git patch text (unified diff format)
- `title`: summary of the patch
- `notes`: execution notes describing the generation approach and any observations

## Execution
1. Checkout repository at `repo_commit_sha`.
2. Create or reset work branch from base branch.
3. Build generation input from objective and snapshot.
4. Invoke external LLM provider API with bounded budget.
5. Attempt patch application to the work branch.
6. Normalize git diff if patch applies successfully.
7. Emit `output_bundle`.

## Gate Criteria
The Exchange Membrane evaluates:
- Patch applies cleanly to the base branch.
- `metadata.branch_name` and `metadata.pr_description_draft` are non-empty.
- Output does not exceed the declared budget window.

## Available To
- Execution azollas (CODE_PATCH_AZOLLA).
