# Differentiating-test run: `django__django-10999`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-10999:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_pipeline_vs_manual_first100_20260818/runs/django__django-10999--20260818T085605Z--70417f`
- Test: `test_negative_sign_after_colon`
- Test command: `./tests/runtests.py utils_tests.test_dateparse.DurationParseTests.test_negative_sign_after_colon`

## Specification gap

Only a leading minus sign may negate a standard duration. Candidate A also accepts a minus sign on an individual component after a colon.

## Input/output contract

Input: Call parse_duration('00:01:-01'), where the seconds component has an internal sign.

Expected output: parse_duration() returns None because the input is not a valid standard, ISO 8601, or PostgreSQL duration. Candidate A instead returns timedelta(seconds=59).

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.9
- Summary: The test correctly identifies a semantic difference between candidates. Candidate A's logic to propagate leading negative signs to components also inadvertently accepts individually-signed components like '00:01:-01', whereas Candidate B's refactored regex captures a single leading sign at the start and rejects component-internal signs. The test input '00:01:-01' has an internal sign on seconds that should not be valid under the intended semantics—negative durations should have a single leading minus applied to the entire duration, not per-component signs. Candidate B correctly returns None for this case, aligning with the original test patch expectation that '-01:-01' (both minutes and seconds with signs) returns None.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
