# Differentiating-test run: `django__django-11138`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-11138:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-04/bidirectional-differentiation/instances/django__django-11138/b_as_gold/differentiation/django__django-11138--20260908T224623Z--c6b476`
- Test: `matching_database_timezone_at_dst_transition`
- Test command: `cd /testbed && ./tests/runtests.py timezones.tests.NewDatabaseTests.test_date_lookup_with_matching_database_timezone_at_dst_transition --settings=test_sqlite`

## Specification gap

When the database connection timezone and active query timezone are identical, date extraction is an identity operation, including for ambiguous DST wall times.

## Input/output contract

Input: Store the explicitly disambiguated Europe/Paris datetime 2019-10-27 02:30 during the DST transition, configure both the database connection and active query timezone as Europe/Paris, then query it through DateTimeField's public __date lookup.

Expected output: The lookup completes without relocalizing the ambiguous wall time and returns True for 2019-10-27.

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
