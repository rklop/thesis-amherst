# Differentiating-test run: `django__django-14170`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-14170:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/django__django-14170/b_as_gold/differentiation/django__django-14170--20260908T135435Z--2d9c28`
- Test: `test_iso_year_lookup_bounds_for_date_field`
- Test command: `python tests/runtests.py backends.base.test_operations.SimpleDatabaseOperationTests.test_iso_year_lookup_bounds_for_date_field`

## Specification gap

ISO-year lookup bounds must cross calendar-year boundaries when necessary and, like ordinary year lookup bounds, must be returned in the backend-adapted representation rather than as raw Python date objects.

## Input/output contract

Input: Call BaseDatabaseOperations.iso_year_lookup_bounds_for_date_field(2015). ISO year 2015 begins on Monday, 2014-12-29, and ends on Sunday, 2016-01-03.

Expected output: The method returns the two backend-adapted date values ['2014-12-29', '2016-01-03']. candidate_b applies adapt_datefield_value() to both bounds; candidate_a returns datetime.date objects instead.

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
