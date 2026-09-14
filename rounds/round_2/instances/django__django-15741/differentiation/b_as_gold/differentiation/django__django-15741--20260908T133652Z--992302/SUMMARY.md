# Differentiating-test run: `django__django-15741`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-15741:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/django__django-15741/b_as_gold/differentiation/django__django-15741--20260908T133652Z--992302`
- Test: `test_get_format_has_no_docstring`
- Test command: `./tests/runtests.py i18n.tests.FormattingTests.test_get_format_has_no_docstring`

## Specification gap

The authoritative patch places format_type coercion before the function's descriptive string literal. Consequently, that literal is no longer get_format's Python docstring and public introspection returns None.

## Input/output contract

Input: Import django.utils.formats.get_format and inspect its __doc__ attribute.

Expected output: get_format.__doc__ is None.

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
