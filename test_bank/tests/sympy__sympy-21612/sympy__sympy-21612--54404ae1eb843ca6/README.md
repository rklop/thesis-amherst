# test_nested_reciprocal_denominator_grouping

- **Instance:** `sympy__sympy-21612`
- **Test ID:** `sympy__sympy-21612--54404ae1eb843ca6`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

A nested reciprocal must retain its own parentheses even when it is one of multiple factors in an outer denominator. The generated candidate only handles a single denominator factor.

## Expected behavior

sstr returns "x/(z*(1/y))", preserving the nested reciprocal rather than flattening it to "x/(z*1/y)".

## Test command

`cd /testbed && python bin/test --no-colors sympy/printing/tests/test_str.py -k test_nested_reciprocal_denominator_grouping`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
