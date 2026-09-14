# Differentiating-test run: `django__django-13128`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-13128:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_pipeline_vs_manual_first100_20260818/runs/django__django-13128--20260818T101645Z--9cb22f`
- Test: `test_temporal_subtraction_explicit_output_field`
- Test command: `cd /testbed && python tests/runtests.py expressions.tests.FTimeDeltaTests.test_temporal_subtraction_explicit_output_field --verbosity 2`

## Specification gap

Automatic recognition of temporal subtraction must not discard an output_field explicitly supplied by the caller. The candidates disagree because candidate A preserves the original expression and its declared field, while candidate B replaces it during resolution with a TemporalSubtraction whose output is DurationField.

## Input/output contract

Input: On SQLite, select Experiment e2 and annotate end - start with the expression's output_field explicitly set to IntegerField. The fixture interval is 44 seconds.

Expected output: The annotation returns the SQLite integer representation of the interval: 44000000 microseconds.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | True | 0 |
| candidate_b | True | False | 1 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.9
- Summary: The test correctly identifies a genuine specification conflict: whether an explicitly set output_field should be honored or overridden by automatic temporal subtraction inference. Candidate A respects the caller's explicit output_field (returns 44000000 as IntegerField), while candidate B silently replaces with TemporalSubtraction returning timedelta. The issue requests making temporal subtraction work 'without ExpressionWrapper' - which means automatic inference should be convenient, but not override explicit user intent. The test is minimal, tests public API behavior (the annotation result), and has a clear oracle based on the explicit output_field setting.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
