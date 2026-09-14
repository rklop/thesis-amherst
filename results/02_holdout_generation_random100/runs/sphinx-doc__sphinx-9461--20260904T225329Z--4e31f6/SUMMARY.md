# Differentiating-test run: `sphinx-doc__sphinx-9461`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sphinx-doc_1776_sphinx-9461:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_gold_pass_current_candidate_fail_random100_20260904/runs/sphinx-doc__sphinx-9461--20260904T225329Z--4e31f6`
- Test: `test_class_property_runtime_value_does_not_replace_descriptor`
- Test command: `cd /testbed && python -m pytest -q tests/test_ext_autodoc_autoproperty.py::test_class_property_runtime_value_does_not_replace_descriptor`

## Specification gap

Autodoc must inspect a classmethod-wrapped property independently of the value returned when its descriptor is accessed. Candidate A only recovers the descriptor when that value is None, while candidate B recovers it for any non-property value.

## Input/output contract

Input: Document Foo.class_prop through the public autoproperty entry point. Its getter has an int return annotation, a docstring, and returns the non-None value 42.

Expected output: The generated reStructuredText contains a py:property directive with :classmethod: and :type: int options, followed by the getter's 'class property docstring'.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.95
- Summary: The test exposes a genuine specification gap: autodoc must recover the classmethod-wrapped property descriptor regardless of the property's runtime return value. Candidate A fails because it only recovers the descriptor when `self.object is None`, missing cases where the property evaluates to a non-None value like 42. Candidate B passes by always checking `__dict__` for the raw descriptor when `inspect.isproperty` returns False. This is a meaningful edge case that follows from the issue's requirement to document such properties, and the expected output (`:classmethod:`, `:type: int`, docstring) is fully justified by the issue and Python/Sphinx API semantics.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
