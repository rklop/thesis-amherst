# Differentiating-test run: `sphinx-doc__sphinx-11445`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sphinx-doc_1776_sphinx-11445:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/sphinx-doc__sphinx-11445/b_as_gold/differentiation/sphinx-doc__sphinx-11445--20260908T172154Z--3a1fb0`
- Test: `test_docinfo_prefix_includes_separator`
- Test command: `cd /testbed && python -m pytest -q tests/test_util_rst_docinfo.py::test_docinfo_prefix_includes_separator`

## Specification gap

A leading docinfo prefix includes its separating whitespace, while a domain-role heading with no whitespace after the second colon is not docinfo.

## Input/output contract

Input: Match the valid docinfo line `:author: Sphinx team` and the reported heading form `:mod:`mypackage2`` using the reStructuredText docinfo recognizer.

Expected output: The valid docinfo match is exactly `:author: `, including its separator. The domain-role heading produces no match.

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
