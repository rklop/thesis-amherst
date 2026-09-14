# test_serialize_compiled_regex_with_combined_flags

- **Instance:** `django__django-11815`
- **Test ID:** `django__django-11815--7f22bb5835f57a93`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

Migration serialization must round-trip compiled regular expressions with combined non-default flags. A regex pattern’s flags are exposed as an integer bitmask, not necessarily as a named enum member.

## Expected behavior

Deserialization succeeds and produces a compiled regex equal to the original, preserving its pattern and both flags.

## Test command

`python tests/runtests.py migrations.test_writer.WriterTests.test_serialize_compiled_regex_with_combined_flags`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
