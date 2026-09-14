# test_setuponly_imports_respect_configured_groups

- **Instance:** `pytest-dev__pytest-7205`
- **Test ID:** `pytest-dev__pytest-7205--da906c5d5fe7cecc`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 2

## What it checks

The repository classifies `_pytest` as a local import, so it must be separated from the third-party `pytest` import. This is the sole observable disagreement between the otherwise runtime-equivalent patches.

## Expected behavior

`import pytest` is followed by a blank line before `from _pytest._io.saferepr import saferepr`; the targeted test exits with status 0.

## Test command

`cd /testbed && python -m pytest -q testing/test_setuponly.py::test_setuponly_imports_respect_configured_groups`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
