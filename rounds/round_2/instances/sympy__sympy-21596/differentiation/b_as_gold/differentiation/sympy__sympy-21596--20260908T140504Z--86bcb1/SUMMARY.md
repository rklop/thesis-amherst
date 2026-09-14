# Differentiating-test run: `sympy__sympy-21596`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sympy_1776_sympy-21596:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/sympy__sympy-21596/b_as_gold/differentiation/sympy__sympy-21596--20260908T140504Z--86bcb1`
- Test: `test_imageset_real_intersection_excludes_singular_parameter`
- Test command: `cd /testbed && bin/test sympy/sets/tests/test_imageset_real_singularity.py --no-colors`

## Specification gap

Intersecting a complex-valued integer ImageSet with Reals must both restrict parameters to zero-imaginary-part solutions and exclude solutions where the mapping is non-finite.

## Input/output contract

Input: Construct the image of all integers under f(n) = 1/(n - 1) + I*(n**2 - 1). Its imaginary part vanishes at n = -1 and n = 1, but n = 1 is a pole.

Expected output: The intersection with Reals is exactly FiniteSet(-1/2), the image of the only nonsingular zero-imaginary parameter n = -1.

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
