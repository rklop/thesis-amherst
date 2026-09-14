# Differentiating-test run: `django__django-11848`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-11848:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/django__django-11848/a_as_gold/differentiation/django__django-11848--20260908T174137Z--ba926e`
- Test: `test_parsing_rfc850_year_in_past`
- Test command: `cd /testbed && python tests/runtests.py utils_tests.test_http.HttpDateProcessingTests.test_parsing_rfc850_year_in_past`

## Specification gap

A two-digit RFC 850 year already in the current century's past must remain there. Only a year appearing more than 50 years in the future is rolled back; old past years aren't symmetrically rolled forward.

## Input/output contract

Input: With the current UTC year fixed at 2075, parse the RFC 850 date "Friday, 06-Nov-20 08:49:37 GMT".

Expected output: parse_http_date() returns the timestamp for 2020-11-06 08:49:37 UTC, not 2120-11-06 08:49:37 UTC.

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
