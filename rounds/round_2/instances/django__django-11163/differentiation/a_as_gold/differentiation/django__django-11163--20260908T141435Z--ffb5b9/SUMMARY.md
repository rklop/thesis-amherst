# Differentiating-test run: `django__django-11163`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-11163:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/django__django-11163/a_as_gold/differentiation/django__django-11163--20260908T141435Z--ffb5b9`
- Test: `test_explicit_m2m_field_with_empty_meta_fields_is_saved`
- Test command: `cd /testbed && python tests/runtests.py model_forms.tests.ModelFormBaseTest.test_explicit_m2m_field_with_empty_meta_fields_is_saved`

## Specification gap

An empty fields list means model_to_dict() should return no fields, but it must not be generalized to suppress explicitly declared ModelForm fields. A declaratively defined many-to-many field remains part of the form and its cleaned value must be saved even when Meta.fields is empty.

## Input/output contract

Input: Submit a valid ModelForm for an existing Article. The form explicitly declares its categories ModelMultipleChoiceField while Meta.fields = [], and the submitted value contains one Category primary key.

Expected output: The form is valid and form.save() persists the submitted Category in the Article's many-to-many categories relation. candidate_a retains this behavior; candidate_b skips the relation because of its additional _save_m2m() change.

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
