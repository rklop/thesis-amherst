# Differentiating-test run: `django__django-12143`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-12143:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_pipeline_vs_manual_first100_20260818/runs/django__django-12143--20260818T094638Z--9082bb`
- Test: `test_get_edited_object_pks_with_regex_chars_in_pk_name`
- Test command: `./tests/runtests.py admin_changelist.tests.ChangeListTests.test_get_edited_object_pks_with_regex_chars_in_pk_name`

## Specification gap

Regex metacharacters must be treated literally not only in a formset prefix, but in every model-derived component used to recognize list-editable POST keys. Django permits dynamically declared field names such as "serial$", but candidate B interpolates the primary-key name into the regex unescaped.

## Input/output contract

Input: A dynamically constructed model has an IntegerField primary key named "serial$". The POST contains the genuine key "form-0-serial$" with value "7" and a regex-lookalike key "form-1-serial" with value "8".

Expected output: _get_edited_object_pks() must return ["7"], selecting the literal primary-key field and rejecting the lookalike without the dollar sign. Candidate B instead treats "$" as an end-of-string anchor and returns ["8"].

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | True | 0 |
| candidate_b | True | False | 1 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.95
- Summary: The test correctly identifies that candidate_b's fix is incomplete - it only escapes the prefix but not the pk.name, leading to incorrect matching when pk.name contains regex metacharacters like '$'. The test reveals a meaningful missing specification: while the original issue focused on prefix escaping, the same logic applies to pk.name since Django allows dynamically declared field names. The test passes candidate_a (full escape) and fails candidate_b (partial escape), correctly identifying the more specification-conformant solution.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
