# Round 4

This final round generates and validates candidates, then performs bidirectional differentiating-test search. By design it does not feed another round of gold-checked constraints back into generation.

Instances with evidence in this round: **29**.

## Terminal outcomes recorded in this round

- `identical`: 1
- `other_pair_generation_exhausted`: 3
- `round_limit`: 20
- `saturated`: 4
- `zero_passing_patches`: 1

## Contents

- `instances/`: one folder per instance, combining generation and differentiation evidence.
- `summary/`: round-wide state, counts, launch metadata, and logs.
- `input_suite/`: the accumulated oracle-augmented test suite used for candidate validation in this round.

## Instances

- [`django__django-11138`](instances/django__django-11138/)
- [`django__django-11400`](instances/django__django-11400/)
- [`django__django-11815`](instances/django__django-11815/)
- [`django__django-11848`](instances/django__django-11848/)
- [`django__django-13297`](instances/django__django-13297/)
- [`django__django-13837`](instances/django__django-13837/)
- [`django__django-14155`](instances/django__django-14155/)
- [`django__django-14170`](instances/django__django-14170/)
- [`django__django-15098`](instances/django__django-15098/)
- [`django__django-15127`](instances/django__django-15127/)
- [`django__django-15268`](instances/django__django-15268/)
- [`django__django-16454`](instances/django__django-16454/)
- [`django__django-16612`](instances/django__django-16612/)
- [`django__django-16661`](instances/django__django-16661/)
- [`matplotlib__matplotlib-14623`](instances/matplotlib__matplotlib-14623/)
- [`matplotlib__matplotlib-22719`](instances/matplotlib__matplotlib-22719/)
- [`matplotlib__matplotlib-22871`](instances/matplotlib__matplotlib-22871/)
- [`matplotlib__matplotlib-24026`](instances/matplotlib__matplotlib-24026/)
- [`matplotlib__matplotlib-24627`](instances/matplotlib__matplotlib-24627/)
- [`matplotlib__matplotlib-25479`](instances/matplotlib__matplotlib-25479/)
- [`pydata__xarray-4356`](instances/pydata__xarray-4356/)
- [`pydata__xarray-6744`](instances/pydata__xarray-6744/)
- [`scikit-learn__scikit-learn-14983`](instances/scikit-learn__scikit-learn-14983/)
- [`sphinx-doc__sphinx-9281`](instances/sphinx-doc__sphinx-9281/)
- [`sphinx-doc__sphinx-9461`](instances/sphinx-doc__sphinx-9461/)
- [`sympy__sympy-12096`](instances/sympy__sympy-12096/)
- [`sympy__sympy-13757`](instances/sympy__sympy-13757/)
- [`sympy__sympy-21612`](instances/sympy__sympy-21612/)
- [`sympy__sympy-23413`](instances/sympy__sympy-23413/)
