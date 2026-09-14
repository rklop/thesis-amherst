# Differentiating-test run: `django__django-11149`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-11149:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_pipeline_vs_manual_first100_20260818/runs/django__django-11149--20260818T090402Z--3f9b4d`
- Test: `test_inline_change_m2m_private_method_name_collision`
- Test command: `python tests/runtests.py admin_inlines.tests.TestInlinePermissions.test_inline_change_m2m_private_method_name_collision`

## Specification gap

Fixing auto-created many-to-many permissions must preserve compatibility with existing InlineModelAdmin subclasses. A subclass-private method that Django previously ignored must not unexpectedly become an internal permission hook.

## Input/output contract

Input: Create a TabularInline for Author.books.through whose subclass already defines `_has_change_permission(self, request)`. Grant the user change_book permission, then call the public `has_change_permission(request)` entry point.

Expected output: The permission check returns True without invoking the unrelated subclass-private method. Candidate A instead dispatches to that method with an additional obj argument and raises TypeError.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## MiniMax judgment

- Rating: **low_signal**
- Confidence: 0.75
- Summary: The test exposes a real backward-compatibility issue in candidate A (method name collision with user subclasses), but it tests implementation internals rather than the core permission bug described in the issue. The test is narrowly focused on a specific helper method name conflict that would be rare in practice, rather than validating the intended permission semantics for auto-created M2M inlines.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
