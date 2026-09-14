# test_log_inverted_near_singular_limits_preserve_direction

- **Instance:** `matplotlib__matplotlib-14623`
- **Test ID:** `matplotlib__matplotlib-14623--a91581f2bc2d6a8d`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 3

## What it checks

Explicitly descending positive log limits must remain descending even when the locator regards the endpoints as singular and automatically expands them.

## Expected behavior

set_xlim emits its singular-limit warning, and the resulting x-axis remains inverted (its left limit is greater than its right limit).

## Test command

`python -m pytest -q lib/matplotlib/tests/test_log_inverted_limits_regression.py::test_log_inverted_near_singular_limits_preserve_direction`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
