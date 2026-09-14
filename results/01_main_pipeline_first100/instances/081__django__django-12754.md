# 081 — django__django-12754

## Comparison

- Manual reference: **Very different**
- Pipeline-derived bucket: **Very different**
- Exact agreement: **yes**
- Ordinal distance: **0**

## Why each side classified it this way

**Manual review:** Gold directly encodes the set relation; the candidate uses indirect global key membership that can classify other migration shapes differently.

**Pipeline mapping:** A real pass/fail separation was found and MiniMax rated it high-signal.

## Differentiating test

- Test: `test_add_model_with_field_removed_from_mixed_case_base_model`
- Candidate that passed: **candidate_a**
- Specification gap tested: Moving a field from a concrete base to a newly created subclass must order RemoveField before CreateModel even when the subclass's public string base reference uses the model's original mixed-case name. Django treats string model references case-insensitively, so dependency detection must do the same.
- Input: The old state contains Readable with a title field. The new state removes Readable.title and creates Book with its own title field, inheriting through the mixed-case string reference 'app.Readable'.
- Expected behavior: The autodetector produces one app migration whose operations are exactly RemoveField followed by CreateModel, preventing the inherited-field clash.

## MiniMax assessment

- Rating: **high_signal**
- Confidence: **1.0**
- Summary: The test validates that the autodetector correctly orders migration operations when moving a field from a concrete base model to a newly created subclass, specifically testing case-insensitivity of model references. Candidate A passes because it normalizes base model names to lowercase before ProjectState lookup, matching Django's internal key handling. Candidate B fails because it uses the mixed-case name directly, causing the dependency check to miss the required ordering constraint.

## Interpretation boundary

Agreement is measured against a human-produced reference review, not an objective ground truth. This note summarizes recorded evidence and does not expose private model chain-of-thought.
