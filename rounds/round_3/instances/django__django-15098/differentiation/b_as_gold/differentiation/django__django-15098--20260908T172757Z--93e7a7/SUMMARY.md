# Differentiating-test run: `django__django-15098`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-15098:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/django__django-15098/b_as_gold/differentiation/django__django-15098--20260908T172757Z--93e7a7`
- Test: `test_language_path_with_script_region_and_modifier`
- Test command: `python tests/runtests.py i18n.tests.CountrySpecificLanguageTests.test_language_path_with_script_region_and_modifier`

## Specification gap

Language-script-region path prefixes must remain compatible with Django’s existing gettext-style @modifier suffix.

## Input/output contract

Input: GET /sr-latn-rs@latin/simple/ with sr-latn-rs@latin configured in LANGUAGES and the route inside i18n_patterns().

Expected output: The request resolves successfully with HTTP status 200. candidate_a rejects the combined third suffix and returns 404; candidate_b recognizes the complete language prefix.

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
