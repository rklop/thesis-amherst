# Differentiating-test run: `django__django-11999`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-11999:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_pipeline_vs_manual_first100_20260818/runs/django__django-11999--20260818T094159Z--7c9bc5`
- Test: `abstract_model_inherited_get_FIELD_display_override`
- Test command: `cd /testbed && python tests/runtests.py model_fields.tests.GetFieldDisplayTests.test_overriding_FIELD_display_inherited_from_abstract_model`

## Specification gap

Override behavior is underspecified for inherited methods. When a choices field is copied from an abstract model to a concrete subclass, Django must not shadow the abstract model’s custom get_<field>_display() method.

## Input/output contract

Input: Define an abstract model with an IntegerField whose choices map 1 to "foo", and override get_foo_bar_display() to return "something". Instantiate a concrete subclass with foo_bar=1 and call the inherited method.

Expected output: get_foo_bar_display() returns "something", not the automatically generated choice label "foo".

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.9
- Summary: The generated test exposes a meaningful semantic disagreement between candidates regarding Django's abstract model inheritance. Candidate A uses cls.__dict__ which only checks the concrete class and shadows inherited overrides, while candidate B uses hasattr which respects the inheritance chain. The test uses public Django API (abstract models) and has a clear, defensible oracle based on the issue's core requirement that users should be able to override get_FOO_display().

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
