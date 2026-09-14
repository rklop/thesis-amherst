# test_concise_formatter_respects_show_offset_false_for_months

- **Instance:** `matplotlib__matplotlib-22871`
- **Test ID:** `matplotlib__matplotlib-22871--eb6ce521b567d4bf`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

The documented show_offset=False option must remain authoritative when formatting a sub-year range whose month ticks omit January; automatic year-offset selection must not re-enable an explicitly disabled offset.

## Expected behavior

After formatting the ticks, get_offset() returns the empty string. candidate_a preserves the caller's false setting, while candidate_b changes it to true for this month pattern and produces "2021".

## Test command

`cd /testbed && python -m pytest -q lib/matplotlib/tests/test_dates.py::test_concise_formatter_respects_show_offset_false_for_months`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
