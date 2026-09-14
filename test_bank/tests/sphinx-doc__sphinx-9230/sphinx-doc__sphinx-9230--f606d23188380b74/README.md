# test_info_field_list_untyped_param

- **Instance:** `sphinx-doc__sphinx-9230`
- **Test ID:** `sphinx-doc__sphinx-9230--f606d23188380b74`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

The inline type in `:param [type] name:` is optional. A one-token field argument is solely the parameter name, not a declaration with an empty type.

## Expected behavior

The rendered parameter paragraph is exactly `value -- description`, without empty type parentheses.

## Test command

`cd /testbed && python -m pytest -q tests/test_domain_py.py::test_info_field_list_untyped_param`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
