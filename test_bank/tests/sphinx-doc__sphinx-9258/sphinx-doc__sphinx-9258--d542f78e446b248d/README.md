# test_info_field_pipe_union_quoted_forward_reference

- **Instance:** `sphinx-doc__sphinx-9258`
- **Test ID:** `sphinx-doc__sphinx-9258--d542f78e446b248d`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 2

## What it checks

Pipe-separated Python type fields follow Python annotation semantics for each operand. In particular, a quoted forward-reference operand denotes the referenced type; the quote characters are not part of the cross-reference target.

## Expected behavior

The rendered union type contains a working link to `types.html#Payload`. The supplied gold parses the quoted operand as the type name `Payload`; the generated candidate instead tries to resolve the literal target `"Payload"`.

## Test command

`cd /testbed && python -m pytest -q tests/test_domain_py.py::test_info_field_pipe_union_quoted_forward_reference`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
