# test_setup_show_bytes_parameter_not_truncated

- **Instance:** `pytest-dev__pytest-7205`
- **Test ID:** `pytest-dev__pytest-7205--6dc0038e6e311659`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

Using a safe representation must not introduce arbitrary truncation for modest fixture parameters. A 40-byte payload has a 43-character repr including b'' syntax; candidate_b's maxsize=42 loses content, while candidate_a preserves it.

## Expected behavior

The run succeeds without BytesWarning, and stdout contains the complete setup line ending with SETUP F data[b'0123456789012345678901234567890123456789'].

## Test command

`cd /testbed && python -bb -m pytest -q testing/test_setuponly.py::test_setup_show_bytes_parameter_not_truncated`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
