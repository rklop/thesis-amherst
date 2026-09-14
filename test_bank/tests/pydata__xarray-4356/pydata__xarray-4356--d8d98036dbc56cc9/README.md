# test_sum_min_count_multiple_dims_with_units

- **Instance:** `pydata__xarray-4356`
- **Test ID:** `pydata__xarray-4356--d8d98036dbc56cc9`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 3

## What it checks

A multi-dimension min_count reduction must apply the validity threshold independently to each remaining output element, including for supported non-NumPy duck arrays.

## Expected behavior

A meter-valued DataArray over z with magnitudes [9.0, 14.0].

## Test command

`cd /testbed && pytest -q xarray/tests/test_units.py::test_sum_min_count_multiple_dims_with_units`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
