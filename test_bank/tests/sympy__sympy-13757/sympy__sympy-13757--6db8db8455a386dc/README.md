# test_Poly_matrix_scalar_mul_generator_order

- **Instance:** `sympy__sympy-13757`
- **Test ID:** `sympy__sympy-13757--6db8db8455a386dc`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 3

## What it checks

Poly must outrank ordinary Expr operands to make left-hand multiplication evaluate, but it must not take dispatch away from higher-priority containers such as Matrix. Matrix scalar multiplication must preserve its established polynomial-unification order.

## Expected behavior

Matrix([[Poly(x*y, x, y, domain='ZZ')]]). The entry is a polynomial in public generator order (x, y), not (y, x).

## Test command

`python -c "from sympy.polys.tests.test_polytools import test_Poly_matrix_scalar_mul_generator_order; test_Poly_matrix_scalar_mul_generator_order()"`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
