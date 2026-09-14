# Differentiating-test run: `django__django-15098`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-15098:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/django__django-15098/b_as_gold/differentiation/django__django-15098--20260908T134334Z--4c9234`
- Test: `test_get_language_from_request_path_gettext_modifier`
- Test command: `cd /testbed && python tests/runtests.py i18n.tests.CountrySpecificLanguageTests.test_get_language_from_request_path_gettext_modifier`

## Specification gap

Adding support for three-part language prefixes must preserve Django's existing gettext-style @modifier syntax. The generated candidate accepts only hyphen-separated components, although the repository already recognizes sr-RS@latin as a valid language code.

## Input/output contract

Input: Configure sr-RS@latin in LANGUAGES and pass a request for /sr-RS@latin/ to the public get_language_from_request() API with path checking enabled.

Expected output: get_language_from_request(request, check_path=True) returns the exact configured language code 'sr-RS@latin'.

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
