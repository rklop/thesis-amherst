# test_nonsingular_no_positive_without_axis

- **Instance:** `matplotlib__matplotlib-14623`
- **Test ID:** `matplotlib__matplotlib-14623--286f848503c82727`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

An unattached LogLocator must handle an entirely non-positive range without consulting Axis state. Because no positive endpoint exists, nonsingular should emit its existing warning and return the standard positive fallback directly.

## Expected behavior

The call emits a `UserWarning` indicating that the data has no positive values and returns `(1, 10)`. Candidate_a instead accesses `self.axis.get_minpos()` first and raises `AttributeError`; candidate_b reaches the axis-independent fallback.

## Test command

`python -m pytest -q lib/matplotlib/tests/test_ticker.py::TestLogLocator::test_nonsingular_no_positive_without_axis`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
