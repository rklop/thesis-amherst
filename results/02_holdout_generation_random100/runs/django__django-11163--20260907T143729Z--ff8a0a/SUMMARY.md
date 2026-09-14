# Differentiating-test run: `django__django-11163`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-11163:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_gold_pass_current_candidate_fail_random100_20260904/runs/django__django-11163--20260907T143729Z--ff8a0a`
- Test: `test_declared_many_to_many_field_saved_with_empty_fields`
- Test command: `python tests/runtests.py model_forms.tests.ModelFormBaseTest.test_declared_many_to_many_field_saved_with_empty_fields`

## Specification gap

A declaratively defined ModelForm field remains part of the form even when Meta.fields is an empty list. The specification does not explicitly state whether save() must still persist such a declared many-to-many field. The established public behavior is that the validated declared field is saved.

## Input/output contract

Input: Create a ModelForm for ColourfulItem with an explicitly declared colours ModelMultipleChoiceField and Meta.fields = []. Bind it to an existing item with one selected Colour, validate it, and call save().

Expected output: The saved item's colours relation contains the submitted Colour. Candidate B preserves this behavior; candidate A's additional empty-fields check silently skips the many-to-many update, leaving the relation empty.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.9
- Summary: The test successfully differentiates between candidates by testing ModelForm.save() behavior with explicitly declared many-to-many fields when Meta.fields=[]. Candidate B (only fixing model_to_dict) passes because it preserves Django's established semantics: explicitly declared fields remain in the form even when Meta.fields is empty. Candidate A fails because its additional fix to BaseModelForm silently drops these declared fields, breaking the documented behavior that explicitly declared fields are always included. The test correctly identifies candidate B as specification-conformant and candidate A as introducing an unintended behavioral change beyond the scope of the original issue.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
