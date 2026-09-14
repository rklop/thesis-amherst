# test_where_keep_attrs_preserves_x_coordinate_attrs

- **Instance:** `pydata__xarray-7229`
- **Test ID:** `pydata__xarray-7229--e386f9fa0418ccbb`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

With keep_attrs=True, coordinate attributes must come from x even when x has no top-level attributes and cond provides the same coordinate without metadata.

## Expected behavior

The result's time coordinate has attrs exactly {'standard_name': 'time'}.

## Test command

`cd /testbed && pytest -q xarray/tests/test_computation.py::test_where_keep_attrs_preserves_x_coordinate_attrs`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
