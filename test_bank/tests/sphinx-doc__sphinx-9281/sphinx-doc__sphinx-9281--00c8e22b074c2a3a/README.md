# test_signature_enum_default_uses_textual_member_name

- **Instance:** `sphinx-doc__sphinx-9281`
- **Test ID:** `sphinx-doc__sphinx-9281--00c8e22b074c2a3a`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

Enum defaults should be rendered from the textual value of the member name. A valid string subclass used for `Enum.name` must not have its custom formatting protocol alter the displayed member identity.

## Expected behavior

`stringify_signature` returns `(state=State.ready)`. Candidate B uses string conversion and should produce this output; candidate A invokes the custom formatting protocol and should instead produce `(state=State.misformatted)`.

## Test command

`python -m pytest -q tests/test_util_inspect.py::test_signature_enum_default_uses_textual_member_name`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
