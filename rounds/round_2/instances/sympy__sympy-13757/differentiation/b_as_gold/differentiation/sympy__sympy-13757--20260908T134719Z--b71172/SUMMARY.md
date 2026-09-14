# Differentiating-test run: `sympy__sympy-13757`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sympy_1776_sympy-13757:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/sympy__sympy-13757/b_as_gold/differentiation/sympy__sympy-13757--20260908T134719Z--b71172`
- Test: `test_Poly_mul_matrix_scalar`
- Test command: `cd /testbed && python -c "from sympy.polys.tests.test_polytools import test_Poly_mul_matrix_scalar; test_Poly_mul_matrix_scalar()"`

## Specification gap

Poly should outrank ordinary scalar expressions, but not a composite object's multiplication semantics. Matrix multiplication by a Poly scalar must remain entrywise matrix scaling and preserve Poly results.

## Input/output contract

Input: Multiply the public mutable Matrix([[1, 2]]) on the left by Poly(x, x).

Expected output: Matrix([[Poly(x, x), Poly(2*x, x)]]). The operation must return a matrix whose entries are evaluated polynomials.

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
