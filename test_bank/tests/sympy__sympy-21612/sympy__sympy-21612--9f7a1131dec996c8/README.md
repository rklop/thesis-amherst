# test_Mul_groups_assumption_negative_nested_power

- **Instance:** `sympy__sympy-21612`
- **Test ID:** `sympy__sympy-21612--9f7a1131dec996c8`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

When printing an outer reciprocal, a nested power known to have a negative exponent through assumptions must be treated as a denominator and explicitly grouped; recognition must not depend only on a literal negative coefficient.

## Expected behavior

The string printer returns "x/(y**n)". The parentheses preserve the nested-denominator grouping.

## Test command

`python -c "from sympy.printing.tests.test_str import test_Mul_groups_assumption_negative_nested_power as test; test()"`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
