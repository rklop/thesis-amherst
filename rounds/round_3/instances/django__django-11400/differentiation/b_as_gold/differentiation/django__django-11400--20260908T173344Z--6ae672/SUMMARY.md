# Differentiating-test run: `django__django-11400`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-11400:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/django__django-11400/b_as_gold/differentiation/django__django-11400--20260908T173344Z--6ae672`
- Test: `test_get_choices_model_ordering_uses_resolved_related_model`
- Test command: `cd /testbed && ./tests/runtests.py model_fields.tests.GetChoicesOrderingTests.test_get_choices_model_ordering_uses_resolved_related_model`

## Specification gap

Fallback ordering must come from the same resolved related model whose objects populate the choices. candidate_a performs a second target-model lookup, while candidate_b consistently reuses the initially resolved model.

## Input/output contract

Input: Call ForeignKey.get_choices(include_blank=False) without explicit ordering through lazily resolving relation metadata. The choice model Foo has Meta.ordering equivalent to ('-a',); a subsequent model resolution exposes Bar with ordering ('a',).

Expected output: Choices contain foo2 followed by foo1, matching descending Foo.a ordering. candidate_a instead applies the later ascending ordering and returns the opposite order.

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
