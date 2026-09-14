# Differentiating-test run: `django__django-11400`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-11400:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/django__django-11400/a_as_gold/differentiation/django__django-11400--20260908T134907Z--9f830e`
- Test: `test_get_choices_falls_back_to_falsy_nonempty_model_ordering`
- Test command: `cd /testbed && python tests/runtests.py model_fields.tests.GetChoicesOrderingTests.test_get_choices_falls_back_to_falsy_nonempty_model_ordering`

## Specification gap

A related model's Meta.ordering is an ordering sequence, not merely a boolean flag. A valid tuple subclass may be false-valued while still containing ordering terms; get_choices() must preserve and apply those terms when falling back to Meta.ordering.

## Input/output contract

Input: Create Foo rows with values 'a' and then 'b'. Configure Foo.Meta.ordering with a false-valued tuple containing '-a', then call Bar's foreign-key field get_choices(include_blank=False) without explicit ordering.

Expected output: The choices are ordered by '-a': Foo 'b' appears before Foo 'a'.

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
