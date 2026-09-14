# 093 — django__django-13128

## Comparison

- Manual reference: **Very different**
- Pipeline-derived bucket: **Very different**
- Exact agreement: **yes**
- Ordinal distance: **0**

## Why each side classified it this way

**Manual review:** One changes output metadata only; the other changes the expression class and backend SQL compilation route.

**Pipeline mapping:** A real pass/fail separation was found and MiniMax rated it high-signal.

## Differentiating test

- Test: `test_temporal_subtraction_explicit_output_field`
- Candidate that passed: **candidate_a**
- Specification gap tested: Automatic recognition of temporal subtraction must not discard an output_field explicitly supplied by the caller. The candidates disagree because candidate A preserves the original expression and its declared field, while candidate B replaces it during resolution with a TemporalSubtraction whose output is DurationField.
- Input: On SQLite, select Experiment e2 and annotate end - start with the expression's output_field explicitly set to IntegerField. The fixture interval is 44 seconds.
- Expected behavior: The annotation returns the SQLite integer representation of the interval: 44000000 microseconds.

## MiniMax assessment

- Rating: **high_signal**
- Confidence: **0.9**
- Summary: The test correctly identifies a genuine specification conflict: whether an explicitly set output_field should be honored or overridden by automatic temporal subtraction inference. Candidate A respects the caller's explicit output_field (returns 44000000 as IntegerField), while candidate B silently replaces with TemporalSubtraction returning timedelta. The issue requests making temporal subtraction work 'without ExpressionWrapper' - which means automatic inference should be convenient, but not override explicit user intent. The test is minimal, tests public API behavior (the annotation result), and has a clear oracle based on the explicit output_field setting.

## Interpretation boundary

Agreement is measured against a human-produced reference review, not an objective ground truth. This note summarizes recorded evidence and does not expose private model chain-of-thought.
