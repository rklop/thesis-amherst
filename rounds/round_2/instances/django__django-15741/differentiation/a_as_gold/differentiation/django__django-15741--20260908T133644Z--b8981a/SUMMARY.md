# Differentiating-test run: `django__django-15741`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-15741:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/django__django-15741/a_as_gold/differentiation/django__django-15741--20260908T133644Z--b8981a`
- Test: `test_get_format_keeps_docstring`
- Test command: `cd /testbed && python tests/runtests.py i18n.tests.FormattingTests.test_get_format_keeps_docstring`

## Specification gap

Accepting a lazy format_type must not remove get_format()'s existing public documentation metadata. candidate_b places executable code before the function docstring, causing get_format.__doc__ to become None.

## Input/output contract

Input: Inspect the public get_format callable's docstring and check that it still documents the format_type parameter.

Expected output: get_format.__doc__ contains "format_type is the name of the format". candidate_a preserves this output; candidate_b exposes None instead.

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
