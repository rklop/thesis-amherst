# Differentiating-test run: `django__django-11477`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-11477:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_pipeline_vs_manual_first100_20260818/runs/django__django-11477--20260818T092109Z--d4bad2`
- Test: `test_converter_ignores_unmatched_named_subgroups`
- Test command: `cd /testbed && python tests/runtests.py urlpatterns.tests.SimplifiedURLTests.test_converter_ignores_unmatched_named_subgroups`

## Specification gap

Optional named regex subgroups should be omitted consistently when resolving path() routes, including subgroups inside a registered custom converter. Only parameters declared by the route should be converted and exposed as view kwargs.

## Input/output contract

Input: Register a custom path converter whose regex matches either "primary-<qualifier>" or "fallback". Resolve "/item/fallback/", leaving the converter's internal named "qualifier" subgroup unmatched.

Expected output: resolve() succeeds with URL name "optional-subgroup" and kwargs {'value': 'FALLBACK'}. The unmatched internal subgroup is neither exposed nor treated as a route converter.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | True | 0 |
| candidate_b | True | False | 1 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.9
- Summary: The test validates that unmatched optional named groups in URL patterns (both re_path and path() with custom converters) are filtered out of kwargs, which is the root cause of the translate_url() issue. Candidate A passes by filtering None values in both RegexPattern and RoutePattern, while candidate B only fixes RegexPattern and fails when a custom converter's internal named subgroup is unmatched. The test reveals that the fix must apply consistently across both pattern types to handle the full scope of optional named group scenarios.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
