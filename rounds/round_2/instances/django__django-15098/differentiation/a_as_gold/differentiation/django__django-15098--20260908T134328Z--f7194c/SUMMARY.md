# Differentiating-test run: `django__django-15098`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-15098:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/django__django-15098/a_as_gold/differentiation/django__django-15098--20260908T134328Z--f7194c`
- Test: `test_get_language_from_path_rejects_overlong_subtag`
- Test command: `PYTHONDONTWRITEBYTECODE=1 ./tests/runtests.py i18n.tests.MiscTests.test_get_language_from_path_rejects_overlong_subtag`

## Specification gap

Language subtags parsed from URL prefixes are limited to eight characters. A malformed longer subtag must be rejected before generic-language fallback occurs.

## Input/output contract

Input: Call get_language_from_path('/en-abcdefghi/') with English configured. The second subtag contains nine characters.

Expected output: The function returns None. It must not interpret the malformed prefix as the supported generic language 'en'.

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
