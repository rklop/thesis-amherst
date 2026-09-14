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
pk setup for MTI to parent get confused by multiple OneToOne references.
Description
	
class Document(models.Model):
	pass
class Picking(Document):
	document_ptr = models.OneToOneField(Document, on_delete=models.CASCADE, parent_link=True, related_name='+')
	origin = models.OneToOneField(Document, related_name='picking', on_delete=models.PROTECT)
produces django.core.exceptions.ImproperlyConfigured: Add parent_link=True to appname.Picking.origin.
class Picking(Document):
	origin = models.OneToOneField(Document, related_name='picking', on_delete=models.PROTECT)
	document_ptr = models.OneToOneField(Document, on_delete=models.CASCADE, parent_link=True, related_name='+')
Works
First issue is that order seems to matter?
Even if ordering is required "by design"(It shouldn't be we have explicit parent_link marker) shouldn't it look from top to bottom like it does with managers and other things?

</issue_statement>

<original_test_patch>
diff --git a/tests/invalid_models_tests/test_models.py b/tests/invalid_models_tests/test_models.py
--- a/tests/invalid_models_tests/test_models.py
+++ b/tests/invalid_models_tests/test_models.py
@@ -3,7 +3,6 @@
 from django.conf import settings
 from django.core.checks import Error, Warning
 from django.core.checks.model_checks import _check_lazy_references
-from django.core.exceptions import ImproperlyConfigured
 from django.db import connection, connections, models
 from django.db.models.functions import Lower
 from django.db.models.signals import post_init
@@ -1006,14 +1005,24 @@ class ShippingMethodPrice(models.Model):
 
         self.assertEqual(ShippingMethod.check(), [])
 
-    def test_missing_parent_link(self):
-        msg = 'Add parent_link=True to invalid_models_tests.ParkingLot.parent.'
-        with self.assertRaisesMessage(ImproperlyConfigured, msg):
-            class Place(models.Model):
-                pass
+    def test_onetoone_with_parent_model(self):
+        class Place(models.Model):
+            pass
+
+        class ParkingLot(Place):
+            other_place = models.OneToOneField(Place, models.CASCADE, related_name='other_parking')
+
+        self.assertEqual(ParkingLot.check(), [])
+
+    def test_onetoone_with_explicit_parent_link_parent_model(self):
+        class Place(models.Model):
+            pass
+
+        class ParkingLot(Place):
+            place = models.OneToOneField(Place, models.CASCADE, parent_link=True, primary_key=True)
+            other_place = models.OneToOneField(Place, models.CASCADE, related_name='other_parking')
 
-            class ParkingLot(Place):
-                parent = models.OneToOneField(Place, models.CASCADE)
+        self.assertEqual(ParkingLot.check(), [])
 
     def test_m2m_table_name_clash(self):
         class Foo(models.Model):
diff --git a/tests/invalid_models_tests/test_relative_fields.py b/tests/invalid_models_tests/test_relative_fields.py
--- a/tests/invalid_models_tests/test_relative_fields.py
+++ b/tests/invalid_models_tests/test_relative_fields.py
@@ -1291,6 +1291,33 @@ class Model(models.Model):
             ),
         ])
 
+    def test_clash_parent_link(self):
+        class Parent(models.Model):
+            pass
+
+        class Child(Parent):
+            other_parent = models.OneToOneField(Parent, models.CASCADE)
+
+        errors = [
+            ('fields.E304', 'accessor', 'parent_ptr', 'other_parent'),
+            ('fields.E305', 'query name', 'parent_ptr', 'other_parent'),
+            ('fields.E304', 'accessor', 'other_parent', 'parent_ptr'),
+            ('fields.E305', 'query name', 'other_parent', 'parent_ptr'),
+        ]
+        self.assertEqual(Child.check(), [
+            Error(
+                "Reverse %s for 'Child.%s' clashes with reverse %s for "
+                "'Child.%s'." % (attr, field_name, attr, clash_name),
+                hint=(
+                    "Add or change a related_name argument to the definition "
+                    "for 'Child.%s' or 'Child.%s'." % (field_name, clash_name)
+                ),
+                obj=Child._meta.get_field(field_name),
+                id=error_id,
+            )
+            for error_id, attr, field_name, clash_name in errors
+        ])
+
 
 @isolate_apps('invalid_models_tests')
 class M2mThroughFieldsTests(SimpleTestCase):
diff --git a/tests/migrations/test_state.py b/tests/migrations/test_state.py
--- a/tests/migrations/test_state.py
+++ b/tests/migrations/test_state.py
@@ -345,6 +345,7 @@ def test_render(self):
                     'migrations.Tag',
                     models.CASCADE,
                     auto_created=True,
+                    parent_link=True,
                     primary_key=True,
                     to_field='id',
                     serialize=False,

</original_test_patch>

<candidate_a_patch>
diff --git a/django/db/models/base.py b/django/db/models/base.py
index 8ea6c05ef9..24453e218a 100644
--- a/django/db/models/base.py
+++ b/django/db/models/base.py
@@ -202,7 +202,7 @@ class ModelBase(type):
                 continue
             # Locate OneToOneField instances.
             for field in base._meta.local_fields:
-                if isinstance(field, OneToOneField):
+                if isinstance(field, OneToOneField) and field.remote_field.parent_link:
                     related = resolve_relation(new_class, field.remote_field.model)
                     parent_links[make_model_tuple(related)] = field
 

</candidate_a_patch>

<candidate_b_patch>
diff --git a/django/db/models/base.py b/django/db/models/base.py
--- a/django/db/models/base.py
+++ b/django/db/models/base.py
@@ -202,7 +202,7 @@ def __new__(cls, name, bases, attrs, **kwargs):
                 continue
             # Locate OneToOneField instances.
             for field in base._meta.local_fields:
-                if isinstance(field, OneToOneField):
+                if isinstance(field, OneToOneField) and field.remote_field.parent_link:
                     related = resolve_relation(new_class, field.remote_field.model)
                     parent_links[make_model_tuple(related)] = field
 
diff --git a/django/db/models/options.py b/django/db/models/options.py
--- a/django/db/models/options.py
+++ b/django/db/models/options.py
@@ -5,7 +5,7 @@
 
 from django.apps import apps
 from django.conf import settings
-from django.core.exceptions import FieldDoesNotExist, ImproperlyConfigured
+from django.core.exceptions import FieldDoesNotExist
 from django.db import connections
 from django.db.models import Manager
 from django.db.models.fields import AutoField
@@ -251,10 +251,6 @@ def _prepare(self, model):
                     field = already_created[0]
                 field.primary_key = True
                 self.setup_pk(field)
-                if not field.remote_field.parent_link:
-                    raise ImproperlyConfigured(
-                        'Add parent_link=True to %s.' % field,
-                    )
             else:
                 auto = AutoField(verbose_name='ID', primary_key=True, auto_created=True)
                 model.add_to_class('id', auto)

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_abstract_onetoone_named_as_parent_pointer",
  "specification_gap": "An unrelated OneToOneField inherited from an abstract base must not be silently repurposed as the concrete model's multi-table-inheritance parent link merely because its name matches the generated parent pointer. Such an ambiguous model definition must be rejected unless the field explicitly declares parent_link=True.",
  "input_description": "Define an abstract model containing place_ptr = OneToOneField(Place) without parent_link=True, then define ParkingLot(AbstractParkingLot, Place).",
  "expected_output": "Defining ParkingLot raises ImproperlyConfigured. It must not construct a model whose unrelated OneToOneField is implicitly promoted to the inheritance link.",
  "expected_passing_candidate": "candidate_a",
  "why_interesting": "This exercises abstract-field inheritance, an alternate public entry path missed by the ordinary local-field tests. Candidate A retains the validation that rejects the ambiguous field, while candidate B removes it and silently constructs an inconsistent model.",
  "test_patch": "diff --git a/tests/invalid_models_tests/test_abstract_parent_link.py b/tests/invalid_models_tests/test_abstract_parent_link.py\nnew file mode 100644\n--- /dev/null\n+++ b/tests/invalid_models_tests/test_abstract_parent_link.py\n@@ -0,0 +1,21 @@\n+from django.core.exceptions import ImproperlyConfigured\n+from django.db import models\n+from django.test import SimpleTestCase\n+from django.test.utils import isolate_apps\n+\n+\n+@isolate_apps('invalid_models_tests')\n+class ParentLinkTests(SimpleTestCase):\n+    def test_abstract_onetoone_named_as_parent_pointer(self):\n+        class Place(models.Model):\n+            pass\n+\n+        class AbstractParkingLot(models.Model):\n+            place_ptr = models.OneToOneField(Place, models.CASCADE)\n+\n+            class Meta:\n+                abstract = True\n+\n+        with self.assertRaises(ImproperlyConfigured):\n+            class ParkingLot(AbstractParkingLot, Place):\n+                pass\n",
  "test_command": "PYTHONPATH=. python tests/runtests.py invalid_models_tests.test_abstract_parent_link.ParentLinkTests.test_abstract_onetoone_named_as_parent_pointer"
}
</generated_test_proposal>

<validated_execution>
## candidate_a
returncode=0 passed=True test_ran=True
log=02_execution/attempt_01/candidate_a.log
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

## candidate_b
returncode=1 passed=False test_ran=True
log=02_execution/attempt_01/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
F
======================================================================
FAIL: test_abstract_onetoone_named_as_parent_pointer (invalid_models_tests.test_abstract_parent_link.ParentLinkTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/invalid_models_tests/test_abstract_parent_link.py", line 20, in test_abstract_onetoone_named_as_parent_pointer
    class ParkingLot(AbstractParkingLot, Place):
AssertionError: ImproperlyConfigured not raised

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (failures=1)
[pipeline] test_exit_code=1

```
</validated_execution>
