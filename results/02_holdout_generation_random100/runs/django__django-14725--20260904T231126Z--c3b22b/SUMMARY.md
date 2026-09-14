# Differentiating-test run: `django__django-14725`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-14725:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_gold_pass_current_candidate_fail_random100_20260904/runs/django__django-14725--20260904T231126Z--c3b22b`
- Test: `test_edit_only_skips_custom_save_new_objects`
- Test command: `./tests/runtests.py model_formsets.tests.ModelFormsetTest.test_edit_only_skips_custom_save_new_objects`

## Specification gap

The edit_only guarantee must remain authoritative for custom model formset classes supplied through the public formset= extension point. Candidate_a guards only BaseModelFormSet.save_new_objects(), so an existing override can still create objects. Candidate_b skips the new-object saving phase entirely.

## Input/output contract

Input: Submit one valid new Author form to a model formset configured with edit_only=True and a custom BaseModelFormSet subclass whose save_new_objects() implements conventional new-form saving.

Expected output: The formset is valid, save() returns an empty list, and no Author is inserted into the database.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.95
- Summary: The test verifies a critical specification requirement: the edit_only feature must prevent new object creation even when users provide custom BaseModelFormSet subclasses via the public formset= extension point. Candidate_a guards only BaseModelFormSet.save_new_objects(), allowing custom overrides to still create objects. Candidate_b correctly handles this by skipping the entire new-object saving phase in save(). The test is minimal, targets externally observable behavior (save() return value and database state), and reveals a real gap in candidate_a's approach that would affect users who customize formset behavior through Django's documented API.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
