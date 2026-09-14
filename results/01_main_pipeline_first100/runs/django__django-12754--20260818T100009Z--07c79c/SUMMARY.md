# Differentiating-test run: `django__django-12754`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-12754:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_pipeline_vs_manual_first100_20260818/runs/django__django-12754--20260818T100009Z--07c79c`
- Test: `test_add_model_with_field_removed_from_mixed_case_base_model`
- Test command: `cd /testbed && ./tests/runtests.py migrations.test_autodetector.AutodetectorTests.test_add_model_with_field_removed_from_mixed_case_base_model`

## Specification gap

Moving a field from a concrete base to a newly created subclass must order RemoveField before CreateModel even when the subclass's public string base reference uses the model's original mixed-case name. Django treats string model references case-insensitively, so dependency detection must do the same.

## Input/output contract

Input: The old state contains Readable with a title field. The new state removes Readable.title and creates Book with its own title field, inheriting through the mixed-case string reference 'app.Readable'.

Expected output: The autodetector produces one app migration whose operations are exactly RemoveField followed by CreateModel, preventing the inherited-field clash.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | True | 0 |
| candidate_b | True | False | 1 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 1.0
- Summary: The test validates that the autodetector correctly orders migration operations when moving a field from a concrete base model to a newly created subclass, specifically testing case-insensitivity of model references. Candidate A passes because it normalizes base model names to lowercase before ProjectState lookup, matching Django's internal key handling. Candidate B fails because it uses the mixed-case name directly, causing the dependency check to miss the required ordering constraint.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
