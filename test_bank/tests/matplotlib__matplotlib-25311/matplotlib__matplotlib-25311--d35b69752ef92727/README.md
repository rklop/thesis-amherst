# test_pickle_while_dragging_annotation_with_blit

- **Instance:** `matplotlib__matplotlib-25311`
- **Test ID:** `matplotlib__matplotlib-25311--d35b69752ef92727`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

A draggable figure must remain pickleable while a blitted drag is actively in progress, not only while the draggable helper is idle. Transient backend blit state must not become part of the serialized figure.

## Expected behavior

The pick event targets the annotation and pickle.dumps(fig) returns successfully without an exception. In particular, the transient backend blit buffer must not cause a pickling TypeError.

## Test command

`python -m pytest -q lib/matplotlib/tests/test_pickle.py::test_pickle_while_dragging_annotation_with_blit`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
