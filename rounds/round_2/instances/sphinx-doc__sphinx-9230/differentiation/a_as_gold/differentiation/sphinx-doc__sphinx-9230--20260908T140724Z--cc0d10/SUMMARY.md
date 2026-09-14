# Differentiating-test run: `sphinx-doc__sphinx-9230`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sphinx-doc_1776_sphinx-9230:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/sphinx-doc__sphinx-9230/a_as_gold/differentiation/sphinx-doc__sphinx-9230--20260908T140724Z--cc0d10`
- Test: `test_info_field_list_untyped_param`
- Test command: `cd /testbed && python -m pytest -q tests/test_domain_py.py::test_info_field_list_untyped_param`

## Specification gap

The inline type in `:param [type] name:` is optional. A one-token field argument is solely the parameter name, not a declaration with an empty type.

## Input/output contract

Input: Parse a Python class directive containing `:param value: description`.

Expected output: The rendered parameter paragraph is exactly `value -- description`, without empty type parentheses.

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
