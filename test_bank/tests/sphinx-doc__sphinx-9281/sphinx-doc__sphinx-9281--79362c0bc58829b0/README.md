# test_stringify_signature_enum_uses_custom_str

- **Instance:** `sphinx-doc__sphinx-9281`
- **Test ID:** `sphinx-doc__sphinx-9281--79362c0bc58829b0`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

Enum defaults should preserve the member's public str() representation, including a custom __str__ override, rather than always reconstructing ClassName.member.

## Expected behavior

stringify_signature returns exactly (value=CustomEnum.display_value).

## Test command

`python -m pytest -q tests/test_util_inspect.py::test_stringify_signature_enum_uses_custom_str`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
