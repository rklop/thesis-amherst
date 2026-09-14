# Differentiating-test run: `django__django-13121`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-13121:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_gold_pass_current_candidate_fail_random100_20260904/runs/django__django-13121--20260904T233403Z--227115`
- Test: `test_avg_duration_field_arithmetic`
- Test command: `cd /testbed && python tests/runtests.py aggregation.tests.AggregateTestCase.test_avg_duration_field_arithmetic`

## Specification gap

Duration-only arithmetic must work not only for stored integer-backed DurationField values, but also for duration expressions such as Avg() whose SQLite result is numeric but represented as a float.

## Input/output contract

Input: Aggregate Publisher.duration values of 1 day and 2 days (ignoring existing NULL durations), then add a 12-hour timedelta to the 1.5-day average through Django's public ORM expression API.

Expected output: Publisher.objects.aggregate() returns {'adjusted': datetime.timedelta(days=2)}.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.95
- Summary: The generated test validates that duration-only arithmetic (Avg(duration) + timedelta) works correctly, which is the core issue: durations-only expressions failing on SQLite/MySQL. Candidate B passes because it fundamentally changes duration arithmetic to handle duration-duration operations numerically instead of routing them through date/time helpers that fail on numeric aggregates. Candidate A fails because its fix only handles timedelta results in _sqlite_format_dtdelta but still routes duration-duration arithmetic through date/time functions that can't handle AVG's float result. The test uses the public ORM API (Avg + timedelta) and verifies a semantically meaningful behavior: two durations added together should produce a duration.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
