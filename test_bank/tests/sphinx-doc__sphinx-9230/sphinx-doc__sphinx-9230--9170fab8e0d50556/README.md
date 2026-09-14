# test_info_field_list_nested_whitespace_is_not_separator

- **Instance:** `sphinx-doc__sphinx-9230`
- **Test ID:** `sphinx-doc__sphinx-9230--9170fab8e0d50556`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 2

## What it checks

A :param argument uses inline TYPE NAME syntax only when separated by top-level whitespace. Whitespace inside parentheses is part of the argument; without a top-level separator, the complete argument remains the parameter name and must match a separate :type field.

## Expected behavior

The rendered Parameters field body is exactly `dict(str, str) (Mapping) -- optional metadata`. Candidate A instead splits at the whitespace inside the parentheses and rearranges the name and type.

## Test command

`cd /testbed && python -m pytest -q tests/test_domain_py.py::test_info_field_list_nested_whitespace_is_not_separator`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
