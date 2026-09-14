# test_sum_min_count_object_all_dimensions

- **Instance:** `pydata__xarray-4356`
- **Test ID:** `pydata__xarray-4356--af0b51f63c0b1f7d`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

When `dim` is omitted, `sum` reduces all dimensions and must still enforce `min_count` when object-array reduction produces a plain Python scalar without a `dtype` attribute.

## Expected behavior

A scalar missing value (`NaN`), because only one valid element is present while two are required.

## Test command

`python -m pytest -q xarray/tests/test_duck_array_ops.py::test_sum_min_count_object_all_dimensions`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
