# test_enum_default_uses_public_member_name

- **Instance:** `sphinx-doc__sphinx-9281`
- **Test ID:** `sphinx-doc__sphinx-9281--237d9559be70e8fc`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 3

## What it checks

Enum defaults should be rendered from the member's public `name` attribute, not the private `_name_` storage. This matters when an Enum validly customizes which alias its public name exposes.

## Expected behavior

The rendered signature is `(value=Mode.AliasForValueA)`. This remains an evaluable Enum reference to the original default value.

## Test command

`cd /testbed && python -m pytest -q tests/test_util_inspect_enum.py`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
