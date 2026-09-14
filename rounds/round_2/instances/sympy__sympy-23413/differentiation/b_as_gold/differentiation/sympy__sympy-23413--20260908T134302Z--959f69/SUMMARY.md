# Differentiating-test run: `sympy__sympy-23413`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sympy_1776_sympy-23413:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/sympy__sympy-23413/b_as_gold/differentiation/sympy__sympy-23413--20260908T134302Z--959f69`
- Test: `test_hermite_normal_tall_single_column`
- Test command: `python bin/test sympy/matrices/tests/test_normalforms.py --no-colors -k test_hermite_normal_tall_single_column`

## Specification gap

For a tall one-column matrix, the lowest nonzero entry is the sole pivot and must be positive. Rows above it are not additional pivot rows and must not undo that sign normalization.

## Input/output contract

Input: Call the public hermite_normal_form API with Matrix([[2], [-1]]), a 2x1 integer matrix whose bottom pivot is negative.

Expected output: Matrix([[-2], [1]]). The only unimodular transformation available for one column is a sign change, making the bottom pivot positive. candidate_a reprocesses the upper row and flips the column back, leaving a negative pivot.

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
