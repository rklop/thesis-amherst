# Differentiating-test run: `scikit-learn__scikit-learn-14983`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.scikit-learn_1776_scikit-learn-14983:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/scikit-learn__scikit-learn-14983/a_as_gold/differentiation/scikit-learn__scikit-learn-14983--20260908T133154Z--63c391`
- Test: `test_repeated_split_preserves_cvargs_keyword`
- Test command: `cd /testbed && python -m pytest -q sklearn/model_selection/tests/test_repeated_split_cvargs.py::test_repeated_split_preserves_cvargs_keyword`

## Specification gap

Adding repr support must expose the wrapper's n_splits without promoting every delegated cross-validator keyword to a wrapper attribute. `_RepeatedSplits` documents that arbitrary constructor parameters are forwarded through `**cvargs`; therefore, a legitimate forwarded parameter named `cvargs` must not overwrite the wrapper's internal forwarding dictionary.

## Input/output contract

Input: Construct `_RepeatedSplits` with two repetitions around a concrete splitter whose constructor accepts `cvargs=3` and whose `get_n_splits()` returns that value.

Expected output: `get_n_splits()` returns 6: the delegated value 3 multiplied by 2 repetitions. candidate_b instead overwrites the internal `cvargs` mapping with integer 3 and raises TypeError when attempting `**self.cvargs`.

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
