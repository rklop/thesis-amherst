# Differentiating-test run: `sympy__sympy-12096`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sympy_1776_sympy-12096:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/sympy__sympy-12096/a_as_gold/differentiation/sympy__sympy-12096--20260908T173841Z--3d39ff`
- Test: `test_implemented_function_evalf_nested_argument_precision`
- Test command: `cd /testbed && bin/test sympy/core/tests/test_evalf.py -k test_implemented_function_evalf_nested_argument_precision`

## Specification gap

Recursive evaluation of an implemented function's arguments must preserve enough working precision for the enclosing numerical implementation, not merely enough digits to display the final result. Otherwise a nested identity function can change a precision-sensitive computation.

## Input/output contract

Input: Evaluate oscillatory(identity(pi)) to 15 digits, where identity(x) returns x and oscillatory(x) computes sin(10**20*x). Mathematically this is sin(10**20*pi), which is zero.

Expected output: The returned numeric value has absolute value below 1e-12. The tolerance allows harmless numerical residue while requiring the nested identity implementation to preserve pi accurately enough for the outer high-frequency sine calculation.

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
