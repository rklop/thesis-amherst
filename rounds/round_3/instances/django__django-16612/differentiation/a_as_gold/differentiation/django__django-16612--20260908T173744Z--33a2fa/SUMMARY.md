# Differentiating-test run: `django__django-16612`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-16612:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/django__django-16612/a_as_gold/differentiation/django__django-16612--20260908T173744Z--33a2fa`
- Test: `test_missing_slash_append_slash_true_enum_query_string`
- Test command: `cd /testbed && python tests/runtests.py admin_views.tests.AdminSiteFinalCatchAllPatternTests.test_missing_slash_append_slash_true_enum_query_string`

## Specification gap

The query-preservation contract also applies when `request.META["QUERY_STRING"]` is an enum-backed value: the redirect must use the enum's string value instead of rejecting the request. The supplied gold patch explicitly normalizes values exposing `.value`, while candidate_b passes them to `get_full_path()`, whose URI quoting rejects a non-string Enum.

## Input/output contract

Input: An authenticated staff GET to the admin article changelist URL without its trailing slash, with `APPEND_SLASH=True` and `QUERY_STRING` set to an Enum member whose value is `id=123`.

Expected output: A 301 response with `Location: /test_admin/admin/admin_views/article/?id=123`.

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
