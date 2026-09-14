# 088 — django__django-13033

## Comparison

- Manual reference: **Somewhat different**
- Pipeline-derived bucket: **Very different**
- Exact agreement: **no**
- Ordinal distance: **1**

## Why each side classified it this way

**Manual review:** They agree on relation_id but can treat nested relation__pk ordering differently.

**Pipeline mapping:** A real pass/fail separation was found and MiniMax rated it high-signal.

## Differentiating test

- Test: `test_order_by_nested_pk_relational_primary_key`
- Candidate that passed: **candidate_a**
- Specification gap tested: Whether the `pk` shortcut suppresses related-model default ordering when it appears as the final component of a nested lookup. Candidate A checks `pieces[-1] != 'pk'`; candidate B checks the entire lookup string and therefore treats `childarticle__pk` as an ordinary relation.
- Input: Create two `ChildArticle` rows in ascending primary-key order, with publication dates ordered oppositely, then order their parent `Article` rows by `childarticle__pk`.
- Expected behavior: The observable headline sequence is `['first', 'second']`, reflecting ascending child primary-key values rather than `Article.Meta.ordering`, whose leading `-pub_date` would reverse them.

## MiniMax assessment

- Rating: **high_signal**
- Confidence: **0.95**
- Summary: The test correctly validates that ordering by a nested pk field (`childarticle__pk`) should use ascending pk order rather than inheriting the related model's Meta ordering (descending pub_date). Candidate A passes and candidate B fails, which aligns with the specification that the pk shortcut should suppress default ordering at nested lookup endpoints. The test is minimal, targets the public API behavior, and has a clear oracle based on Django's documented pk shortcut semantics.

## Interpretation boundary

Agreement is measured against a human-produced reference review, not an objective ground truth. This note summarizes recorded evidence and does not expose private model chain-of-thought.
