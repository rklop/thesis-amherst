# 044 — django__django-11477

## Comparison

- Manual reference: **Not different**
- Pipeline-derived bucket: **Very different**
- Exact agreement: **no**
- Ordinal distance: **2**

## Why each side classified it this way

**Manual review:** The candidate repeats the same filter in one additional matcher; the operative implementation is unchanged.

**Pipeline mapping:** A real pass/fail separation was found and MiniMax rated it high-signal.

## Differentiating test

- Test: `test_converter_ignores_unmatched_named_subgroups`
- Candidate that passed: **candidate_a**
- Specification gap tested: Optional named regex subgroups should be omitted consistently when resolving path() routes, including subgroups inside a registered custom converter. Only parameters declared by the route should be converted and exposed as view kwargs.
- Input: Register a custom path converter whose regex matches either "primary-<qualifier>" or "fallback". Resolve "/item/fallback/", leaving the converter's internal named "qualifier" subgroup unmatched.
- Expected behavior: resolve() succeeds with URL name "optional-subgroup" and kwargs {'value': 'FALLBACK'}. The unmatched internal subgroup is neither exposed nor treated as a route converter.

## MiniMax assessment

- Rating: **high_signal**
- Confidence: **0.9**
- Summary: The test validates that unmatched optional named groups in URL patterns (both re_path and path() with custom converters) are filtered out of kwargs, which is the root cause of the translate_url() issue. Candidate A passes by filtering None values in both RegexPattern and RoutePattern, while candidate B only fixes RegexPattern and fails when a custom converter's internal named subgroup is unmatched. The test reveals that the fix must apply consistently across both pattern types to handle the full scope of optional named group scenarios.

## Severe-disagreement adjudication

- Category: **Plausible difference; scope judgment required**
- Assessment: The candidate handles optional named groups in custom path converters as well as regex routes, while the issue does not explicitly define that wider scope.

## Interpretation boundary

Agreement is measured against a human-produced reference review, not an objective ground truth. This note summarizes recorded evidence and does not expose private model chain-of-thought.
