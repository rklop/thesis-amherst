# test_empty_category_conversion_returns_independent_array

- **Instance:** `matplotlib__matplotlib-22719`
- **Test ID:** `matplotlib__matplotlib-22719--1f365ef6269c74ad`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

An empty categorical conversion should produce an independent, shape-preserving float array, not a view backed by an unrelated temporary array.

## Expected behavior

Conversion emits no warning under Matplotlib's warnings-as-errors test configuration and returns shape (0, 2). The result can be resized to (1, 2), populated with [[0, 1]], and read back successfully.

## Test command

`python -m pytest -q lib/matplotlib/tests/test_category.py::test_empty_category_conversion_returns_independent_array`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
