# test_Mul_nested_reciprocal_symbolic_negative_assumption

- **Instance:** `sympy__sympy-21612`
- **Test ID:** `sympy__sympy-21612--63f7938bfdccc07f`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 3

## What it checks

Reciprocal grouping should depend on whether the nested power’s exponent is known negative, not on that truth value being the exact Python `True` singleton.

## Expected behavior

String printing returns `x/(y**n)`, preserving parentheses around the complete denominator. Printing `x/y**n` would expose different grouping.

## Test command

`cd /testbed && python -c "from sympy.printing.tests.test_str import test_Mul_nested_reciprocal_symbolic_negative_assumption as test; test()"`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
