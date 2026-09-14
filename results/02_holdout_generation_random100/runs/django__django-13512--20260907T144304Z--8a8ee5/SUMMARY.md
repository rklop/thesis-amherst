# Differentiating-test run: `django__django-13512`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-13512:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_gold_pass_current_candidate_fail_random100_20260904/runs/django__django-13512--20260907T144304Z--8a8ee5`
- Test: `test_json_display_preserves_database_serialization`
- Test command: `python tests/runtests.py admin_utils.tests.UtilsTests.test_json_display_preserves_database_serialization`

## Specification gap

Non-ASCII JSON should be readable in admin presentation without changing JSONField's database-preparation behavior. Candidate A changes serialization globally, while candidate B confines the change to presentation entry points.

## Input/output contract

Input: Pass {'country': '中国'} through admin display_for_field() and through get_prep_value() on the same models.JSONField instance.

Expected output: display_for_field() returns '{"country": "中国"}', while get_prep_value() retains the database serialization '{"country": "\u4e2d\u56fd"}'.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.9
- Summary: The test correctly distinguishes between presentation (admin display) and persistence (database serialization), which aligns with the issue scope: fix admin display without changing database behavior. Candidate B passes because it only modifies admin utils and form fields for presentation, while preserving the original get_prep_value behavior. Candidate A fails because it changes both presentation AND database serialization (adding ensure_ascii=False to model field's get_prep_value), causing the test's expectation about database behavior to fail. The test has a defensible oracle: display should be readable (Unicode), database should be stable (ASCII-escaped).

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
