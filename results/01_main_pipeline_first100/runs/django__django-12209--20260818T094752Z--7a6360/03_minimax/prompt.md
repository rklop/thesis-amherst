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
Change in behaviour when saving a model instance with an explcit pk value if the pk field has a default
Description
	 
		(last modified by Reupen Shah)
	 
Consider the following model:
from uuid import uuid4
from django.db import models
class Sample(models.Model):
	id = models.UUIDField(primary_key=True, default=uuid4)
	name = models.CharField(blank=True, max_length=100)
In Django 2.2 and earlier, the following commands would result in an INSERT followed by an UPDATE:
s0 = Sample.objects.create()
s1 = Sample(pk=s0.pk, name='Test 1')
s1.save()
However, in Django 3.0, this results in two INSERTs (naturally the second one fails). The behaviour also changes if default=uuid4 is removed from the id field.
This seems related to https://code.djangoproject.com/ticket/29260.
The change in behaviour also has the side effect of changing the behaviour of the loaddata management command when the fixture contains explicit pk values and the objects already exist (e.g. when loading the fixture multiple times).
Perhaps the intention was to only change the behaviour if an explicit pk value was not set on the model instance being saved? (At least, that would be more backwards-compatible behaviour...)

</issue_statement>

<original_test_patch>
diff --git a/tests/serializers/models/data.py b/tests/serializers/models/data.py
--- a/tests/serializers/models/data.py
+++ b/tests/serializers/models/data.py
@@ -4,6 +4,8 @@
 NULL values, where allowed.
 The basic idea is to have a model for each Django data type.
 """
+import uuid
+
 from django.contrib.contenttypes.fields import (
     GenericForeignKey, GenericRelation,
 )
@@ -257,6 +259,10 @@ class UUIDData(models.Model):
     data = models.UUIDField(primary_key=True)
 
 
+class UUIDDefaultData(models.Model):
+    data = models.UUIDField(primary_key=True, default=uuid.uuid4)
+
+
 class FKToUUID(models.Model):
     data = models.ForeignKey(UUIDData, models.CASCADE)
 
diff --git a/tests/serializers/test_data.py b/tests/serializers/test_data.py
--- a/tests/serializers/test_data.py
+++ b/tests/serializers/test_data.py
@@ -26,7 +26,7 @@
     ModifyingSaveData, NullBooleanData, O2OData, PositiveBigIntegerData,
     PositiveIntegerData, PositiveIntegerPKData, PositiveSmallIntegerData,
     PositiveSmallIntegerPKData, SlugData, SlugPKData, SmallData, SmallPKData,
-    Tag, TextData, TimeData, UniqueAnchor, UUIDData,
+    Tag, TextData, TimeData, UniqueAnchor, UUIDData, UUIDDefaultData,
 )
 from .tests import register_tests
 
@@ -351,6 +351,7 @@ def inherited_compare(testcase, pk, klass, data):
     # (pk_obj, 790, XMLPKData, "<foo></foo>"),
     (pk_obj, 791, UUIDData, uuid_obj),
     (fk_obj, 792, FKToUUID, uuid_obj),
+    (pk_obj, 793, UUIDDefaultData, uuid_obj),
 
     (data_obj, 800, AutoNowDateTimeData, datetime.datetime(2006, 6, 16, 10, 42, 37)),
     (data_obj, 810, ModifyingSaveData, 42),

</original_test_patch>

<candidate_a_patch>
diff --git a/django/db/models/base.py b/django/db/models/base.py
index e3b14a41a0..7a42898bd1 100644
--- a/django/db/models/base.py
+++ b/django/db/models/base.py
@@ -415,6 +415,25 @@ class Model(metaclass=ModelBase):
         # Set up the storage for instance state
         self._state = ModelState()
 
+        # Track if pk was explicitly provided
+        pk_field = opts.pk
+        if pk_field:
+            pk_attname = pk_field.attname
+            # Check if pk was provided via args (if pk is first field, it would be args[0])
+            if args:
+                # Check if pk field is in the args
+                pk_field_index = None
+                for i, field in enumerate(opts.concrete_fields):
+                    if field == pk_field:
+                        pk_field_index = i
+                        break
+                if pk_field_index is not None and len(args) > pk_field_index:
+                    if args[pk_field_index] is not _DEFERRED:
+                        self._state.fields_cache['pk_set_explicitly'] = True
+            # Check if pk was provided via kwargs
+            if 'pk' in kwargs or pk_attname in kwargs:
+                self._state.fields_cache['pk_set_explicitly'] = True
+
         # There is a rather weird disparity here; if kwargs, it's set, then args
         # overrides it. It should be one or the other; don't duplicate the work
         # The reason for the kwargs check is that standard iterator passes in by
@@ -569,6 +588,8 @@ class Model(metaclass=ModelBase):
         return getattr(self, meta.pk.attname)
 
     def _set_pk_val(self, value):
+        if self._state.adding and value is not None:
+            self._state.fields_cache['pk_set_explicitly'] = True
         return setattr(self, self._meta.pk.attname, value)
 
     pk = property(_get_pk_val, _set_pk_val)
@@ -847,12 +868,14 @@ class Model(metaclass=ModelBase):
         if not pk_set and (force_update or update_fields):
             raise ValueError("Cannot force an update in save() with no primary key.")
         updated = False
-        # Skip an UPDATE when adding an instance and primary key has a default.
+        # Skip an UPDATE when adding an instance and primary key has a default,
+        # but only if the pk was not explicitly set.
         if (
             not force_insert and
             self._state.adding and
             self._meta.pk.default and
-            self._meta.pk.default is not NOT_PROVIDED
+            self._meta.pk.default is not NOT_PROVIDED and
+            not self._state.fields_cache.get('pk_set_explicitly')
         ):
             force_insert = True
         # If possible, try an UPDATE. If that doesn't update anything, do an INSERT.

</candidate_a_patch>

<candidate_b_patch>
diff --git a/django/db/models/base.py b/django/db/models/base.py
--- a/django/db/models/base.py
+++ b/django/db/models/base.py
@@ -849,6 +849,7 @@ def _save_table(self, raw=False, cls=None, force_insert=False,
         updated = False
         # Skip an UPDATE when adding an instance and primary key has a default.
         if (
+            not raw and
             not force_insert and
             self._state.adding and
             self._meta.pk.default and

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_raw_save_with_default_primary_key",
  "specification_gap": "A raw model save must bypass the primary-key-default INSERT optimization regardless of how the primary key was assigned. Its behavior must not depend on detecting an explicit PK during Model.__init__() or assignment through the generic pk property.",
  "input_description": "Create a NaturalPKWithDefault row, construct another instance so its UUID default is generated normally, assign the existing UUID through the model's public id field, change name, and call save_base(raw=True).",
  "expected_output": "The existing row is updated to name='updated', no duplicate-key error occurs, and the table still contains exactly one row.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "Candidate B disables the optimization unconditionally for raw saves. Candidate A relies on explicit-PK tracking, but direct assignment through the named primary-key field isn't tracked, so it attempts a duplicate INSERT. This covers an alternate public PK assignment entry point while testing the general raw-save invariant used by serialization.",
  "test_patch": "diff --git a/tests/serializers/test_natural.py b/tests/serializers/test_natural.py\n--- a/tests/serializers/test_natural.py\n+++ b/tests/serializers/test_natural.py\n@@ -9,7 +9,15 @@ from .tests import register_tests\n \n \n class NaturalKeySerializerTests(TestCase):\n-    pass\n+    def test_raw_save_with_default_primary_key(self):\n+        original = NaturalPKWithDefault.objects.create(name='original')\n+        obj = NaturalPKWithDefault(name='updated')\n+        obj.id = original.id\n+        obj.save_base(raw=True)\n+\n+        original.refresh_from_db()\n+        self.assertEqual(original.name, 'updated')\n+        self.assertEqual(NaturalPKWithDefault.objects.count(), 1)\n \n \n def natural_key_serializer_test(self, format):\n",
  "test_command": "python tests/runtests.py serializers.test_natural.NaturalKeySerializerTests.test_raw_save_with_default_primary_key"
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
Creating test database for alias 'default'...
E
======================================================================
ERROR: test_raw_save_with_default_primary_key (serializers.test_natural.NaturalKeySerializerTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/django/db/backends/utils.py", line 84, in _execute
    return self.cursor.execute(sql, params)
  File "/testbed/django/db/backends/sqlite3/base.py", line 401, in execute
    return Database.Cursor.execute(self, query, params)
sqlite3.IntegrityError: UNIQUE constraint failed: serializers_naturalpkwithdefault.id

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
  File "/testbed/tests/serializers/test_natural.py", line 17, in test_raw_save_with_default_primary_key
    obj.save_base(raw=True)
  File "/testbed/django/db/models/base.py", line 806, in save_base
    force_update, using, update_fields,
  File "/testbed/django/db/models/base.py", line 910, in _save_table
    results = self._do_insert(cls._base_manager, using, fields, returning_fields, raw)
  File "/testbed/django/db/models/base.py", line 949, in _do_insert
    using=using, raw=raw,
  File "/testbed/django/db/models/manager.py", line 82, in manager_method
    return getattr(self.get_queryset(), name)(*args, **kwargs)
  File "/testbed/django/db/models/query.py", line 1226, in _insert
    return query.get_compiler(using=using).execute_sql(returning_fields)
  File "/testbed/django/db/models/sql/compiler.py", line 1374, in execute_sql
    cursor.execute(sql, params)
  File "/testbed/django/db/backends/utils.py", line 66, in execute
    return self._execute_with_wrappers(sql, params, many=False, executor=self._execute)
  File "/testbed/django/db/backends/utils.py", line 75, in _execute_with_wrappers
    return executor(sql, params, many, context)
  File "/testbed/django/db/backends/utils.py", line 84, in _execute
    return self.cursor.execute(sql, params)
  File "/testbed/django/db/utils.py", line 90, in __exit__
    raise dj_exc_value.with_traceback(traceback) from exc_value
  File "/testbed/django/db/backends/utils.py", line 84, in _execute
    return self.cursor.execute(sql, params)
  File "/testbed/django/db/backends/sqlite3/base.py", line 401, in execute
    return Database.Cursor.execute(self, query, params)
django.db.utils.IntegrityError: UNIQUE constraint failed: serializers_naturalpkwithdefault.id

----------------------------------------------------------------------
Ran 1 test in 0.002s

FAILED (errors=1)
Destroying test database for alias 'default'...
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
Creating test database for alias 'default'...
.Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).

----------------------------------------------------------------------
Ran 1 test in 0.001s

OK
Destroying test database for alias 'default'...
[pipeline] test_exit_code=0

```
</validated_execution>
