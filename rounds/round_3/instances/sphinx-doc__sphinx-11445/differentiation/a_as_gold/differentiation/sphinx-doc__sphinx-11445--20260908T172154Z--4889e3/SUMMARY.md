# Differentiating-test run: `sphinx-doc__sphinx-11445`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sphinx-doc_1776_sphinx-11445:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/sphinx-doc__sphinx-11445/a_as_gold/differentiation/sphinx-doc__sphinx-11445--20260908T172154Z--4889e3`
- Test: `test_docinfo_re_matches_only_field_marker`
- Test command: `cd /testbed && python -m pytest -q tests/test_util_rst.py::test_docinfo_re_matches_only_field_marker`

## Specification gap

A docinfo match should include only the field marker, not the whitespace separating it from the field body.

## Input/output contract

Input: Match the valid reStructuredText docinfo line ':author: Sphinx team' with the docinfo detector.

Expected output: The match succeeds and its matched text is exactly ':author:'. candidate_a preserves this boundary with a zero-width lookahead; candidate_b incorrectly includes the following space.

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
