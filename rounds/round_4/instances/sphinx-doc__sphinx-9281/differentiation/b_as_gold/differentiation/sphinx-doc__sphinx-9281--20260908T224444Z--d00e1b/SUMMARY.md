# Differentiating-test run: `sphinx-doc__sphinx-9281`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.sphinx-doc_1776_sphinx-9281:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-04/bidirectional-differentiation/instances/sphinx-doc__sphinx-9281/b_as_gold/differentiation/sphinx-doc__sphinx-9281--20260908T224444Z--d00e1b`
- Test: `test_signature_mapping_enum_default`
- Test command: `python -m pytest -q tests/test_util_inspect.py::test_signature_mapping_enum_default`

## Specification gap

Enum formatting must not override Sphinx's established deterministic rendering for objects that are also dictionaries. A dict-valued Enum member used as a parameter default should retain the canonical, key-sorted mapping representation.

## Input/output contract

Input: Define MappingEnum as both dict and enum.Enum with OPTION = {'z': 1, 'a': 2}, then stringify a function signature whose option parameter defaults to MappingEnum.OPTION.

Expected output: The signature is exactly "(option={'a': 2, 'z': 1})". candidate_b checks specialized container rendering before Enum rendering and therefore preserves this output; candidate_a instead produces "(option=MappingEnum.OPTION)".

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
