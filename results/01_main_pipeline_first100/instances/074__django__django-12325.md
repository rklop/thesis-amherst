# 074 — django__django-12325

## Comparison

- Manual reference: **Very different**
- Pipeline-derived bucket: **Very different**
- Exact agreement: **yes**
- Ordinal distance: **0**

## Why each side classified it this way

**Manual review:** The candidate leaves a second rule that can still reject configurations gold intentionally permits.

**Pipeline mapping:** A real pass/fail separation was found and MiniMax rated it high-signal.

## Differentiating test

- Test: `test_abstract_onetoone_named_as_parent_pointer`
- Candidate that passed: **candidate_a**
- Specification gap tested: An unrelated OneToOneField inherited from an abstract base must not be silently repurposed as the concrete model's multi-table-inheritance parent link merely because its name matches the generated parent pointer. Such an ambiguous model definition must be rejected unless the field explicitly declares parent_link=True.
- Input: Define an abstract model containing place_ptr = OneToOneField(Place) without parent_link=True, then define ParkingLot(AbstractParkingLot, Place).
- Expected behavior: Defining ParkingLot raises ImproperlyConfigured. It must not construct a model whose unrelated OneToOneField is implicitly promoted to the inheritance link.

## MiniMax assessment

- Rating: **high_signal**
- Confidence: **0.9**
- Summary: The test exercises a valid public behavior: Django should raise ImproperlyConfigured when an abstract model's OneToOneField could be confused for an MTI parent link, requiring explicit parent_link=True. Candidate_a passes because it preserves this validation; candidate_b fails because it removes the check entirely, allowing silent misconfiguration. The test targets the specification gap in abstract model inheritance that wasn't covered by the original issue or patch.

## Interpretation boundary

Agreement is measured against a human-produced reference review, not an objective ground truth. This note summarizes recorded evidence and does not expose private model chain-of-thought.
