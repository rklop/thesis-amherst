# 068 — django__django-12209

## Comparison

- Manual reference: **Very different**
- Pipeline-derived bucket: **Very different**
- Exact agreement: **yes**
- Ordinal distance: **0**

## Why each side classified it this way

**Manual review:** The candidate changes ordinary explicit-PK save semantics; gold identifies fixture/raw saving as the failing condition.

**Pipeline mapping:** A real pass/fail separation was found and MiniMax rated it high-signal.

## Differentiating test

- Test: `test_raw_save_with_default_primary_key`
- Candidate that passed: **candidate_b**
- Specification gap tested: A raw model save must bypass the primary-key-default INSERT optimization regardless of how the primary key was assigned. Its behavior must not depend on detecting an explicit PK during Model.__init__() or assignment through the generic pk property.
- Input: Create a NaturalPKWithDefault row, construct another instance so its UUID default is generated normally, assign the existing UUID through the model's public id field, change name, and call save_base(raw=True).
- Expected behavior: The existing row is updated to name='updated', no duplicate-key error occurs, and the table still contains exactly one row.

## MiniMax assessment

- Rating: **high_signal**
- Confidence: **0.9**
- Summary: The test reveals a genuine specification gap: raw saves (used by serialization) must unconditionally bypass the pk-default INSERT optimization, regardless of how the pk was assigned. Candidate B correctly fixes this by adding `not raw` to the condition. Candidate A fails because its explicit-pk tracking doesn't cover direct attribute assignment (obj.id = value), which is a valid way to set pk in serialization workflows. The test exercises a legitimate entry point (direct field assignment) that represents real usage in loaddata/serialization.

## Interpretation boundary

Agreement is measured against a human-produced reference review, not an objective ground truth. This note summarizes recorded evidence and does not expose private model chain-of-thought.
