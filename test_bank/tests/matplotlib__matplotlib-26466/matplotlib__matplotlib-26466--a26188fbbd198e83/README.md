# test_annotation_xy_duck_array_preserves_units

- **Instance:** `matplotlib__matplotlib-26466`
- **Test ID:** `matplotlib__matplotlib-26466--a26188fbbd198e83`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

Copying `xy` must preserve the semantics of NumPy-dispatchable coordinate objects. In particular, coercing a unit-aware duck array to a plain ndarray can strip units and change the rendered annotation position.

## Expected behavior

The rendered arrow endpoint remains at data coordinate (0.25 km, 0.25 km). Its zero-shrink path endpoint equals `ax.transData.transform((0.25, 0.25))`.

## Test command

`cd /testbed && python -m pytest -q lib/matplotlib/tests/test_text.py::test_annotation_xy_duck_array_preserves_units`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
