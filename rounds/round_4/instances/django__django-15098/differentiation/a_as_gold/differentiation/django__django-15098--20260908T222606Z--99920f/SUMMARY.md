# Differentiating-test run: `django__django-15098`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-15098:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-04/bidirectional-differentiation/instances/django__django-15098/a_as_gold/differentiation/django__django-15098--20260908T222606Z--99920f`
- Test: `test_get_language_from_path_variant_fallback`
- Test command: `python tests/runtests.py i18n.tests.MiscTests.test_get_language_from_path_variant_fallback`

## Specification gap

An unlisted BCP 47 tag with a variant subtag should retain the established fallback to its configured, immediately less-specific language tag. Three or more subtags must not be rejected categorically.

## Input/output contract

Input: Configure `de` and `de-at`, then call the public `translation.get_language_from_path()` with `/de-at-1996/`, where `1996` is a BCP 47 variant subtag.

Expected output: The function returns `de-at`, the configured parent language tag.

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
