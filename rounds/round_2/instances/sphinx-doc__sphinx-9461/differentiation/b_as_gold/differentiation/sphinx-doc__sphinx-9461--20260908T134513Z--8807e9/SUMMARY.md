# Differentiating-test run: `sphinx-doc__sphinx-9461`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sphinx-doc_1776_sphinx-9461:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/sphinx-doc__sphinx-9461/b_as_gold/differentiation/sphinx-doc__sphinx-9461--20260908T134513Z--8807e9`
- Test: `test_class_property_uses_registered_attrgetter`
- Test command: `cd /testbed && python -m pytest -q tests/test_ext_autodoc_autoproperty.py::test_class_property_uses_registered_attrgetter`

## Specification gap

Documenting a @classmethod/@property must continue to route attribute lookup through a type-specific getter registered with the public add_autodoc_attrgetter API.

## Input/output contract

Input: A class property returning a string is documented with autoproperty while a registered type getter records requests for that member.

Expected output: Autodoc emits the class property directive and docstring, and the registered getter records lookup of Foo.classprop.

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
