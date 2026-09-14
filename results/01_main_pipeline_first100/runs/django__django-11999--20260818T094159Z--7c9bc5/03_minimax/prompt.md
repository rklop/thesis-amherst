Judge the quality of a validated differentiating test for a
SWE-Bench task. Return exactly one JSON object and no Markdown.

Allowed ratings:

- `high_signal`: the test exercises intended public behavior, has a defensible
  oracle, reveals a meaningful missing specification, and is reasonably
  general/minimal rather than tailored to implementation internals.
- `low_signal`: the split is real but primarily reflects brittle internals,
  incidental formatting, undefined behavior, a contrived exploit, unrelated
  regressions, or an oracle not supported by the issue/API contract.
- `ambiguous`: the available evidence does not justify either quality judgment
  without a human deciding an underspecified semantic question.

Do not rate a test high merely because one candidate passes and one fails.
Assess whether the expected output follows from the issue and established API
semantics. Explicitly identify which candidate passed and whether that winner
appears more specification-conformant.

Required JSON keys: `rating`, `confidence`, `summary`, `specification_signal`,
`oracle_quality`, `concerns`, `human_review_questions`. `rating` must be one of
the three allowed values. `confidence` must be a number from 0 to 1. The last
three keys may contain arrays of strings.

<issue_statement>
Cannot override get_FOO_display() in Django 2.2+.
Description
	
I cannot override the get_FIELD_display function on models since version 2.2. It works in version 2.1.
Example:
class FooBar(models.Model):
	foo_bar = models.CharField(_("foo"), choices=[(1, 'foo'), (2, 'bar')])
	def __str__(self):
		return self.get_foo_bar_display() # This returns 'foo' or 'bar' in 2.2, but 'something' in 2.1
	def get_foo_bar_display(self):
		return "something"
What I expect is that I should be able to override this function.

</issue_statement>

<original_test_patch>
diff --git a/tests/model_fields/tests.py b/tests/model_fields/tests.py
--- a/tests/model_fields/tests.py
+++ b/tests/model_fields/tests.py
@@ -168,6 +168,16 @@ def test_get_FIELD_display_translated(self):
         self.assertIsInstance(val, str)
         self.assertEqual(val, 'translated')
 
+    def test_overriding_FIELD_display(self):
+        class FooBar(models.Model):
+            foo_bar = models.IntegerField(choices=[(1, 'foo'), (2, 'bar')])
+
+            def get_foo_bar_display(self):
+                return 'something'
+
+        f = FooBar(foo_bar=1)
+        self.assertEqual(f.get_foo_bar_display(), 'something')
+
     def test_iterator_choices(self):
         """
         get_choices() works with Iterators.

</original_test_patch>

<candidate_a_patch>
diff --git a/django/db/models/fields/__init__.py b/django/db/models/fields/__init__.py
index ee45bb941e..7879115ee4 100644
--- a/django/db/models/fields/__init__.py
+++ b/django/db/models/fields/__init__.py
@@ -763,8 +763,9 @@ class Field(RegisterLookupMixin):
             if not getattr(cls, self.attname, None):
                 setattr(cls, self.attname, self.descriptor_class(self))
         if self.choices is not None:
-            setattr(cls, 'get_%s_display' % self.name,
-                    partialmethod(cls._get_FIELD_display, field=self))
+            if 'get_%s_display' % self.name not in cls.__dict__:
+                setattr(cls, 'get_%s_display' % self.name,
+                        partialmethod(cls._get_FIELD_display, field=self))
 
     def get_filter_kwargs_for_object(self, obj):
         """

</candidate_a_patch>

<candidate_b_patch>
diff --git a/django/db/models/fields/__init__.py b/django/db/models/fields/__init__.py
--- a/django/db/models/fields/__init__.py
+++ b/django/db/models/fields/__init__.py
@@ -763,8 +763,12 @@ def contribute_to_class(self, cls, name, private_only=False):
             if not getattr(cls, self.attname, None):
                 setattr(cls, self.attname, self.descriptor_class(self))
         if self.choices is not None:
-            setattr(cls, 'get_%s_display' % self.name,
-                    partialmethod(cls._get_FIELD_display, field=self))
+            if not hasattr(cls, 'get_%s_display' % self.name):
+                setattr(
+                    cls,
+                    'get_%s_display' % self.name,
+                    partialmethod(cls._get_FIELD_display, field=self),
+                )
 
     def get_filter_kwargs_for_object(self, obj):
         """

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "abstract_model_inherited_get_FIELD_display_override",
  "specification_gap": "Override behavior is underspecified for inherited methods. When a choices field is copied from an abstract model to a concrete subclass, Django must not shadow the abstract model\u2019s custom get_<field>_display() method.",
  "input_description": "Define an abstract model with an IntegerField whose choices map 1 to \"foo\", and override get_foo_bar_display() to return \"something\". Instantiate a concrete subclass with foo_bar=1 and call the inherited method.",
  "expected_output": "get_foo_bar_display() returns \"something\", not the automatically generated choice label \"foo\".",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "The benchmark covers only a same-class override. Abstract model inheritance is a public Django mechanism and exposes the exact semantic disagreement: candidate A checks only the concrete class dictionary and replaces the inherited override, while candidate B respects normal inherited attribute lookup.",
  "test_patch": "diff --git a/tests/model_fields/tests.py b/tests/model_fields/tests.py\n--- a/tests/model_fields/tests.py\n+++ b/tests/model_fields/tests.py\n@@ -168,6 +168,21 @@ class GetFieldDisplayTests(SimpleTestCase):\n         self.assertIsInstance(val, str)\n         self.assertEqual(val, 'translated')\n \n+    def test_overriding_FIELD_display_inherited_from_abstract_model(self):\n+        class AbstractFooBar(models.Model):\n+            foo_bar = models.IntegerField(choices=[(1, 'foo'), (2, 'bar')])\n+\n+            def get_foo_bar_display(self):\n+                return 'something'\n+\n+            class Meta:\n+                abstract = True\n+\n+        class FooBar(AbstractFooBar):\n+            pass\n+\n+        self.assertEqual(FooBar(foo_bar=1).get_foo_bar_display(), 'something')\n+\n     def test_iterator_choices(self):\n         \"\"\"\n         get_choices() works with Iterators.\n",
  "test_command": "cd /testbed && python tests/runtests.py model_fields.tests.GetFieldDisplayTests.test_overriding_FIELD_display_inherited_from_abstract_model"
}
</generated_test_proposal>

<validated_execution>
## candidate_a
returncode=1 passed=False test_ran=True
log=02_execution/attempt_01/candidate_a.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
F
======================================================================
FAIL: test_overriding_FIELD_display_inherited_from_abstract_model (model_fields.tests.GetFieldDisplayTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/model_fields/tests.py", line 184, in test_overriding_FIELD_display_inherited_from_abstract_model
    self.assertEqual(FooBar(foo_bar=1).get_foo_bar_display(), 'something')
AssertionError: 'foo' != 'something'
- foo
+ something


----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (failures=1)
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
[pipeline] test_exit_code=1

```

## candidate_b
returncode=0 passed=True test_ran=True
log=02_execution/attempt_01/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
.Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).

----------------------------------------------------------------------
Ran 1 test in 0.001s

OK
[pipeline] test_exit_code=0

```
</validated_execution>
