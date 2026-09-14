# test_concise_formatter_months_without_january_preserves_year

- **Instance:** `matplotlib__matplotlib-22871`
- **Test ID:** `matplotlib__matplotlib-22871--ff16774d69189f59`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

For month-level ticks, ConciseDateFormatter must keep the date complete: when January is absent and therefore no tick label contains the year, the year offset is required even if show_offset was initialized false.

## Expected behavior

format_ticks returns ['Feb', 'Mar'], and get_offset returns '2021'.

## Test command

`cd /testbed && python -m pytest -q lib/matplotlib/tests/test_dates.py::test_concise_formatter_months_without_january_preserves_year`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
