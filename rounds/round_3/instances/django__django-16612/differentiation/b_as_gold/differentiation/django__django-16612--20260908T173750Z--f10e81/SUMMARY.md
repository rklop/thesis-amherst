# Differentiating-test run: `django__django-16612`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-16612:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/django__django-16612/b_as_gold/differentiation/django__django-16612--20260908T173750Z--f10e81`
- Test: `test_missing_slash_preserves_encoded_question_mark_and_query`
- Test command: `./tests/runtests.py admin_views.tests.AdminSiteFinalCatchAllPatternTests.test_missing_slash_preserves_encoded_question_mark_and_query`

## Specification gap

When appending a slash, the redirect must preserve the distinction between percent-encoded path data and the actual query-string delimiter. Reconstructing the URL from the decoded request path can turn an encoded question mark into a query delimiter.

## Input/output contract

Input: An authenticated staff user requests the slashless admin change URL for object ID "?" (represented as `%3F` in the URL), with `?test=1` appended as the query string.

Expected output: A 301 response whose Location is `/test_admin/admin/admin_views/article/%3F/change/?test=1`: the slash is inserted before the query string, `%3F` remains path data, and the query string is preserved.

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
