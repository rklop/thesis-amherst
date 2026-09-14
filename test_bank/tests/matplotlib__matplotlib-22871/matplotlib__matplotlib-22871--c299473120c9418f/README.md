# test_concise_formatter_months_without_january_show_year

- **Instance:** `matplotlib__matplotlib-22871`
- **Test ID:** `matplotlib__matplotlib-22871--c299473120c9418f`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 3

## What it checks

The direct ConciseDateFormatter.format_ticks entry point must retain the common year as its offset when month-level ticks omit January. The fix must also remain within the repository's configured 79-column source-line limit.

## Expected behavior

The tick labels are ['Feb', 'Mar'] and get_offset() returns '2021'; dates.py contains no source line longer than the repository-configured 79 columns.

## Test command

`cd /testbed && python -m pytest -q lib/matplotlib/tests/test_dates_year_offset.py`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
