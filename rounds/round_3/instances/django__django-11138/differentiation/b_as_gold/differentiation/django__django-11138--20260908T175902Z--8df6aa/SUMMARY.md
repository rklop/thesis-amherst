# Differentiating-test run: `django__django-11138`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-11138:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/django__django-11138/b_as_gold/differentiation/django__django-11138--20260908T175902Z--8df6aa`
- Test: `test_query_datetime_lookup_with_database_timezone`
- Test command: `cd /testbed && python tests/runtests.py timezones.tests.NewDatabaseTests.test_query_datetime_lookup_with_database_timezone --parallel 1 --verbosity 2`

## Specification gap

Database TIME_ZONE must affect datetime component lookups such as __hour, not only __date and __time casts. SQLite extraction must interpret stored naive values in the database timezone before converting them to the active timezone.

## Input/output contract

Input: With USE_TZ enabled, the database connection timezone is temporarily set to Asia/Bangkok while the active application timezone is Africa/Nairobi. Store 2011-01-01 10:30 UTC, which is persisted as 17:30 Bangkok time, then query Event.dt using dt__hour=13 (the corresponding Nairobi hour).

Expected output: The hour lookup returns exactly one Event. candidate_b converts the stored Bangkok-local value to Nairobi before extracting the hour; candidate_a extracts after treating it as UTC and returns no match.

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
