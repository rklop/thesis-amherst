# test_info_field_list_union_preserves_literal_pipe

- **Instance:** `sphinx-doc__sphinx-9258`
- **Test ID:** `sphinx-doc__sphinx-9258--5e2a8ba4bb9edf85`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

Union separators in Python type fields must be recognized according to expression syntax. A `|` inside a nested string literal is literal data, not a union operator or type-reference boundary.

## Expected behavior

Only `Literal` and `str` become type cross-reference targets. The nested `left|right` value remains ordinary literal content and is not split into spurious type links.

## Test command

`python -m pytest -q tests/test_domain_py.py::test_info_field_list_union_preserves_literal_pipe`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
