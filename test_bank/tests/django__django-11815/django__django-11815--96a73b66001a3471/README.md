# test_serialize_enum_expression_and_import_are_consistent

- **Instance:** `django__django-11815`
- **Test ID:** `django__django-11815--96a73b66001a3471`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 3

## What it checks

Enum serialization must resolve the member name before rendering module-dependent output, so the serialized expression and its required import refer to the same resolved module.

## Expected behavior

MigrationWriter.serialize() returns ("migrations.test_writer.DynamicNameEnum['VALUE']", {'import migrations.test_writer'}).

## Test command

`python tests/runtests.py migrations.test_writer.WriterTests.test_serialize_enum_expression_and_import_are_consistent`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
