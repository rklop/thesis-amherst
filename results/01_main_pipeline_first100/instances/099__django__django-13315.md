# 099 — django__django-13315

## Comparison

- Manual reference: **Very different**
- Pipeline-derived bucket: **Very different**
- Exact agreement: **yes**
- Ordinal distance: **0**

## Why each side classified it this way

**Manual review:** The candidate changes generic choice query semantics and relies on DISTINCT; gold avoids duplicates without broadening those APIs.

**Pipeline mapping:** A real pass/fail separation was found and MiniMax rated it high-signal.

## Differentiating test

- Test: `test_get_choices_removes_duplicates`
- Candidate that passed: **candidate_a**
- Specification gap tested: Duplicate suppression should also hold when choices are generated through the public relation Field.get_choices() API, not only when a ModelForm constructs its queryset.
- Input: Create a second Bar pointing to foo1, then call the ForeignKey field's get_choices(include_blank=False) with Q(bars__isnull=False). The join matches foo1 twice and foo2 once.
- Expected behavior: The returned choices are exactly [(foo1.pk, str(foo1)), (foo2.pk, str(foo2))], with one tuple per related object.

## MiniMax assessment

- Rating: **high_signal**
- Confidence: **0.95**
- Summary: The test exercises the Field.get_choices() public API with a Q object involving a join (bars__isnull=False), matching the issue description. Candidate A passes by applying .distinct() consistently across all code paths (Field.get_choices, reverse_related, and ModelForm). Candidate B fails because it only modifies the ModelForm pathway but not the Field.get_choices() API, revealing an incomplete fix. The test correctly identifies that the duplicate-suppression invariant should hold for all entry points where limit_choices_to is applied, not just form rendering.

## Interpretation boundary

Agreement is measured against a human-produced reference review, not an objective ground truth. This note summarizes recorded evidence and does not expose private model chain-of-thought.
