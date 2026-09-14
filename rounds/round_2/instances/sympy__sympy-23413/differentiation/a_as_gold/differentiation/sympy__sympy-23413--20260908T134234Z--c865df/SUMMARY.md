# Differentiating-test run: `sympy__sympy-23413`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sympy_1776_sympy-23413:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/sympy__sympy-23413/a_as_gold/differentiation/sympy__sympy-23413--20260908T134234Z--c865df`
- Test: `test_hermite_normal_tall_single_column_sign`
- Test command: `cd /testbed && python bin/test sympy/polys/matrices/tests/test_normalforms.py -k test_hermite_normal_tall_single_column_sign`

## Specification gap

For a tall integer matrix, rows above the bottom n rows still participate in canonical HNF sign normalization. In a one-column matrix, the column must be globally negated when its first nonzero entry is negative.

## Input/output contract

Input: Call the public DomainMatrix hermite_normal_form API with the 2-by-1 integer matrix DM([[-2], [3]], ZZ).

Expected output: DM([[2], [-3]], ZZ): the equivalent column is sign-normalized so its first nonzero entry is positive. The generated candidate instead exits after processing the bottom row and leaves DM([[-2], [3]], ZZ).

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
