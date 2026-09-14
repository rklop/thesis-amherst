# Differentiating-test run: `sympy__sympy-18199`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sympy_1776_sympy-18199:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/sympy__sympy-18199/b_as_gold/differentiation/sympy__sympy-18199--20260908T141342Z--116a89`
- Test: `test_nthroot_mod_negative_residue_representative`
- Test command: `cd /testbed && /opt/miniconda3/envs/testbed/bin/python -c 'from sympy.ntheory.tests.test_residue import test_nthroot_mod_negative_residue_representative as test; test()'`

## Specification gap

A radicand is an integer representative of a residue class, so the composite-modulus path should normalize negative representatives before testing or solving them.

## Input/output contract

Input: Call nthroot_mod(-1, 3, 8, all_roots=True). Since -1 is congruent to 7 modulo 8, this is equivalent to finding all cube roots of 7 modulo 8.

Expected output: [7], because 7 is the unique residue modulo 8 whose cube is congruent to -1.

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
