# test_min_count_python_scalar_object_result

- **Instance:** `pydata__xarray-4356`
- **Test ID:** `pydata__xarray-4356--20d3c02c85b8ff14`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

Object dtype is outside the documented skipna support, so enabling multi-dimensional min_count must not additionally coerce a Python scalar object result to floating-point NaN. The supplied gold preserves such scalar-like results; the generated candidate does not.

## Expected behavior

The zero-dimensional result retains Decimal("1"); result.item() equals Decimal("1") rather than NaN.

## Test command

`python -m pytest -q xarray/tests/test_duck_array_ops.py::test_min_count_python_scalar_object_result`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
