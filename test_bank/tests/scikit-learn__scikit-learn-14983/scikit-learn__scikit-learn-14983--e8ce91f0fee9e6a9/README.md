# test_repeated_split_preserves_cvargs_keyword

- **Instance:** `scikit-learn__scikit-learn-14983`
- **Test ID:** `scikit-learn__scikit-learn-14983--e8ce91f0fee9e6a9`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

Adding repr support must expose the wrapper's n_splits without promoting every delegated cross-validator keyword to a wrapper attribute. `_RepeatedSplits` documents that arbitrary constructor parameters are forwarded through `**cvargs`; therefore, a legitimate forwarded parameter named `cvargs` must not overwrite the wrapper's internal forwarding dictionary.

## Expected behavior

`get_n_splits()` returns 6: the delegated value 3 multiplied by 2 repetitions. candidate_b instead overwrites the internal `cvargs` mapping with integer 3 and raises TypeError when attempting `**self.cvargs`.

## Test command

`cd /testbed && python -m pytest -q sklearn/model_selection/tests/test_repeated_split_cvargs.py::test_repeated_split_preserves_cvargs_keyword`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
