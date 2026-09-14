# test_symbolically_negative_power_in_denominator

- **Instance:** `sympy__sympy-21612`
- **Test ID:** `sympy__sympy-21612--944d3353b8b52b3a`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

Nested reciprocal printing must handle powers whose negative exponent has a symbolic magnitude; determining negativity from the explicit negative coefficient must not require resolving the symbol's sign.

## Expected behavior

str(expr) returns exactly "z/(x**(-y))" without raising an exception.

## Test command

`cd /testbed && /opt/miniconda3/envs/testbed/bin/python -m unittest -q sympy.printing.tests.test_str_nested_reciprocal.NestedReciprocalStringTest.test_symbolically_negative_power_in_denominator`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
