# Differentiating-test run: `django__django-11555`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-11555:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_pipeline_vs_manual_first100_20260818/runs/django__django-11555--20260818T092501Z--b11457`
- Test: `test_unwrapped_function_in_parent_ordering`
- Test command: `./tests/runtests.py ordering.test_related_expression_ordering.RelatedExpressionOrderingTests.test_unwrapped_function_in_parent_ordering`

## Specification gap

Related or parent-pointer ordering must support any valid Meta.ordering expression, including a bare database function, not only expressions already wrapped in OrderBy.

## Input/output contract

Input: Create multi-table child rows named 'Zebra' and 'alpha'. Order them by the inherited parent pointer, whose parent model declares Meta.ordering=(Lower('name'),).

Expected output: Evaluating the queryset returns ['alpha', 'Zebra'] without raising a type error.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | True | 0 |
| candidate_b | True | False | 1 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.95
- Summary: The test validates a real specification gap: parent pointer ordering must handle any valid Meta.ordering expression (including bare functions like Lower), not just strings or OrderBy-wrapped expressions. Candidate A correctly resolves and normalizes expressions, while B only handles pre-wrapped OrderBy and crashes on bare functions. The test's oracle (checking for correct alphabetical ordering without type errors) follows directly from the issue's complaint about crashes when Meta.ordering contains expressions during multi-table inheritance.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
