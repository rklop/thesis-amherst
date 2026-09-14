# Differentiating-test run: `django__django-13297`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-13297:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/django__django-13297/b_as_gold/differentiation/django__django-13297--20260908T172154Z--a69095`
- Test: `test_lazy_parameter_preserves_wrapped_bytes`
- Test command: `cd /testbed && ./tests/runtests.py backends.sqlite.tests.Tests.test_lazy_parameter_preserves_wrapped_bytes`

## Specification gap

A SimpleLazyObject used as a database parameter must preserve the wrapped value's database-relevant type. Resolving every lazy value with str() avoids the reported string-slug crash but corrupts non-string values such as bytes.

## Input/output contract

Input: Bind SimpleLazyObject(lambda: b'\x00\xff') as a parameter through Django's public SQLite cursor API and select it back.

Expected output: The fetched value is exactly b'\x00\xff'. candidate_b delegates SQLite adaptation to the wrapped bytes value, while candidate_a's blanket str adapter returns text representing the bytes literal.

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
