# Differentiating-test run: `sympy__sympy-21612`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sympy_1776_sympy-21612:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/sympy__sympy-21612/b_as_gold/differentiation/sympy__sympy-21612--20260908T172154Z--6a2739`
- Test: `test_nested_reciprocal_with_conditional_exponent_sign`
- Test command: `cd /testbed && python bin/test sympy/printing/tests/test_str_nested_reciprocal.py --no-colors`

## Specification gap

String printing a nested reciprocal should apply special parentheses only when the inner exponent is definitely negative; an unresolved symbolic sign condition must not be coerced to a Python boolean.

## Input/output contract

Input: Construct the unevaluated quotient x/(y**e), where e is a custom symbolic exponent whose `is_negative` result is the unresolved relation `x < 0`, and pass it to the public `str()` printer.

Expected output: Printing completes without an exception and returns exactly `x/y**ConditionalSignExponent()`.

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
