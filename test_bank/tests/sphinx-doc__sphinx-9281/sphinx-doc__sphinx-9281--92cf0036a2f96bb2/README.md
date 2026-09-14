# test_stringify_signature_enum_with_overridden_name

- **Instance:** `sphinx-doc__sphinx-9281`
- **Test ID:** `sphinx-doc__sphinx-9281--92cf0036a2f96bb2`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 3

## What it checks

An Enum default must be rendered from its declared symbolic member identity, not from the member's overridable `name` attribute. Otherwise a valid Enum subclass can produce a misleading signature.

## Expected behavior

The signature is exactly `(value=CustomNameEnum.VALUE)`. candidate_a derives the declared member identifier and should pass; candidate_b accesses the overridden `name` property and produces `(value=CustomNameEnum.overridden)`.

## Test command

`cd /testbed && python -m pytest -q tests/test_util_inspect.py::test_stringify_signature_enum_with_overridden_name`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
