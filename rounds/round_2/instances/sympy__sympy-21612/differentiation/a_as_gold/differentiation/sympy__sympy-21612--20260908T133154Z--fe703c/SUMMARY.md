# Differentiating-test run: `sympy__sympy-21612`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sympy_1776_sympy-21612:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/sympy__sympy-21612/a_as_gold/differentiation/sympy__sympy-21612--20260908T133154Z--fe703c`
- Test: `test_symbolically_negative_power_in_denominator`
- Test command: `cd /testbed && /opt/miniconda3/envs/testbed/bin/python -m unittest -q sympy.printing.tests.test_str_nested_reciprocal.NestedReciprocalStringTest.test_symbolically_negative_power_in_denominator`

## Specification gap

Nested reciprocal printing must handle powers whose negative exponent has a symbolic magnitude; determining negativity from the explicit negative coefficient must not require resolving the symbol's sign.

## Input/output contract

Input: Construct the unevaluated expression z * (x**(-y))**(-1), where x, y, and z are unconstrained symbols, then convert it to a string.

Expected output: str(expr) returns exactly "z/(x**(-y))" without raising an exception.

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
