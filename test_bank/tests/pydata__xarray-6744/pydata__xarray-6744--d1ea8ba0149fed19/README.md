# test_rolling_iter_uses_only_window_labels

- **Instance:** `pydata__xarray-6744`
- **Test ID:** `pydata__xarray-6744--d1ea8ba0149fed19`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 3

## What it checks

Fixing centered manual iteration must preserve the iterator's label domain: every yielded item must correspond to an existing `window_labels` entry, rather than synthesizing positional labels when the label sequence and rolled-axis length differ.

## Expected behavior

The iterator yields exactly the declared labels `[0, 1]`. It must not append synthesized labels `[2, 3, 4]`.

## Test command

`pytest -q xarray/tests/test_rolling.py::TestDataArrayRolling::test_rolling_iter_uses_only_window_labels`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
