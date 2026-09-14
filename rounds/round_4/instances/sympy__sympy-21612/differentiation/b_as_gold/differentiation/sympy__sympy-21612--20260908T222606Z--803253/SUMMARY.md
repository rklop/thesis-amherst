# Differentiating-test run: `sympy__sympy-21612`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sympy_1776_sympy-21612:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-04/bidirectional-differentiation/instances/sympy__sympy-21612/b_as_gold/differentiation/sympy__sympy-21612--20260908T222606Z--803253`
- Test: `test_Mul_non_Pow_subclass_denominator`
- Test command: `python bin/test --no-colors sympy/printing/tests/test_str.py -k test_Mul_non_Pow_subclass_denominator`

## Specification gap

A custom expression subclass that explicitly opts out of Pow semantics with is_Pow=False must remain printable as a denominator; reciprocal-specific exponent inspection should not be selected solely from Python inheritance.

## Input/output contract

Input: Construct an unevaluated Pow subclass representing y**nan with is_Pow=False, place its reciprocal in the unevaluated product x/(y**nan), and convert the product to a string.

Expected output: String conversion completes without an exception and returns "x/y**nan". Exponentiation already binds more tightly than division, so this preserves the intended grouping.

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
