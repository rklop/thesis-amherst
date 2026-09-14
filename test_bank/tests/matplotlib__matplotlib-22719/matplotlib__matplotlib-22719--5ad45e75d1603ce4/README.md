# test_convert_empty_multidimensional_array

- **Instance:** `matplotlib__matplotlib-22719`
- **Test ID:** `matplotlib__matplotlib-22719--5ad45e75d1603ce4`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

For categorical unit conversion, “empty data” means an array with zero total elements, including arrays whose outer dimension is nonzero. Such input should be handled before attempting to map its rows as categories.

## Expected behavior

Conversion completes without an exception and returns a floating-point array containing zero elements.

## Test command

`cd /testbed && python -m pytest -q lib/matplotlib/tests/test_category.py::test_convert_empty_multidimensional_array`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
