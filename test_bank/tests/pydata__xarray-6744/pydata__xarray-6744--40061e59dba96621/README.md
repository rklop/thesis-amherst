# test_rolling_iter_uses_hashable_dimension_size

- **Instance:** `pydata__xarray-6744`
- **Test ID:** `pydata__xarray-6744--40061e59dba96621`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

A rolling iterator must emit one window per element of the selected rolling dimension. DataArrays obtained through the public Dataset entry point can carry a hashable integer dimension, where `da[0]` performs positional indexing and therefore cannot be used to determine dimension 0's length.

## Expected behavior

Iteration produces exactly four windows, whose lengths along dimension 0 are `[2, 3, 3, 2]`.

## Test command

`cd /testbed && python -m pytest -q xarray/tests/test_rolling.py::TestDataArrayRolling::test_rolling_iter_uses_hashable_dimension_size`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
