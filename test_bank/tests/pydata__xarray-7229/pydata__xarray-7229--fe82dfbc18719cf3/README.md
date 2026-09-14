# test_where_keep_attrs_dataarray_x_with_dataset_condition

- **Instance:** `pydata__xarray-7229`
- **Test ID:** `pydata__xarray-7229--fe82dfbc18719cf3`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

`keep_attrs=True` must preserve attributes from a DataArray `x` when Dataset return-type priority broadcasts it into an output variable with a different name. Attribute preservation must not depend on `x.name` matching the Dataset variable name.

## Expected behavior

The result is a Dataset whose `mask` variable has values `[10, -1]` and attrs `{"units": "metres"}` inherited from `x`.

## Test command

`cd /testbed && python -m pytest -q xarray/tests/test_computation.py::test_where_keep_attrs_dataarray_x_with_dataset_condition`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
