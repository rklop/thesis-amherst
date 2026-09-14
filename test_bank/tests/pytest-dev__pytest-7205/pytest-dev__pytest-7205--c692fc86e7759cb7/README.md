# test_setuponly_keeps_local_imports_in_one_group

- **Instance:** `pytest-dev__pytest-7205`
- **Test ID:** `pytest-dev__pytest-7205--c692fc86e7759cb7`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

The project classifies both pytest and _pytest as local imports, so they must remain in one contiguous import group. The supplied patches are otherwise behaviorally identical.

## Expected behavior

The saferepr import immediately follows import pytest without an intervening blank line.

## Test command

`cd /testbed && pytest -q testing/test_setuponly.py::test_setuponly_keeps_local_imports_in_one_group`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
