# Differentiating-test run: `django__django-11138`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-11138:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/django__django-11138/a_as_gold/differentiation/django__django-11138--20260908T140833Z--4d2028`
- Test: `test_query_time_lookup`
- Test command: `cd /testbed && python tests/runtests.py timezones.tests.NewDatabaseTests.test_query_time_lookup --verbosity 2`

## Specification gap

Adding database-time-zone support to SQLite’s date lookup must preserve the adjacent public DateTimeField `__time` lookup. The generated candidate changes that lookup’s registered SQL-function arity without changing its caller.

## Input/output contract

Input: With USE_TZ enabled and Africa/Nairobi active, save an Event at 2011-09-01 13:20:30 EAT, then query `dt__time=datetime.time(13, 20, 30)`.

Expected output: The queryset’s `exists()` result is True. Evaluation completes normally rather than raising a SQLite wrong-number-of-arguments error.

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
