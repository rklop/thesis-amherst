# Differentiating-test run: `django__django-11848`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-11848:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/django__django-11848/a_as_gold/differentiation/django__django-11848--20260908T135715Z--f8e9d6`
- Test: `test_parsing_rfc850_fifty_year_century_boundary`
- Test command: `cd /testbed && ./tests/runtests.py utils_tests.test_http.HttpDateProcessingTests.test_parsing_rfc850_fifty_year_century_boundary`

## Specification gap

Rollback applies only when an RFC 850 year is more than 50 years in the future. The generated candidate incorrectly maps `20` to 1920 when the current year is 1970, although 2020 is exactly 50 years ahead and must remain unchanged.

## Input/output contract

Input: Patch the parser's UTC clock to 1970-01-01 and parse `Friday, 06-Nov-20 08:49:37 GMT`.

Expected output: `parse_http_date()` returns an epoch value representing 2020-11-06 08:49:37 UTC, not 1920.

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
