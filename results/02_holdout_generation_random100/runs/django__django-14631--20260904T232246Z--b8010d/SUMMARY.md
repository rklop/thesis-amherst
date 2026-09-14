# Differentiating-test run: `django__django-14631`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-14631:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_gold_pass_current_candidate_fail_random100_20260904/runs/django__django-14631--20260904T232246Z--b8010d`
- Test: `test_custom_boundfield_data_used_for_cleaning_and_changed_data`
- Test command: `cd /testbed && python tests/runtests.py forms_tests.tests.test_forms.FormsTestCase.test_custom_boundfield_data_used_for_cleaning_and_changed_data`

## Specification gap

Form cleaning and change detection must consume data through the BoundField returned by Field.get_bound_field(), not independently extract the widget value. Otherwise the public BoundField can report one value while cleaned_data and changed_data use another.

## Input/output contract

Input: A custom BoundField exposes submitted "mixed" as "MIXED" through its data property. The field's initial value is also "MIXED".

Expected output: The form is valid, its BoundField data and cleaned_data are "MIXED", and changed_data is empty. Candidate B routes both operations through the BoundField; candidate A bypasses it, producing cleaned_data "mixed" and marking the field changed.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.95
- Summary: The test creates a custom BoundField subclass that transforms submitted data (uppercase) and verifies that both cleaned_data and changed_data respect this transformation. Candidate B passes because it routes all value access through BoundField (using bf.data and bf.initial), while candidate A fails because it bypasses BoundField and extracts widget data directly, resulting in cleaned_data='mixed' instead of 'MIXED'. The test exercises the exact issue requirement (accessing values via BoundField) using the public Field.get_bound_field() API entry point, not implementation internals. The oracle is defensible: it checks that the public BoundField.data property and cleaned_data are consistent, which is the core specification requirement.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
