# test_info_field_list_type_containing_quoted_bracket

- **Instance:** `sphinx-doc__sphinx-9230`
- **Test ID:** `sphinx-doc__sphinx-9230--d0f5ae244aa92e4a`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

Inline typed parameter fields must use the final whitespace-delimited token as the parameter name, even when the preceding type contains quoted grouping characters.

## Expected behavior

The field body renders as `bracket (Literal["["]) -- opening bracket`, preserving the complete type and identifying `bracket` as the parameter name.

## Test command

`python -m pytest -q tests/test_domain_py.py::test_info_field_list_type_containing_quoted_bracket`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
