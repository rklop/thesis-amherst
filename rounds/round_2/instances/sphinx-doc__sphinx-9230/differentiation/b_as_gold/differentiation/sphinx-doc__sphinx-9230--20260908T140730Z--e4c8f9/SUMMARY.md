# Differentiating-test run: `sphinx-doc__sphinx-9230`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sphinx-doc_1776_sphinx-9230:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/sphinx-doc__sphinx-9230/b_as_gold/differentiation/sphinx-doc__sphinx-9230--20260908T140730Z--e4c8f9`
- Test: `test_info_field_list_nested_whitespace_is_not_separator`
- Test command: `cd /testbed && python -m pytest -q tests/test_domain_py.py::test_info_field_list_nested_whitespace_is_not_separator`

## Specification gap

A :param argument uses inline TYPE NAME syntax only when separated by top-level whitespace. Whitespace inside parentheses is part of the argument; without a top-level separator, the complete argument remains the parameter name and must match a separate :type field.

## Input/output contract

Input: Parse a Python function directive with `:param dict(str, str): optional metadata` and the separate declaration `:type dict(str, str): Mapping`.

Expected output: The rendered Parameters field body is exactly `dict(str, str) (Mapping) -- optional metadata`. Candidate A instead splits at the whitespace inside the parentheses and rearranges the name and type.

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
