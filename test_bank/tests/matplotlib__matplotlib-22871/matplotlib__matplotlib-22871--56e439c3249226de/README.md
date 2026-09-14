# test_concise_formatter_january_year_label_with_array_like_show_offset

- **Instance:** `matplotlib__matplotlib-22871`
- **Test ID:** `matplotlib__matplotlib-22871--56e439c3249226de`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

When month-level ticks include January, ConciseDateFormatter should put the year in January's tick label and suppress the offset without needing to evaluate the offset preference. This clarifies that the presence of January determines where the year is displayed.

## Expected behavior

format_ticks returns ['2021', 'Feb'] and get_offset returns an empty string. candidate_a checks for January first and produces this output; candidate_b attempts to truth-test the multi-element array first and raises ValueError.

## Test command

`python -m pytest -q lib/matplotlib/tests/test_dates.py::test_concise_formatter_january_year_label_with_array_like_show_offset`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
