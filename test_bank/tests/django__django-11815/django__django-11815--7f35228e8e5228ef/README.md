# test_serialize_enum_member_name_before_class_name

- **Instance:** `django__django-11815`
- **Test ID:** `django__django-11815--7f35228e8e5228ef`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

Enum serialization must resolve the member's stable name before constructing its class reference. This preserves an executable name-based reference when reading the member name finalizes lazy enum metadata.

## Expected behavior

MigrationWriter.serialize() returns ("migrations.test_writer.NameAwareEnum['VALUE']", {'import migrations.test_writer'}).

## Test command

`./tests/runtests.py migrations.test_writer.WriterTests.test_serialize_enum_member_name_before_class_name`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
