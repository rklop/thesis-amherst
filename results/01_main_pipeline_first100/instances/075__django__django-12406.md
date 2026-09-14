# 075 — django__django-12406

## Comparison

- Manual reference: **Not different**
- Pipeline-derived bucket: **Very different**
- Exact agreement: **no**
- Ordinal distance: **2**

## Why each side classified it this way

**Manual review:** The timing differs, but the same RadioSelect/blank rule is implemented.

**Pipeline mapping:** A real pass/fail separation was found and MiniMax rated it high-signal.

## Differentiating test

- Test: `test_choices_radio_blank_default_widget`
- Candidate that passed: **candidate_b**
- Specification gap tested: RadioSelect behavior should depend on the field's effective widget, including a widget inherited from a ModelChoiceField subclass, not only an explicit widget= argument.
- Input: Define a ModelChoiceField subclass with widget = forms.RadioSelect, then instantiate it with blank=False over three Category objects without passing widget explicitly.
- Expected behavior: The public choices iterator contains exactly the three database-backed categories and no empty ('', '---------') choice.

## MiniMax assessment

- Rating: **high_signal**
- Confidence: **0.9**
- Summary: The test correctly identifies a real specification gap: candidate B properly examines the resolved widget instance (including inherited/default widgets), while candidate A only checks the constructor argument and fails to suppress the blank option when the widget is inherited from a field subclass. The test exercises the intended public behavior (RadioSelect should not show blank option when blank=False) using a common pattern (ModelChoiceField subclass with default widget). Candidate B passes because it checks `self.widget` (the actual widget instance) while candidate A checks the `widget` argument before widget resolution. The winner (candidate B) appears more specification-conformant as it handles the standard field-subclass entry point for default widget selection.

## Severe-disagreement adjudication

- Category: **Strong new behavioral evidence**
- Assessment: A ModelChoiceField subclass with inherited RadioSelect shows that only gold applies the blank-choice rule to the effective widget.

## Interpretation boundary

Agreement is measured against a human-produced reference review, not an objective ground truth. This note summarizes recorded evidence and does not expose private model chain-of-thought.
