# test_setup_show_truncates_long_bytes_parameter

- **Instance:** `pytest-dev__pytest-7205`
- **Test ID:** `pytest-dev__pytest-7205--f5e805abf23db4e8`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

The --setup-show representation of a raw fixture parameter should remain short when the parameter has a long repr; safely calling repr is insufficient if the default 240-character limit allows verbose values through unchanged.

## Expected behavior

The test passes, and both fixture-action entries contain an ellipsized representation such as b'01234567890123456...1234567890123456789', shorter than the complete bytes repr.

## Test command

`cd /testbed && python -m pytest -q testing/test_setuponly.py::test_setup_show_truncates_long_bytes_parameter`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
