# 065 — django__django-12143

## Comparison

- Manual reference: **Not different**
- Pipeline-derived bucket: **Very different**
- Exact agreement: **no**
- Ordinal distance: **2**

## Why each side classified it this way

**Manual review:** Both neutralize the user-relevant regex characters; Django field names cannot supply the extra characters escaped by the candidate.

**Pipeline mapping:** A real pass/fail separation was found and MiniMax rated it high-signal.

## Differentiating test

- Test: `test_get_edited_object_pks_with_regex_chars_in_pk_name`
- Candidate that passed: **candidate_a**
- Specification gap tested: Regex metacharacters must be treated literally not only in a formset prefix, but in every model-derived component used to recognize list-editable POST keys. Django permits dynamically declared field names such as "serial$", but candidate B interpolates the primary-key name into the regex unescaped.
- Input: A dynamically constructed model has an IntegerField primary key named "serial$". The POST contains the genuine key "form-0-serial$" with value "7" and a regex-lookalike key "form-1-serial" with value "8".
- Expected behavior: _get_edited_object_pks() must return ["7"], selecting the literal primary-key field and rejecting the lookalike without the dollar sign. Candidate B instead treats "$" as an end-of-string anchor and returns ["8"].

## MiniMax assessment

- Rating: **high_signal**
- Confidence: **0.95**
- Summary: The test correctly identifies that candidate_b's fix is incomplete - it only escapes the prefix but not the pk.name, leading to incorrect matching when pk.name contains regex metacharacters like '$'. The test reveals a meaningful missing specification: while the original issue focused on prefix escaping, the same logic applies to pk.name since Django allows dynamically declared field names. The test passes candidate_a (full escape) and fails candidate_b (partial escape), correctly identifying the more specification-conformant solution.

## Severe-disagreement adjudication

- Category: **Plausible difference; scope judgment required**
- Assessment: Escaping a dynamically installed primary-key name containing '$' reveals broader regex safety, but normal declared field names do not use that form.

## Interpretation boundary

Agreement is measured against a human-produced reference review, not an objective ground truth. This note summarizes recorded evidence and does not expose private model chain-of-thought.
