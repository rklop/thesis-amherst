# Differentiating-test run: `django__django-11400`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-11400:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_pipeline_vs_manual_first100_20260818/runs/django__django-11400--20260818T091650Z--fbfa02`
- Test: `test_relatedonlyfieldlistfilter_inherits_custom_ordering`
- Test command: `python tests/runtests.py admin_filters.tests.ListFiltersTests.test_relatedonlyfieldlistfilter_inherits_custom_ordering`

## Specification gap

RelatedOnlyFieldListFilter should use the same overridable related-field ordering hook as its RelatedFieldListFilter parent. Otherwise custom filter subclasses cannot change ordering without duplicating the parent’s restricted-choice logic.

## Input/output contract

Input: A custom RelatedOnlyFieldListFilter overrides field_admin_ordering() to sort employees by name. Two books reference employees created in the opposite order: John Blue first and Jack Red second.

Expected output: The externally visible filter choices contain only the referenced employees and appear as Jack Red followed by John Blue.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.9
- Summary: The test validates that custom RelatedOnlyFieldListFilter subclasses can override ordering behavior through an inherited hook method, which is a legitimate API extensibility concern. Candidate B passes by providing an overridable field_admin_ordering() method used by both filter variants, while Candidate A fails because it bypasses this hook with direct registry lookup.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
