# test_empty_multidimensional_axis_conversion_skips_numeric_cast

- **Instance:** `matplotlib__matplotlib-22719`
- **Test ID:** `matplotlib__matplotlib-22719--1348111cfbb6658f`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 3

## What it checks

Empty categorical data contains no numeric values to cast. Axis conversion should short-circuit that operation while preserving the empty input's multidimensional shape in a float ndarray.

## Expected behavior

Conversion does not request object-to-float element casting and returns an ndarray with shape (0, 2) and float dtype.

## Test command

`python -m pytest -q lib/matplotlib/tests/test_category.py::TestStrCategoryConverter::test_empty_multidimensional_axis_conversion_skips_numeric_cast`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
