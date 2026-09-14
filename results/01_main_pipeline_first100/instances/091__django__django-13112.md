# 091 — django__django-13112

## Comparison

- Manual reference: **Not different**
- Pipeline-derived bucket: **Very different**
- Exact agreement: **no**
- Ordinal distance: **2**

## Why each side classified it this way

**Manual review:** Valid relation labels contain one separator, so split and rsplit implement the same rule.

**Pipeline mapping:** A real pass/fail separation was found and MiniMax rated it high-signal.

## Differentiating test

- Test: `test_foreign_key_deconstruct_dotted_mixed_case_app_label`
- Candidate that passed: **candidate_a**
- Specification gap tested: ForeignKey deconstruction must preserve the complete, case-sensitive app label while normalizing only the model name, including when a valid app label contains dots. Its serialized output must remain stable when reconstructed through Field.clone().
- Input: Create a model with app_label='package.MixedCaseApp', create a ForeignKey to its class, clone the field so the relation becomes its serialized string form, and deconstruct the clone.
- Expected behavior: The reconstructed field deconstructs with kwargs['to'] equal to 'package.MixedCaseApp.target'. Candidate B instead raises ValueError because split('.') produces three components.

## MiniMax assessment

- Rating: **high_signal**
- Confidence: **0.95**
- Summary: The test correctly identifies that candidate A properly handles dotted app labels with mixed case by using rsplit('.', 1), while candidate B incorrectly uses split('.') which fails on multiple dots. The test verifies the specification requirement: ForeignKey deconstruction must preserve complete case-sensitive app labels while normalizing only the model name. Candidate A passes because it correctly separates the app label from the model name at the rightmost dot, preserving the app label's case. Candidate B fails with 'too many values to unpack' because split('.') produces three components when given 'package.MixedCaseApp.target'.

## Severe-disagreement adjudication

- Category: **Questionable or out-of-domain test**
- Assessment: The differentiator relies on a dotted app label, while Django app labels are required to be valid Python identifiers.

## Interpretation boundary

Agreement is measured against a human-produced reference review, not an objective ground truth. This note summarizes recorded evidence and does not expose private model chain-of-thought.
