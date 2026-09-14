# Differentiating-test run: `sphinx-doc__sphinx-9258`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sphinx-doc_1776_sphinx-9258:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/sphinx-doc__sphinx-9258/b_as_gold/differentiation/sphinx-doc__sphinx-9258--20260908T134101Z--2ae6a2`
- Test: `test_info_field_pipe_union_quoted_forward_reference`
- Test command: `cd /testbed && python -m pytest -q tests/test_domain_py.py::test_info_field_pipe_union_quoted_forward_reference`

## Specification gap

Pipe-separated Python type fields follow Python annotation semantics for each operand. In particular, a quoted forward-reference operand denotes the referenced type; the quote characters are not part of the cross-reference target.

## Input/output contract

Input: Build HTML for a function whose `:type value:` field is `bytes | "Payload"`, with `Payload` documented as a Python class on `types.rst`.

Expected output: The rendered union type contains a working link to `types.html#Payload`. The supplied gold parses the quoted operand as the type name `Payload`; the generated candidate instead tries to resolve the literal target `"Payload"`.

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
