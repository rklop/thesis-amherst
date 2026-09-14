# Differentiating-test run: `django__django-11880`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-11880:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_pipeline_vs_manual_first100_20260818/runs/django__django-11880--20260818T093931Z--45ad9f`
- Test: `test_error_message_values_independence`
- Test command: `PYTHONPATH=. ./tests/runtests.py forms_tests.tests.test_forms.FormsTestCase.test_error_message_values_independence --verbosity 0`

## Specification gap

The existing test requires only a distinct error_messages mapping, so a shallow dictionary copy passes. It doesn't specify whether mutable message values accepted by Django's ValidationError path are also isolated between form instances.

## Input/output contract

Input: Define a Form whose required-field error message is a two-item list, instantiate two bound forms, and append a third message through only the first form's field.

Expected output: The first form reports ['First error.', 'Second error.', 'Third error.'], while the second still reports ['First error.', 'Second error.'].

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | True | 0 |
| candidate_b | True | False | 1 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.95
- Summary: This is an excellent differentiating test that exercises the core semantic issue: error_messages values that are mutable (like lists) must be deep-copied to ensure form instance independence. Candidate A passes because it uses copy.deepcopy for recursive copying, while candidate B fails because .copy() only shallow-copies the dictionary, causing the list value to be shared between form instances. The test legitimately uses Django's supported feature of list-valued error messages and verifies the expected isolation behavior described in the issue.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
