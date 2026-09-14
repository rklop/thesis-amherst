# test_rolling_iter_centered_uses_current_data

- **Instance:** `pydata__xarray-6744`
- **Test ID:** `pydata__xarray-6744--d85e59a92fde0900`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

Centered manual iteration should apply each centered positional bound to the DataArray's current backing data, rather than clipping it to the potentially stale length captured by window_labels.

## Expected behavior

For the three existing iterator labels, manual window means are [0.5, 1.0, 2.0], matching the first three values of rolling_obj.mean(). In particular, the final centered window is positions 1:4, containing [1, 2, 3].

## Test command

`pytest -q xarray/tests/test_rolling.py::TestDataArrayRolling::test_rolling_iter_centered_uses_current_data`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
