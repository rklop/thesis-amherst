# Differentiating-test run: `django__django-11848`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-11848:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/django__django-11848/b_as_gold/differentiation/django__django-11848--20260908T174338Z--1175aa`
- Test: `test_parsing_rfc850_year_exactly_50_years_in_future`
- Test command: `cd /testbed && ./tests/runtests.py utils_tests.test_http.HttpDateProcessingTests.test_parsing_rfc850_year_exactly_50_years_in_future`

## Specification gap

A two-digit RFC 850 year exactly 50 years in the future must remain future-facing even when it crosses a century boundary; only dates more than 50 years ahead roll into the past.

## Input/output contract

Input: Pin the current UTC date to 1969-01-01 and parse `Tuesday, 01-Jan-19 00:00:00 GMT`, representing the exact 50-year boundary.

Expected output: `parse_http_date()` returns a timestamp corresponding to 2019-01-01 00:00:00 UTC. Candidate_a instead resolves the year to 1919 by anchoring it to the current century.

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
