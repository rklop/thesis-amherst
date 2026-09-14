# Differentiating-test run: `django__django-11848`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-11848:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/django__django-11848/b_as_gold/differentiation/django__django-11848--20260908T135728Z--7862df`
- Test: `test_parsing_rfc850_year_at_century_boundary`
- Test command: `cd /testbed && python tests/runtests.py utils_tests.test_http.HttpDateProcessingTests.test_parsing_rfc850_year_at_century_boundary`

## Specification gap

Two-digit RFC 850 years must be anchored to the current century, not hard-coded to the 2000s. At the century boundary, a year exactly 50 years ahead remains in the future because only dates more than 50 years ahead roll back.

## Input/output contract

Input: With UTC now pinned to 2100, parse `Friday, 06-Nov-50 08:49:37 GMT`.

Expected output: The returned timestamp represents 2150-11-06 08:49:37 UTC. candidate_a instead produces 2050 because it always adds 2000.

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
