# Differentiating-test run: `sympy__sympy-13757`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sympy_1776_sympy-13757:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/sympy__sympy-13757/a_as_gold/differentiation/sympy__sympy-13757--20260908T134633Z--91c209`
- Test: `test_Poly_mul_immutable_matrix`
- Test command: `cd /testbed && python bin/test -C --no-colors -k test_Poly_mul_immutable_matrix sympy/polys/tests/test_polytools.py`

## Specification gap

Left-hand multiplication must also dispatch to Poly when the expression is an ImmutableMatrix, an Expr-derived public type with its own arithmetic precedence.

## Input/output contract

Input: Multiply the one-element expression ImmutableMatrix([[2]]) on the left by Poly(x, x).

Expected output: The result is Poly(2*x, x, domain='ZZ'), rather than an ImmutableMatrix containing that polynomial. Candidate_b only ties ImmutableMatrix's precedence and therefore leaves the matrix as the outer result.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | True | 0 |
| candidate_b | True | False | 1 |

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
