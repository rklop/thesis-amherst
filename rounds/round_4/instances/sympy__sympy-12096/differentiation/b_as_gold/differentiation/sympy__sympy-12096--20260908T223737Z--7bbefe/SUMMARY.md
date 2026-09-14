# Differentiating-test run: `sympy__sympy-12096`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sympy_1776_sympy-12096:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-04/bidirectional-differentiation/instances/sympy__sympy-12096/b_as_gold/differentiation/sympy__sympy-12096--20260908T223737Z--7bbefe`
- Test: `test_implemented_function_evalf_numeric_matrix`
- Test command: `python -c "from sympy.core.tests.test_evalf import test_implemented_function_evalf_numeric_matrix as test; test()"`

## Specification gap

Recursive evalf must accept fully numeric non-scalar arguments, not reject them merely because their scalar `is_number` flag is false. An ImmutableMatrix is a valid numeric argument even though it is not a scalar Expr.

## Input/output contract

Input: An implemented unary function computes the determinant of ImmutableMatrix([[1, 2], [3, 4]]), and evalf() is called on the applied function.

Expected output: The result is numerically convertible to -2.0 rather than remaining an unevaluated function application.

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
