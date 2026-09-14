# Differentiating-test run: `django__django-11138`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-11138:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/django__django-11138/b_as_gold/differentiation/django__django-11138--20260908T140955Z--a88c7f`
- Test: `test_datetime_cast_date_without_connection_timezone`
- Test command: `python tests/runtests.py backends.mysql.test_operations.DatabaseOperationsTests.test_datetime_cast_date_without_connection_timezone`

## Specification gap

A missing database timezone is a sentinel, not a timezone named "None". Date-cast SQL must not attempt timezone conversion unless the connection supplies a concrete source timezone.

## Input/output contract

Input: With USE_TZ enabled, call the MySQL backend's date-cast operation for the field "created_at" and target timezone "Europe/Paris" using a connection whose timezone_name is None.

Expected output: datetime_cast_date_sql() returns "DATE(created_at)". candidate_b skips conversion, while candidate_a produces "DATE(CONVERT_TZ(created_at, 'None', 'Europe/Paris'))".

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
