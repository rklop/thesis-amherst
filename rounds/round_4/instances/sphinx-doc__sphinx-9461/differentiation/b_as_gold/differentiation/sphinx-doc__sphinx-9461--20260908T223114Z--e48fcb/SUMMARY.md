# Differentiating-test run: `sphinx-doc__sphinx-9461`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sphinx-doc_1776_sphinx-9461:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-04/bidirectional-differentiation/instances/sphinx-doc__sphinx-9461/b_as_gold/differentiation/sphinx-doc__sphinx-9461--20260908T223114Z--e48fcb`
- Test: `test_class_property_returning_property_uses_descriptor_docstring`
- Test command: `pytest -q tests/test_ext_autodoc_autoproperty.py::test_class_property_returning_property_uses_descriptor_docstring`

## Specification gap

A classmethod-wrapped property must be documented from its declared descriptor, even when evaluating it returns another property object.

## Input/output contract

Input: Explicitly autodocument a @classmethod @property whose getter docstring is "Class property descriptor docstring." but whose runtime value is a different property with docstring "Runtime property value docstring."

Expected output: The generated py:property output includes the :classmethod: marker and the declared getter's docstring, and excludes the runtime value's docstring.

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
