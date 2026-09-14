# test_class_property_direct_entry_point

- **Instance:** `sphinx-doc__sphinx-9461`
- **Test ID:** `sphinx-doc__sphinx-9461--a38e20b9f09ac45a`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

An explicitly requested class property must be documented from its descriptor even when normal attribute access evaluates it to a non-None value. This covers the public `autoproperty` entry point, not only discovery through class members.

## Expected behavior

Autodoc emits a `py:property` directive for `ClassProperty.answer` with `:classmethod:`, `:type: int`, and the getter's docstring. It must not document the evaluated integer value.

## Test command

`cd /testbed && python -m pytest -q tests/test_ext_autodoc_autoproperty.py::test_class_property_direct_entry_point`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
