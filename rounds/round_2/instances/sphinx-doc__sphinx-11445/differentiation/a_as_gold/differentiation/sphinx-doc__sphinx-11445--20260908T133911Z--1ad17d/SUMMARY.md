# Differentiating-test run: `sphinx-doc__sphinx-11445`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sphinx-doc_1776_sphinx-11445:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/sphinx-doc__sphinx-11445/a_as_gold/differentiation/sphinx-doc__sphinx-11445--20260908T133911Z--1ad17d`
- Test: `test_rst_prolog_preserves_colon_delimited_heading`
- Test command: `cd /testbed && python -m pytest -q tests/test_markup.py::test_rst_prolog_preserves_colon_delimited_heading`

## Specification gap

A leading `:name:` prefix is document metadata only when its closing colon is followed by whitespace or end-of-line. Without that separator, the line remains ordinary reStructuredText content and must stay adjacent to a following heading underline.

## Input/output contract

Input: Build HTML with a nonempty `rst_prolog` and a toctree child whose top-level heading is the plain text `:RFC:2822`. The `2` immediately after the second colon makes this a heading, not a docinfo field.

Expected output: The generated index page contains a link to `colon-heading.html` whose visible caption is `:RFC:2822`.

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
