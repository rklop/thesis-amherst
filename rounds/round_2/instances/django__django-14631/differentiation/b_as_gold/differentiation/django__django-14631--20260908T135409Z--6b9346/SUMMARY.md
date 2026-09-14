# Differentiating-test run: `django__django-14631`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-14631:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/django__django-14631/b_as_gold/differentiation/django__django-14631--20260908T135409Z--6b9346`
- Test: `Custom BoundField data is used throughout form processing`
- Test command: `python tests/runtests.py forms_tests.tests.test_forms.FormsTestCase.test_custom_boundfield_data_used_by_form`

## Specification gap

BaseForm must treat BoundField.data as the canonical submitted value for both field cleaning and change detection, including when Field.get_bound_field() supplies a custom BoundField.

## Input/output contract

Input: A custom BoundField uppercases its submitted data. The form receives lowercase "alice" while the field's initial value is uppercase "ALICE".

Expected output: The form is valid, cleaned_data is {"name": "ALICE"}, and changed_data is empty because both operations observe the normalized BoundField value.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
