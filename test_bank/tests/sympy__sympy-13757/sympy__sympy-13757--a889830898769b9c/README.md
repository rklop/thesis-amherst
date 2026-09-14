# test_Poly_mul_matrix_scalar

- **Instance:** `sympy__sympy-13757`
- **Test ID:** `sympy__sympy-13757--a889830898769b9c`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

Poly should outrank ordinary scalar expressions, but not a composite object's multiplication semantics. Matrix multiplication by a Poly scalar must remain entrywise matrix scaling and preserve Poly results.

## Expected behavior

Matrix([[Poly(x, x), Poly(2*x, x)]]). The operation must return a matrix whose entries are evaluated polynomials.

## Test command

`cd /testbed && python -c "from sympy.polys.tests.test_polytools import test_Poly_mul_matrix_scalar; test_Poly_mul_matrix_scalar()"`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
