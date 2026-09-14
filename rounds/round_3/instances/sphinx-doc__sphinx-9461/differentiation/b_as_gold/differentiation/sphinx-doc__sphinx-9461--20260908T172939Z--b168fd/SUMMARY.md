# Differentiating-test run: `sphinx-doc__sphinx-9461`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sphinx-doc_1776_sphinx-9461:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/sphinx-doc__sphinx-9461/b_as_gold/differentiation/sphinx-doc__sphinx-9461--20260908T172939Z--b168fd`
- Test: `test_private_class_property`
- Test command: `cd /testbed && python -m pytest -q tests/test_ext_autodoc_autoproperty.py::test_private_class_property`

## Specification gap

Explicit autodoc lookup of a private class property must honor Python name mangling before recovering the `classmethod(property(...))` descriptor. Otherwise Sphinx documents the computed value instead of the property.

## Input/output contract

Input: Document `target.properties.PrivateClassProperty.__prop`, a name-mangled `@classmethod @property` returning `42`, annotated as `int`, and carrying its own getter docstring.

Expected output: Autodoc output contains a `py:property` declaration for `PrivateClassProperty.__prop`, the `:classmethod:` and `:type: int` options, and `private class property docstring`.

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
