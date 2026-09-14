# Differentiating-test run: `sphinx-doc__sphinx-11445`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sphinx-doc_1776_sphinx-11445:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_gold_pass_current_candidate_fail_random100_20260904/runs/sphinx-doc__sphinx-11445--20260904T223021Z--ce465a`
- Test: `test_rst_prolog_preserves_multiword_docinfo_field`
- Test command: `cd /testbed && python -m pytest -q tests/test_metadata.py::test_rst_prolog_preserves_multiword_docinfo_field`

## Specification gap

Initial reStructuredText docinfo may use valid multiword field names. Candidate A only recognizes single-word fields matching `:word: `, while candidate B uses Docutils' complete field-marker grammar.

## Input/output contract

Input: Build a document beginning with `:field name: metadata value` while `rst_prolog` injects a visible paragraph.

Expected output: The build environment must expose `field name` as document metadata with value `metadata value`, showing that the prolog was inserted after the initial field list. Candidate A inserts the paragraph first, so the later field is no longer docinfo; candidate B preserves it.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.9
- Summary: The generated test validates that multiword docinfo fields (like `:field name: value`) are preserved when rst_prolog is injected. Candidate B uses docutils' official FIELD_NAME_RE pattern and passes; candidate A's regex change breaks multiword fields. This tests a genuine API contract (document metadata extraction) rather than implementation details, and both candidates stem from the same root cause as the original bug: the overly-broad docinfo regex incorrectly matching domain roles like `:mod:` in headings.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
