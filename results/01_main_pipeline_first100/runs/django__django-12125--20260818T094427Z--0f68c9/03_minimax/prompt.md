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
makemigrations produces incorrect path for inner classes
Description
	
When you define a subclass from django.db.models.Field as an inner class of some other class, and use this field inside a django.db.models.Model class, then when you run manage.py makemigrations, a migrations file is created which refers to the inner class as if it were a top-level class of the module it is in.
To reproduce, create the following as your model:
class Outer(object):
	class Inner(models.CharField):
		pass
class A(models.Model):
	field = Outer.Inner(max_length=20)
After running manage.py makemigrations, the generated migrations file contains the following:
migrations.CreateModel(
	name='A',
	fields=[
		('id', models.AutoField(auto_created=True, primary_key=True, serialize=False, verbose_name='ID')),
		('field', test1.models.Inner(max_length=20)),
	],
),
Note the test1.models.Inner, which should have been test1.models.Outer.Inner.
The real life case involved an EnumField from django-enumfields, defined as an inner class of a Django Model class, similar to this:
import enum
from enumfields import Enum, EnumField
class Thing(models.Model):
	@enum.unique
	class State(Enum):
		on = 'on'
		off = 'off'
	state = EnumField(enum=State)
This results in the following migrations code:
migrations.CreateModel(
	name='Thing',
	fields=[
		('id', models.AutoField(auto_created=True, primary_key=True, serialize=False, verbose_name='ID')),
		('state', enumfields.fields.EnumField(enum=test1.models.State, max_length=10)),
	],
),
This refers to test1.models.State, instead of to test1.models.Thing.State.

</issue_statement>

<original_test_patch>
diff --git a/tests/migrations/test_writer.py b/tests/migrations/test_writer.py
--- a/tests/migrations/test_writer.py
+++ b/tests/migrations/test_writer.py
@@ -26,6 +26,11 @@
 from .models import FoodManager, FoodQuerySet
 
 
+class DeconstructibleInstances:
+    def deconstruct(self):
+        return ('DeconstructibleInstances', [], {})
+
+
 class Money(decimal.Decimal):
     def deconstruct(self):
         return (
@@ -188,6 +193,10 @@ class NestedEnum(enum.IntEnum):
         A = 1
         B = 2
 
+    class NestedChoices(models.TextChoices):
+        X = 'X', 'X value'
+        Y = 'Y', 'Y value'
+
     def safe_exec(self, string, value=None):
         d = {}
         try:
@@ -383,6 +392,18 @@ class DateChoices(datetime.date, models.Choices):
             "default=datetime.date(1969, 11, 19))"
         )
 
+    def test_serialize_nested_class(self):
+        for nested_cls in [self.NestedEnum, self.NestedChoices]:
+            cls_name = nested_cls.__name__
+            with self.subTest(cls_name):
+                self.assertSerializedResultEqual(
+                    nested_cls,
+                    (
+                        "migrations.test_writer.WriterTests.%s" % cls_name,
+                        {'import migrations.test_writer'},
+                    ),
+                )
+
     def test_serialize_uuid(self):
         self.assertSerializedEqual(uuid.uuid1())
         self.assertSerializedEqual(uuid.uuid4())
@@ -726,10 +747,6 @@ def test_deconstruct_class_arguments(self):
         # Yes, it doesn't make sense to use a class as a default for a
         # CharField. It does make sense for custom fields though, for example
         # an enumfield that takes the enum class as an argument.
-        class DeconstructibleInstances:
-            def deconstruct(self):
-                return ('DeconstructibleInstances', [], {})
-
         string = MigrationWriter.serialize(models.CharField(default=DeconstructibleInstances))[0]
         self.assertEqual(string, "models.CharField(default=migrations.test_writer.DeconstructibleInstances)")
 

</original_test_patch>

<candidate_a_patch>
diff --git a/django/db/migrations/serializer.py b/django/db/migrations/serializer.py
index 7d226955d5..00e7bea1b7 100644
--- a/django/db/migrations/serializer.py
+++ b/django/db/migrations/serializer.py
@@ -267,9 +267,9 @@ class TypeSerializer(BaseSerializer):
         if hasattr(self.value, "__module__"):
             module = self.value.__module__
             if module == builtins.__name__:
-                return self.value.__name__, set()
+                return self.value.__qualname__, set()
             else:
-                return "%s.%s" % (module, self.value.__name__), {"import %s" % module}
+                return "%s.%s" % (module, self.value.__qualname__), {"import %s" % module}
 
 
 class UUIDSerializer(BaseSerializer):

</candidate_a_patch>

<candidate_b_patch>
diff --git a/django/db/migrations/serializer.py b/django/db/migrations/serializer.py
--- a/django/db/migrations/serializer.py
+++ b/django/db/migrations/serializer.py
@@ -269,7 +269,7 @@ def serialize(self):
             if module == builtins.__name__:
                 return self.value.__name__, set()
             else:
-                return "%s.%s" % (module, self.value.__name__), {"import %s" % module}
+                return "%s.%s" % (module, self.value.__qualname__), {"import %s" % module}
 
 
 class UUIDSerializer(BaseSerializer):

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_serialize_nested_builtin_type",
  "specification_gap": "The specification does not state whether qualified-name serialization applies when a nested class reports the public `builtins` module. Candidate A applies `__qualname__` uniformly, while candidate B special-cases builtins and emits only `__name__`, producing an unresolvable reference for a nested builtins type.",
  "input_description": "Create `MigrationTypes.Nested`, expose `MigrationTypes` through the builtins namespace, and set the nested class's public module and qualified name to `builtins` and `MigrationTypes.Nested`. Serialize the nested class through `MigrationWriter` and execute the generated expression.",
  "expected_output": "The generated expression must evaluate to the identical `MigrationTypes.Nested` class. Candidate A emits `MigrationTypes.Nested` with no imports; candidate B emits `Nested`, which cannot be resolved.",
  "expected_passing_candidate": "candidate_a",
  "why_interesting": "This is a boundary case for the existing builtins public entry point and checks the general round-trip invariant rather than an internal serializer choice. It reveals whether support for nested classes is uniform across modules or silently excluded for builtins.",
  "test_patch": "diff --git a/tests/migrations/test_writer.py b/tests/migrations/test_writer.py\n--- a/tests/migrations/test_writer.py\n+++ b/tests/migrations/test_writer.py\n@@ -1,3 +1,4 @@\n+import builtins\n import datetime\n import decimal\n import enum\n@@ -266,8 +267,18 @@ class WriterTests(SimpleTestCase):\n     def test_serialize_builtin_types(self):\n         self.assertSerializedEqual([list, tuple, dict, set, frozenset])\n         self.assertSerializedResultEqual(\n             [list, tuple, dict, set, frozenset],\n             (\"[list, tuple, dict, set, frozenset]\", set())\n         )\n \n+    def test_serialize_nested_builtin_type(self):\n+        class MigrationTypes:\n+            class Nested:\n+                pass\n+\n+        MigrationTypes.Nested.__module__ = builtins.__name__\n+        MigrationTypes.Nested.__qualname__ = 'MigrationTypes.Nested'\n+        with mock.patch.object(builtins, 'MigrationTypes', MigrationTypes, create=True):\n+            self.assertIs(self.serialize_round_trip(MigrationTypes.Nested), MigrationTypes.Nested)\n+\n     def test_serialize_lazy_objects(self):\n         pattern = re.compile(r'^foo$')\n",
  "test_command": "PYTHONPATH=. python tests/runtests.py migrations.test_writer.WriterTests.test_serialize_nested_builtin_type"
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
.
----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
[pipeline] test_exit_code=0

```

## candidate_b
returncode=1 passed=False test_ran=True
log=02_execution/attempt_01/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
F
======================================================================
FAIL: test_serialize_nested_builtin_type (migrations.test_writer.WriterTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/migrations/test_writer.py", line 195, in safe_exec
    exec(string, globals(), d)
  File "<string>", line 2, in <module>
NameError: name 'Nested' is not defined

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/testbed/tests/migrations/test_writer.py", line 282, in test_serialize_nested_builtin_type
    self.assertIs(self.serialize_round_trip(MigrationTypes.Nested), MigrationTypes.Nested)
  File "/testbed/tests/migrations/test_writer.py", line 205, in serialize_round_trip
    return self.safe_exec("%s\ntest_value_result = %s" % ("\n".join(imports), string), value)['test_value_result']
  File "/testbed/tests/migrations/test_writer.py", line 198, in safe_exec
    self.fail("Could not exec %r (from value %r): %s" % (string.strip(), value, e))
AssertionError: Could not exec 'test_value_result = Nested' (from value <class 'Nested'>): name 'Nested' is not defined

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (failures=1)
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
[pipeline] test_exit_code=1

```
</validated_execution>
