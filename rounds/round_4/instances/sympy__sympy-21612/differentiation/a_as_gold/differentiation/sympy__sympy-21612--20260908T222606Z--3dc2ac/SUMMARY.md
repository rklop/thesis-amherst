# Differentiating-test run: `sympy__sympy-21612`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sympy_1776_sympy-21612:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-04/bidirectional-differentiation/instances/sympy__sympy-21612/a_as_gold/differentiation/sympy__sympy-21612--20260908T222606Z--3dc2ac`
- Test: `test_sstr_nested_reciprocal_pow_subclass`
- Test command: `cd /testbed && python bin/test sympy/printing/tests/test_str.py -k test_sstr_nested_reciprocal_pow_subclass --no-colors`

## Specification gap

Nested reciprocal grouping must follow concrete Pow semantics for Pow subclasses, even if a subclass customizes the is_Pow classification flag.

## Input/output contract

Input: Pass sstr() an unevaluated x*(p**-1), where p is a Pow subclass instance representing y**-1 with is_Pow=False.

Expected output: sstr() returns `x/(1/y)`, preserving the inner reciprocal as one denominator. The ungrouped `x/1/y` denotes a different division structure.

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
