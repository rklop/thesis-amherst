# test_sum_min_count_multiple_dims_object_scalar

- **Instance:** `pydata__xarray-4356`
- **Test ID:** `pydata__xarray-4356--8d08500f9c7e322e`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

Multi-dimensional sum must enforce min_count even when an object-typed reduction produces a Python scalar without a dtype attribute.

## Expected behavior

A scalar DataArray containing NaN because only one non-NA value is present. Candidate A instead leaves the partial sum, 1, unchanged.

## Test command

`cd /testbed && pytest -q xarray/tests/test_duck_array_ops.py::test_sum_min_count_multiple_dims_object_scalar`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
