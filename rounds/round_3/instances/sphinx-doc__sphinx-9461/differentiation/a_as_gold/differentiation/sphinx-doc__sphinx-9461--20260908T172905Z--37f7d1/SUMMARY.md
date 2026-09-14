# Differentiating-test run: `sphinx-doc__sphinx-9461`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sphinx-doc_1776_sphinx-9461:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/sphinx-doc__sphinx-9461/a_as_gold/differentiation/sphinx-doc__sphinx-9461--20260908T172905Z--37f7d1`
- Test: `test_autodata_class_property`
- Test command: `python -m pytest -q tests/test_ext_autodoc_autodata.py::test_autodata_class_property`

## Specification gap

A classmethod-wrapped property must expose its getter docstring not only through automatic property-member discovery, but also when addressed explicitly through the public `autodata` directive.

## Input/output contract

Input: Define `Foo.classprop` using `@classmethod` over `@property`, with the getter docstring `Class property documentation.`, then document `target.properties.Foo.classprop` using `autodata` with value rendering disabled.

Expected output: The generated autodoc content contains `Class property documentation.` Candidate_a unwraps the descriptor for `DataDocumenter`; candidate_b leaves it as a Python 3.9 `classmethod`, whose visible docstring is not the wrapped property's docstring.

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
