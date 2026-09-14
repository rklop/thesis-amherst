# Differentiating-test run: `sphinx-doc__sphinx-11445`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sphinx-doc_1776_sphinx-11445:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/sphinx-doc__sphinx-11445/b_as_gold/differentiation/sphinx-doc__sphinx-11445--20260908T133944Z--a489c2`
- Test: `test_prepend_prolog_before_indented_field_list`
- Test command: `cd /testbed && python -m pytest -q tests/test_util_rst.py::test_prepend_prolog_before_indented_field_list`

## Specification gap

Only top-level docinfo may precede rst_prolog. An indented field-list-like body line is ordinary document content, so rst_prolog must remain before it. Candidate A's optional leading-whitespace match incorrectly treats that content as docinfo.

## Input/output contract

Input: Call prepend_prolog with a nonempty substitution-definition prolog and content beginning with the indented line `   :caption: body`, followed by a blank line and text.

Expected output: The resulting text lines are the prolog, a generated blank separator, and then all original content unchanged: `['.. |project| replace:: Sphinx', '', '   :caption: body', '', 'Uses |project|.']`.

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
