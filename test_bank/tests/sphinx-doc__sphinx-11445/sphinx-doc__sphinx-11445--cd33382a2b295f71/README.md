# test_prepend_prolog_before_indented_field_list

- **Instance:** `sphinx-doc__sphinx-11445`
- **Test ID:** `sphinx-doc__sphinx-11445--cd33382a2b295f71`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

Only top-level docinfo may precede rst_prolog. An indented field-list-like body line is ordinary document content, so rst_prolog must remain before it. Candidate A's optional leading-whitespace match incorrectly treats that content as docinfo.

## Expected behavior

The resulting text lines are the prolog, a generated blank separator, and then all original content unchanged: `['.. |project| replace:: Sphinx', '', ' :caption: body', '', 'Uses |project|.']`.

## Test command

`cd /testbed && python -m pytest -q tests/test_util_rst.py::test_prepend_prolog_before_indented_field_list`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
