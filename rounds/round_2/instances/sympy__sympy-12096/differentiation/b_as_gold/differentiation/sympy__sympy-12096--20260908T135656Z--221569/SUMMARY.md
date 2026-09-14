# Differentiating-test run: `sympy__sympy-12096`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sympy_1776_sympy-12096:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/sympy__sympy-12096/b_as_gold/differentiation/sympy__sympy-12096--20260908T135656Z--221569`
- Test: `test_implemented_function_sympy_result_precision`
- Test command: `python bin/test sympy/core/tests/test_evalf.py -k test_implemented_function_sympy_result_precision --no-colors`

## Specification gap

Recursive evalf applies not only to an implemented function's arguments, but also to a SymPy expression returned by its implementation. That returned expression must be evaluated using the requested precision before conversion to Float.

## Input/output contract

Input: Create a one-argument implemented function whose callable returns the exact SymPy constant pi, call it with 1, and request 50-digit evaluation via f(1).evalf(50).

Expected output: The result numerically approximates 3.1415926535897932384626433832795028841971693993751, with absolute error below 1e-49 compared with pi.evalf(50).

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
