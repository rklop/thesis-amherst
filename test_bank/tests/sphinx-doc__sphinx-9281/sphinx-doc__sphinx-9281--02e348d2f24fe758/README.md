# test_signature_enum_default_with_formattable_name

- **Instance:** `sphinx-doc__sphinx-9281`
- **Test ID:** `sphinx-doc__sphinx-9281--02e348d2f24fe758`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 2

## What it checks

When an Enum exposes a string-subclass name with custom formatting, the generated signature should use that name's standard formatting protocol rather than coercing it with str().

## Expected behavior

stringify_signature() returns "(value=Choice.ValueA)". candidate_b instead renders "(value=Choice.internal-name)" because %s bypasses the custom __format__ behavior.

## Test command

`cd /testbed && python -m pytest -q tests/test_util_inspect.py::test_signature_enum_default_with_formattable_name`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
