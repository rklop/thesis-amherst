# test_serialize_enum_with_dynamic_name

- **Instance:** `django__django-11815`
- **Test ID:** `django__django-11815--56c844d2cbec0c5c`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

Enum serialization must produce code that remains resolvable through the enum's original module binding when reading the member's public name property changes class metadata. This narrowly clarifies the serializer's round-trip contract for customized Enum properties.

## Expected behavior

Serialization must emit a reference through migrations.test_writer.RenamingEnum and executing it must return the identical RenamingEnum.VALUE member. candidate_a reads the resolvable class name before the member name; candidate_b reads the member name first and emits the nonexistent migrations.test_writer.RenamedEnum binding.

## Test command

`cd /testbed && ./tests/runtests.py migrations.test_writer.WriterTests.test_serialize_enum_with_dynamic_name`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
