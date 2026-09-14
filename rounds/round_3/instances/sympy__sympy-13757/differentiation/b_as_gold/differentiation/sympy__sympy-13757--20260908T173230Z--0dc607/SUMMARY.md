# Differentiating-test run: `sympy__sympy-13757`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sympy_1776_sympy-13757:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/sympy__sympy-13757/b_as_gold/differentiation/sympy__sympy-13757--20260908T173230Z--0dc607`
- Test: `test_Poly_matrix_scalar_mul_generator_order`
- Test command: `python -c "from sympy.polys.tests.test_polytools import test_Poly_matrix_scalar_mul_generator_order; test_Poly_matrix_scalar_mul_generator_order()"`

## Specification gap

Poly must outrank ordinary Expr operands to make left-hand multiplication evaluate, but it must not take dispatch away from higher-priority containers such as Matrix. Matrix scalar multiplication must preserve its established polynomial-unification order.

## Input/output contract

Input: Multiply the mutable dense matrix Matrix([[Poly(y, y)]]) by the scalar Poly(x, x).

Expected output: Matrix([[Poly(x*y, x, y, domain='ZZ')]]). The entry is a polynomial in public generator order (x, y), not (y, x).

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
