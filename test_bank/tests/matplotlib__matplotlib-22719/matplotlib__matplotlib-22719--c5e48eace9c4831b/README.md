# test_convert_empty_array_preserves_shape

- **Instance:** `matplotlib__matplotlib-22719`
- **Test ID:** `matplotlib__matplotlib-22719--c5e48eace9c4831b`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

Categorical unit conversion should preserve the dimensional shape of an empty ndarray, rather than replacing every empty input with a one-dimensional array.

## Expected behavior

The public conversion call returns an empty ndarray whose shape remains (0, 2).

## Test command

`cd /testbed && python -m pytest -q lib/matplotlib/tests/test_category.py::test_convert_empty_array_preserves_shape`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
