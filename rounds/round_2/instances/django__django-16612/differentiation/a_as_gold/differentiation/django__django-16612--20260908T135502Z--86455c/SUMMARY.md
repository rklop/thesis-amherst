# Differentiating-test run: `django__django-16612`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-16612:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/django__django-16612/a_as_gold/differentiation/django__django-16612--20260908T135502Z--86455c`
- Test: `test_missing_slash_preserves_bytes_query_string`
- Test command: `python tests/runtests.py admin_views.tests.AdminSiteFinalCatchAllPatternTests.test_missing_slash_preserves_bytes_query_string`

## Specification gap

Query-string preservation must normalize UTF-8 bytes as URI data before composing the redirect, rather than interpolating Python's bytes representation into the URL.

## Input/output contract

Input: An authenticated staff GET to the slashless admin changelist with QUERY_STRING=b"q=caf\xc3\xa9".

Expected output: A 301 response with Location `/test_admin/admin/admin_views/article/?q=caf%C3%A9`.

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
