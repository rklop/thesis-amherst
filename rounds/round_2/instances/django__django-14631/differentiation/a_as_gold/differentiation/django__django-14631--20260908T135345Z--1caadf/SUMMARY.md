# Differentiating-test run: `django__django-14631`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-14631:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/django__django-14631/a_as_gold/differentiation/django__django-14631--20260908T135345Z--1caadf`
- Test: `FormsTestCase.test_custom_boundfield`
- Test command: `./tests/runtests.py forms_tests.tests.test_forms.FormsTestCase.test_custom_boundfield`

## Specification gap

Customizing Field.get_bound_field() must not make validation of an enabled, non-file field depend on the returned presentation object exposing concrete BoundField attributes. The initial-value consistency fix should preserve this existing extension-point behavior.

## Input/output contract

Input: A bound form containing an ordinary CharField whose get_bound_field() hook returns a custom tuple, with submitted data {'name': 'Alice'}.

Expected output: The custom object remains available through form['name'], form.is_valid() returns True without raising an exception, and cleaned_data equals {'name': 'Alice'}.

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
