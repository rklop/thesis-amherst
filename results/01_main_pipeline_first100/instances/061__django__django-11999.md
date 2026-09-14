# 061 — django__django-11999

## Comparison

- Manual reference: **Very different**
- Pipeline-derived bucket: **Very different**
- Exact agreement: **yes**
- Ordinal distance: **0**

## Why each side classified it this way

**Manual review:** Inherited overrides survive under gold but can still be overwritten by the candidate.

**Pipeline mapping:** A real pass/fail separation was found and MiniMax rated it high-signal.

## Differentiating test

- Test: `abstract_model_inherited_get_FIELD_display_override`
- Candidate that passed: **candidate_b**
- Specification gap tested: Override behavior is underspecified for inherited methods. When a choices field is copied from an abstract model to a concrete subclass, Django must not shadow the abstract model’s custom get_<field>_display() method.
- Input: Define an abstract model with an IntegerField whose choices map 1 to "foo", and override get_foo_bar_display() to return "something". Instantiate a concrete subclass with foo_bar=1 and call the inherited method.
- Expected behavior: get_foo_bar_display() returns "something", not the automatically generated choice label "foo".

## MiniMax assessment

- Rating: **high_signal**
- Confidence: **0.9**
- Summary: The generated test exposes a meaningful semantic disagreement between candidates regarding Django's abstract model inheritance. Candidate A uses cls.__dict__ which only checks the concrete class and shadows inherited overrides, while candidate B uses hasattr which respects the inheritance chain. The test uses public Django API (abstract models) and has a clear, defensible oracle based on the issue's core requirement that users should be able to override get_FOO_display().

## Interpretation boundary

Agreement is measured against a human-produced reference review, not an objective ground truth. This note summarizes recorded evidence and does not expose private model chain-of-thought.
