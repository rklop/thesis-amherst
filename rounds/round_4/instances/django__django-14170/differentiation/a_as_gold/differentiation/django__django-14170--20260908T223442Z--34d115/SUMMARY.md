# Differentiating-test run: `django__django-14170`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-14170:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-04/bidirectional-differentiation/instances/django__django-14170/a_as_gold/differentiation/django__django-14170--20260908T223442Z--34d115`
- Test: `DateFunctionTests.test_extract_iso_year_lookup_includes_last_sunday`
- Test command: `cd /testbed && ./tests/runtests.py db_functions.datetime.test_extract_trunc.DateFunctionTests.test_extract_iso_year_lookup_includes_last_sunday`

## Specification gap

A DateTimeField ISO-year lookup must include every time on the ISO year's final Sunday, not only midnight at the start of that day.

## Input/output contract

Input: Create a DTModel with start_datetime=2021-01-03 12:00:00. This Sunday is the final day of ISO year 2020. Filter with start_datetime__iso_year=2020.

Expected output: The queryset contains the created model. candidate_a bounds ISO year 2020 through Sunday evening; candidate_b ends its datetime bound at Sunday 00:00:00 and therefore omits the noon record.

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
