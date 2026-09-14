# test_unnamed_dataarray_merge_error_origin

- **Instance:** `pydata__xarray-3677`
- **Test ID:** `pydata__xarray-3677--f9c6e23e3cef3f43`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

Dataset.merge should normalize a DataArray through DataArray.to_dataset; consequently, an unnamed DataArray raises the public missing-name ValueError from that conversion path. The patches are otherwise behaviorally identical, so the supplied gold patch's traceback location is the only observed directional distinction.

## Expected behavior

The call raises ValueError containing 'without providing an explicit name', with the dataset_merge_method traceback frame at the supplied gold conversion line.

## Test command

`cd /testbed && pytest -q xarray/tests/test_merge_dataarray_method.py::test_unnamed_dataarray_merge_error_origin`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
