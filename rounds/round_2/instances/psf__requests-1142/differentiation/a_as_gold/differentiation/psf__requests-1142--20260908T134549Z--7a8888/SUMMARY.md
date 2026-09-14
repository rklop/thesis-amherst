# Differentiating-test run: `psf__requests-1142`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.psf_1776_requests-1142:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/psf__requests-1142/a_as_gold/differentiation/psf__requests-1142--20260908T134549Z--7a8888`
- Test: `test_get_without_body_has_no_none_special_case`
- Test command: `cd /testbed && python test_content_length_regression.py`

## Specification gap

A bodyless GET must omit Content-Length; preparation should not retain a purported special case for checking seek capability on None, which has no such capability.

## Input/output contract

Input: Prepare a public requests.Request for GET with no data, then inspect the preparation method’s documented source behavior.

Expected output: The prepared request has no Content-Length header, and the implementation contains no claim that seek capability is checked specially when the body is None.

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
