# test_info_field_list_parenthesized_untyped_parameter

- **Instance:** `sphinx-doc__sphinx-9230`
- **Test ID:** `sphinx-doc__sphinx-9230--012d2b3675f1217b`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

In the one-argument, untyped `:param name:` form, whitespace inside balanced parentheses belongs to the parameter label. Only whitespace outside brackets may separate an inline type from a parameter name.

## Expected behavior

The transformed Parameters field body reads `(x, y) -- coordinates`; `(x, y)` remains one parameter label and no type annotation is inferred.

## Test command

`cd /testbed && python -m pytest -q tests/test_domain_py.py::test_info_field_list_parenthesized_untyped_parameter`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
