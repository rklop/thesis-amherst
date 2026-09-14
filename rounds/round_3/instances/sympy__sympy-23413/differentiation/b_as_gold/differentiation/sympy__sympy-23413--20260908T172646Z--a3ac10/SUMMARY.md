# Differentiating-test run: `sympy__sympy-23413`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sympy_1776_sympy-23413:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/sympy__sympy-23413/b_as_gold/differentiation/sympy__sympy-23413--20260908T172646Z--a3ac10`
- Test: `test_hermite_normal_exhausted_columns`
- Test command: `cd /testbed && python -c "from sympy.matrices.tests.test_normalforms import test_hermite_normal_exhausted_columns as t; t()"`

## Specification gap

For a tall, full-column-rank integer matrix, HNF must preserve every input row and stop pivot traversal once every available column has a pivot.

## Input/output contract

Input: Call the public hermite_normal_form API on Matrix([[0, 12], [0, 8], [1, 5]]) while tracing the exhausted pivot-column boundary.

Expected output: The call returns Matrix([[12, 0], [8, 0], [0, 1]]) without the pivot-column cursor becoming negative.

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
