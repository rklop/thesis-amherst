# test_rolling_iter_center_numpy_integer_window

- **Instance:** `pydata__xarray-6744`
- **Test ID:** `pydata__xarray-6744--6cea7994cd443ffd`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

Manual centered rolling iteration must accept NumPy integer window sizes just like Python integers, normalizing them before calculating positional slice bounds. Centering must also produce the correct partial windows at both boundaries.

## Expected behavior

The iterator yields centered windows whose means are [1.5, 2.0, 3.0, 4.0, 4.5].

## Test command

`python -m pytest -q xarray/tests/test_rolling.py::TestDataArrayRolling::test_rolling_iter_center_numpy_integer_window`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
