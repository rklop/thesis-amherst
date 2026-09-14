# custom_str_enum_default_signature

- **Instance:** `sphinx-doc__sphinx-9281`
- **Test ID:** `sphinx-doc__sphinx-9281--268000d5808b4812`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

EnumClass.MEMBER rendering should remain stable when an Enum defines a custom __str__; using str(member) lets unrelated display customization replace the member identity in a function signature.

## Expected behavior

Signature stringification returns exactly '(state=State.READY)'. Candidate B instead returns '(state=ready)'.

## Test command

`cd /testbed && python -m pytest -q tests/test_util_inspect.py::test_signature_enum_default_ignores_custom_str`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
