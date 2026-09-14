# 053 — django__django-11790

## Comparison

- Manual reference: **Not different**
- Pipeline-derived bucket: **Very different**
- Exact agreement: **no**
- Ordinal distance: **2**

## Why each side classified it this way

**Manual review:** Both expose the exact same maximum length in HTML.

**Pipeline mapping:** A real pass/fail separation was found and MiniMax rated it high-signal.

## Differentiating test

- Test: `test_custom_username_field_effective_max_length_in_html`
- Candidate that passed: **candidate_a**
- Specification gap tested: For an AuthenticationForm subclass with a custom username field that normalizes assigned maximum lengths, the rendered HTML maxlength should reflect the form field's effective validation limit, not the unnormalized user-model value.
- Input: Instantiate a custom AuthenticationForm whose username CharField caps every assigned max_length at 32. The default user model supplies 150, while the custom field retains an effective limit and validator of 32.
- Expected behavior: The rendered username input contains maxlength="32". Candidate A reads the effective form-field value after assignment; Candidate B instead renders the captured model value, maxlength="150".

## MiniMax assessment

- Rating: **high_signal**
- Confidence: **0.95**
- Summary: The test correctly identifies a meaningful semantic difference between candidates. Candidate A reads the form field's max_length after assignment (which respects custom field transformations), while Candidate B captures the raw model value before assignment. The test uses a legitimate Django subclassing pattern (custom AuthenticationForm with custom username field) to verify that HTML maxlength reflects the effective form-field limit, not the unnormalized model value. Candidate A passes and correctly implements the expected behavior - the widget attribute should reflect what was actually set on the form field, not the pre-normalized value.

## Severe-disagreement adjudication

- Category: **Strong new behavioral evidence**
- Assessment: A supported custom form-field normalization shows that only the candidate renders the effective validation limit into HTML.

## Interpretation boundary

Agreement is measured against a human-produced reference review, not an objective ground truth. This note summarizes recorded evidence and does not expose private model chain-of-thought.
