# Differentiating-test run: `sphinx-doc__sphinx-9281`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sphinx-doc_1776_sphinx-9281:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/sphinx-doc__sphinx-9281/b_as_gold/differentiation/sphinx-doc__sphinx-9281--20260908T175714Z--2aaa79`
- Test: `test_enum_default_uses_public_member_name`
- Test command: `cd /testbed && python -m pytest -q tests/test_util_inspect_enum.py`

## Specification gap

Enum defaults should be rendered from the member's public `name` attribute, not the private `_name_` storage. This matters when an Enum validly customizes which alias its public name exposes.

## Input/output contract

Input: A function whose default is `Mode.ValueA`; `Mode` has an equivalent alias `AliasForValueA`, and its public `name` property selects that alias.

Expected output: The rendered signature is `(value=Mode.AliasForValueA)`. This remains an evaluable Enum reference to the original default value.

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
