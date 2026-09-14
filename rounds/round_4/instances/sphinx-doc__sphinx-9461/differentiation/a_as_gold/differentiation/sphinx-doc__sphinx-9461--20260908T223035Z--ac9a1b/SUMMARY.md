# Differentiating-test run: `sphinx-doc__sphinx-9461`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sphinx-doc_1776_sphinx-9461:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-04/bidirectional-differentiation/instances/sphinx-doc__sphinx-9461/a_as_gold/differentiation/sphinx-doc__sphinx-9461--20260908T223035Z--ac9a1b`
- Test: `test_non_class_property_descriptor_lookup_is_preserved`
- Test command: `cd /testbed && python -m pytest -q tests/test_ext_autodoc_autoproperty.py::test_non_class_property_descriptor_lookup_is_preserved`

## Specification gap

Supporting classmethod-wrapped properties must preserve normal class-level descriptor lookup for unrelated members. When a descriptor resolves to a documented callable, autodoc should document the resolved callable as a method rather than treating the raw descriptor as an attribute.

## Input/output contract

Input: Autodocument DescriptorOwner with its explicit dynamic member. FunctionDescriptor.__get__ resolves dynamic to the documented zero-argument callable _resolved_descriptor.

Expected output: The generated output contains the method directive `.. py:method:: DescriptorOwner.dynamic()` and the text `Resolved descriptor docstring.`

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
