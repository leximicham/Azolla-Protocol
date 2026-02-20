# TRIAGE_DIAZOTROPH

## Description
A stateless worker that converts raw `capture_buffer` contents into structured candidate objectives.

## Input Contract
Receives a `context_snapshot` containing:
- One or more `capture_buffer` artifacts (raw_content and source_label)
- Full prompt text with triage instructions and output format requirements

## Output Contract
Produces one `output_bundle` per batch of captures with:
- `metadata`: `{ "capture_ids": ["<string>", ...], "suggested_target_azolla_type": "<string>" }`
- `content`: structured candidate objective description (title, acceptance criteria, rationale)
- `title`: summary of the triage result
- `notes`: confidence notes, ambiguities, or items requiring operator clarification

## Execution
1. Parse capture_buffer contents from the context_snapshot.
2. Invoke external LLM provider API with bounded budget.
3. Produce structured candidate objective for each capture or group of related captures.
4. Emit `output_bundle`.

## Gate Criteria
The Exchange Membrane evaluates:
- Output validates against the `output_bundle` schema.
- `metadata.capture_ids` references valid capture_buffer identifiers.
- `content` contains non-empty candidate title and acceptance criteria.

## Available To
- Task management azolla (available to all azolla types).
