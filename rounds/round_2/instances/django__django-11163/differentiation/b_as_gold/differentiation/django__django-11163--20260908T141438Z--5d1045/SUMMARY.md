# Differentiating-test run: `django__django-11163`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-11163:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/django__django-11163/b_as_gold/differentiation/django__django-11163--20260908T141438Z--5d1045`
- Test: `test_empty_fields_do_not_save_declared_many_to_many_field`
- Test command: `cd /testbed && ./tests/runtests.py model_forms.tests.ModelFormBaseTest.test_empty_fields_do_not_save_declared_many_to_many_field`

## Specification gap

An empty ModelForm Meta.fields list means no model fields are selected for persistence, including many-to-many fields that are explicitly declared on the form and therefore appear in cleaned_data.

## Input/output contract

Input: Create an item related to an existing colour, then bind it to a ModelForm with Meta.fields = [] and an explicitly declared colours field containing a different colour. Call the public form.save() method.

Expected output: The item's persisted colour relation remains the original colour; the submitted replacement is not saved because colours isn't selected by Meta.fields.

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
