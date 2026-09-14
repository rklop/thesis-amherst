# test_poly_equality_with_matrix_operand

- **Instance:** `sympy__sympy-13757`
- **Test ID:** `sympy__sympy-13757--0dba49d6b50c233d`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 3

## What it checks

Polynomial equality must safely classify public SymPy operands that do not expose `is_Poly`; an incompatible matrix compares unequal rather than raising `AttributeError`.

## Expected behavior

The comparison returns the Python boolean `False` without raising an exception.

## Test command

`cd /testbed && python bin/test sympy/polys/tests/test_polytools_equality_types.py --no-colors`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
