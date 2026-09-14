# Differentiating-test run: `django__django-11790`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-11790:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_pipeline_vs_manual_first100_20260818/runs/django__django-11790--20260818T093137Z--12b004`
- Test: `test_custom_username_field_effective_max_length_in_html`
- Test command: `python tests/runtests.py auth_tests.test_forms.AuthenticationFormTest.test_custom_username_field_effective_max_length_in_html`

## Specification gap

For an AuthenticationForm subclass with a custom username field that normalizes assigned maximum lengths, the rendered HTML maxlength should reflect the form field's effective validation limit, not the unnormalized user-model value.

## Input/output contract

Input: Instantiate a custom AuthenticationForm whose username CharField caps every assigned max_length at 32. The default user model supplies 150, while the custom field retains an effective limit and validator of 32.

Expected output: The rendered username input contains maxlength="32". Candidate A reads the effective form-field value after assignment; Candidate B instead renders the captured model value, maxlength="150".

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | True | 0 |
| candidate_b | True | False | 1 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.95
- Summary: The test correctly identifies a meaningful semantic difference between candidates. Candidate A reads the form field's max_length after assignment (which respects custom field transformations), while Candidate B captures the raw model value before assignment. The test uses a legitimate Django subclassing pattern (custom AuthenticationForm with custom username field) to verify that HTML maxlength reflects the effective form-field limit, not the unnormalized model value. Candidate A passes and correctly implements the expected behavior - the widget attribute should reflect what was actually set on the form field, not the pre-normalized value.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
