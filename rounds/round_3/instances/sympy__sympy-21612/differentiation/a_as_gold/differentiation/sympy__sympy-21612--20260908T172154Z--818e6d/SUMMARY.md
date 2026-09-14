# Differentiating-test run: `sympy__sympy-21612`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sympy_1776_sympy-21612:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/sympy__sympy-21612/a_as_gold/differentiation/sympy__sympy-21612--20260908T172154Z--818e6d`
- Test: `test_Mul_nested_reciprocal_symbolic_negative_assumption`
- Test command: `cd /testbed && python -c "from sympy.printing.tests.test_str import test_Mul_nested_reciprocal_symbolic_negative_assumption as test; test()"`

## Specification gap

Reciprocal grouping should depend on whether the nested power’s exponent is known negative, not on that truth value being the exact Python `True` singleton.

## Input/output contract

Input: An unevaluated product `x * 1/(y**n)`, where `n` is a Symbol subtype reporting negativity with SymPy’s public symbolic truth singleton.

Expected output: String printing returns `x/(y**n)`, preserving parentheses around the complete denominator. Printing `x/y**n` would expose different grouping.

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
