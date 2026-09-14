# Differentiating-test run: `django__django-11848`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-11848:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-04/bidirectional-differentiation/instances/django__django-11848/b_as_gold/differentiation/django__django-11848--20260908T223803Z--46b296`
- Test: `test_parsing_rfc850_year_just_over_fifty_years_ahead`
- Test command: `cd /testbed && ./tests/runtests.py utils_tests.test_http.HttpDateProcessingTests.test_parsing_rfc850_year_just_over_fifty_years_ahead`

## Specification gap

An RFC 850 year exactly 51 years ahead must roll back one century, while the parser should depend only on the current year's integer value rather than an incidental quotient operation.

## Input/output contract

Input: Freeze UTC to 2018 using a datetime subclass whose year is the int-compatible value 2018, then parse `Sunday, 06-Nov-69 08:49:37 GMT`. The initial interpretation, 2069, is 51 years ahead.

Expected output: `parse_http_date()` returns a timestamp representing `1969-11-06 08:49:37` UTC.

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
