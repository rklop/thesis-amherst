# Differentiating-test run: `sphinx-doc__sphinx-9258`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sphinx-doc_1776_sphinx-9258:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/sphinx-doc__sphinx-9258/a_as_gold/differentiation/sphinx-doc__sphinx-9258--20260908T134013Z--f68bf5`
- Test: `test_info_field_list_pipe_union_preserves_abbreviated_xrefs`
- Test command: `cd /testbed && python -m pytest -q tests/test_domain_py.py::test_info_field_list_pipe_union_preserves_abbreviated_xrefs`

## Specification gap

Pipe-separated type fields must preserve Sphinx's per-reference `~` abbreviation semantics. Each union member remains an independent cross-reference whose displayed label omits its qualified prefix.

## Input/output contract

Input: Parse a `py:function` parameter field with `:type value: ~package.First | ~package.Second`.

Expected output: The rendered type is `First | Second`, with two class references targeting `package.First` and `package.Second` respectively.

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
