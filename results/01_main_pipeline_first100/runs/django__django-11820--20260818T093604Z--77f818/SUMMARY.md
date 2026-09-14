# Differentiating-test run: `django__django-11820`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-11820:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_pipeline_vs_manual_first100_20260818/runs/django__django-11820--20260818T093604Z--77f818`
- Test: `test_ordering_rejects_lookup_after_registered_transform`
- Test command: `cd /testbed && python tests/runtests.py invalid_models_tests.test_models.OtherModelTests.test_ordering_rejects_lookup_after_registered_transform`

## Specification gap

Meta.ordering validation must inspect the entire lookup path. Recognizing a registered transform must not cause validation to ignore later nonexistent components.

## Input/output contract

Input: Define a model with a CharField and Meta.ordering = ('test__lower__missing',), while registering the public Lower transform on CharField.

Expected output: Model.check() returns exactly one models.E015 Error naming 'test__lower__missing'. The 'lower' prefix is valid, but the trailing 'missing' lookup is not.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.95
- Summary: The test correctly identifies that candidate_b properly validates the entire lookup path after a registered transform, while candidate_a stops validation after encountering a valid transform and incorrectly accepts nonexistent suffixes. The test uses Django's public Lower transform API and tests a legitimate validation requirement: after a valid transform, subsequent path components must still be validated. Candidate_b passes and demonstrates specification-conformant behavior.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
