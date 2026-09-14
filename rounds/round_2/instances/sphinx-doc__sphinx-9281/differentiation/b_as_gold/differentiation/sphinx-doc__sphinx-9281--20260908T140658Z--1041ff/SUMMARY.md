# Differentiating-test run: `sphinx-doc__sphinx-9281`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sphinx-doc_1776_sphinx-9281:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/sphinx-doc__sphinx-9281/b_as_gold/differentiation/sphinx-doc__sphinx-9281--20260908T140658Z--1041ff`
- Test: `test_signature_enum_default_uses_textual_member_name`
- Test command: `python -m pytest -q tests/test_util_inspect.py::test_signature_enum_default_uses_textual_member_name`

## Specification gap

Enum defaults should be rendered from the textual value of the member name. A valid string subclass used for `Enum.name` must not have its custom formatting protocol alter the displayed member identity.

## Input/output contract

Input: A function whose default is `State.ready`. The Enum exposes its correct member name as a `str` subclass whose `__format__` method returns `misformatted`.

Expected output: `stringify_signature` returns `(state=State.ready)`. Candidate B uses string conversion and should produce this output; candidate A invokes the custom formatting protocol and should instead produce `(state=State.misformatted)`.

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
