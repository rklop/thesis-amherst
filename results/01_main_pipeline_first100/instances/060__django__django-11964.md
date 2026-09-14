# 060 — django__django-11964

## Comparison

- Manual reference: **Somewhat different**
- Pipeline-derived bucket: **Very different**
- Exact agreement: **no**
- Ordinal distance: **1**

## Why each side classified it this way

**Manual review:** Built-in choice types behave the same, but gold also covers other subclasses of the shared base.

**Pipeline mapping:** A real pass/fail separation was found and MiniMax rated it high-signal.

## Differentiating test

- Test: `custom_date_choices_str_uses_value`
- Candidate that passed: **candidate_b**
- Specification gap tested: The issue’s conversion invariant is unspecified for Django’s documented custom `models.Choices` subclasses. These are a public way to use concrete field types other than `str` and `int`, so a custom choice should stringify like its underlying concrete value too.
- Input: Call `str()` on `MoonLandings.APOLLO_11`, an existing `datetime.date`/`models.Choices` member whose underlying value is 1969-07-20.
- Expected behavior: The result is the externally usable date string `1969-07-20`, rather than the enum identity `MoonLandings.APOLLO_11`.

## MiniMax assessment

- Rating: **high_signal**
- Confidence: **0.9**
- Summary: The test correctly differentiates between candidates by checking whether the fix generalizes to custom Choices subclasses with non-standard concrete types (datetime.date). Candidate B passes because it adds __str__ to the base Choices class, while Candidate A fails because it only adds it to IntegerChoices and TextChoices. The test's oracle (str(enum) should equal str(value)) is directly derived from the issue's complaint that str() returns the enum identity instead of the value. While the test doesn't directly exercise model field retrieval, testing the enum's __str__ is sufficient since that's where the fix is applied and it determines the behavior when accessing model fields.

## Interpretation boundary

Agreement is measured against a human-produced reference review, not an objective ground truth. This note summarizes recorded evidence and does not expose private model chain-of-thought.
