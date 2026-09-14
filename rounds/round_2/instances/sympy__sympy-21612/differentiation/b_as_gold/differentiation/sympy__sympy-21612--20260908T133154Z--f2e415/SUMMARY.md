# Differentiating-test run: `sympy__sympy-21612`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sympy_1776_sympy-21612:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/sympy__sympy-21612/b_as_gold/differentiation/sympy__sympy-21612--20260908T133154Z--f2e415`
- Test: `test_Mul_groups_assumption_negative_nested_power`
- Test command: `python -c "from sympy.printing.tests.test_str import test_Mul_groups_assumption_negative_nested_power as test; test()"`

## Specification gap

When printing an outer reciprocal, a nested power known to have a negative exponent through assumptions must be treated as a denominator and explicitly grouped; recognition must not depend only on a literal negative coefficient.

## Input/output contract

Input: Print the unevaluated expression x*(y**n)**(-1), where n is a Symbol declared negative.

Expected output: The string printer returns "x/(y**n)". The parentheses preserve the nested-denominator grouping.

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
