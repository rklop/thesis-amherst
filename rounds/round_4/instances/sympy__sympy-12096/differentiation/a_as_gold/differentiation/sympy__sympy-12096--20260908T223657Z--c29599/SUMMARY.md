# Differentiating-test run: `sympy__sympy-12096`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sympy_1776_sympy-12096:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-04/bidirectional-differentiation/instances/sympy__sympy-12096/a_as_gold/differentiation/sympy__sympy-12096--20260908T223657Z--c29599`
- Test: `test_implemented_function_evalf_non_numeric_argument`
- Test command: `python bin/test sympy/core/tests/test_evalf.py -k implemented_function_evalf_non_numeric_argument`

## Specification gap

After recursively evaluating its arguments, evalf must leave an implemented function unevaluated if any argument remains nonnumeric, including non-Expr SymPy objects. candidate_b only applies this guard to Expr instances.

## Input/output contract

Input: Apply an implemented function whose implementation returns 1 to FiniteSet(1, 2), then call evalf() on that application.

Expected output: The result remains the original symbolic function application f(FiniteSet(1, 2)); the numerical implementation is not applied to the nonnumeric set argument.

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
