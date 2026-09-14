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
Resetting primary key for a child model doesn't work.
Description
	
In the attached example code setting the primary key to None does not work (so that the existing object is overwritten on save()).
The most important code fragments of the bug example:
from django.db import models
class Item(models.Model):
	# uid = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
	uid = models.AutoField(primary_key=True, editable=False)
	f = models.BooleanField(default=False)
	def reset(self):
		self.uid = None
		self.f = False
class Derived(Item):
	pass
class SaveTestCase(TestCase):
	def setUp(self):
		self.derived = Derived.objects.create(f=True) # create the first object
		item = Item.objects.get(pk=self.derived.pk)
		obj1 = item.derived
		obj1.reset()
		obj1.save() # the first object is overwritten
	def test_f_true(self):
		obj = Item.objects.get(pk=self.derived.pk)
		self.assertTrue(obj.f)
Django 2.1.2

</issue_statement>

<original_test_patch>
diff --git a/tests/model_inheritance_regress/tests.py b/tests/model_inheritance_regress/tests.py
--- a/tests/model_inheritance_regress/tests.py
+++ b/tests/model_inheritance_regress/tests.py
@@ -10,10 +10,11 @@
 
 from .models import (
     ArticleWithAuthor, BachelorParty, BirthdayParty, BusStation, Child,
-    DerivedM, InternalCertificationAudit, ItalianRestaurant, M2MChild,
-    MessyBachelorParty, ParkingLot, ParkingLot3, ParkingLot4A, ParkingLot4B,
-    Person, Place, Profile, QualityControl, Restaurant, SelfRefChild,
-    SelfRefParent, Senator, Supplier, TrainStation, User, Wholesaler,
+    Congressman, DerivedM, InternalCertificationAudit, ItalianRestaurant,
+    M2MChild, MessyBachelorParty, ParkingLot, ParkingLot3, ParkingLot4A,
+    ParkingLot4B, Person, Place, Politician, Profile, QualityControl,
+    Restaurant, SelfRefChild, SelfRefParent, Senator, Supplier, TrainStation,
+    User, Wholesaler,
 )
 
 
@@ -558,3 +559,31 @@ def test_id_field_update_on_ancestor_change(self):
         italian_restaurant.restaurant_ptr = None
         self.assertIsNone(italian_restaurant.pk)
         self.assertIsNone(italian_restaurant.id)
+
+    def test_create_new_instance_with_pk_equals_none(self):
+        p1 = Profile.objects.create(username='john')
+        p2 = User.objects.get(pk=p1.user_ptr_id).profile
+        # Create a new profile by setting pk = None.
+        p2.pk = None
+        p2.user_ptr_id = None
+        p2.username = 'bill'
+        p2.save()
+        self.assertEqual(Profile.objects.count(), 2)
+        self.assertEqual(User.objects.get(pk=p1.user_ptr_id).username, 'john')
+
+    def test_create_new_instance_with_pk_equals_none_multi_inheritance(self):
+        c1 = Congressman.objects.create(state='PA', name='John', title='senator 1')
+        c2 = Person.objects.get(pk=c1.pk).congressman
+        # Create a new congressman by setting pk = None.
+        c2.pk = None
+        c2.id = None
+        c2.politician_ptr_id = None
+        c2.name = 'Bill'
+        c2.title = 'senator 2'
+        c2.save()
+        self.assertEqual(Congressman.objects.count(), 2)
+        self.assertEqual(Person.objects.get(pk=c1.pk).name, 'John')
+        self.assertEqual(
+            Politician.objects.get(pk=c1.politician_ptr_id).title,
+            'senator 1',
+        )

</original_test_patch>

<candidate_a_patch>
diff --git a/django/db/models/base.py b/django/db/models/base.py
index 844c01e95e..8ab612bbb1 100644
--- a/django/db/models/base.py
+++ b/django/db/models/base.py
@@ -569,6 +569,12 @@ class Model(metaclass=ModelBase):
         return getattr(self, meta.pk.attname)
 
     def _set_pk_val(self, value):
+        for parent_link in self._meta.parents.values():
+            if parent_link and parent_link != self._meta.pk:
+                setattr(self, parent_link.attname, value)
+        if value is None:
+            for parent in self._meta.parents:
+                setattr(self, parent._meta.pk.attname, value)
         return setattr(self, self._meta.pk.attname, value)
 
     pk = property(_get_pk_val, _set_pk_val)
@@ -847,13 +853,15 @@ class Model(metaclass=ModelBase):
         if not pk_set and (force_update or update_fields):
             raise ValueError("Cannot force an update in save() with no primary key.")
         updated = False
-        # Skip an UPDATE when adding an instance and primary key has a default.
+        # Skip an UPDATE when adding an instance and primary key has a default
+        # or when pk is None (to create a new instance).
         if (
             not raw and
             not force_insert and
-            self._state.adding and
-            self._meta.pk.default and
-            self._meta.pk.default is not NOT_PROVIDED
+            (self._state.adding and
+             self._meta.pk.default and
+             self._meta.pk.default is not NOT_PROVIDED or
+             pk_val is None)
         ):
             force_insert = True
         # If possible, try an UPDATE. If that doesn't update anything, do an INSERT.

</candidate_a_patch>

<candidate_b_patch>
diff --git a/django/db/models/base.py b/django/db/models/base.py
--- a/django/db/models/base.py
+++ b/django/db/models/base.py
@@ -569,6 +569,9 @@ def _get_pk_val(self, meta=None):
         return getattr(self, meta.pk.attname)
 
     def _set_pk_val(self, value):
+        for parent_link in self._meta.parents.values():
+            if parent_link and parent_link != self._meta.pk:
+                setattr(self, parent_link.target_field.attname, value)
         return setattr(self, self._meta.pk.attname, value)
 
     pk = property(_get_pk_val, _set_pk_val)

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_save_with_explicit_new_pk",
  "specification_gap": "Assigning a valid non-None value through a child model's public pk property must propagate that value to the target primary-key field of a non-primary parent link. The existing tests cover only None and manually clear link fields, so they don't distinguish updating the parent's target field from incorrectly updating the child link column.",
  "input_description": "Create a Profile whose own primary key is distinct from its inherited User identity, assign an unused integer to profile.pk, change its username, and save it.",
  "expected_output": "The save succeeds and creates a second Profile/User pair. The original profile still has username 'john'; the copied profile has the explicit new primary key, username 'bill', and a User parent with that same new identity.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "Candidate B propagates the assigned value to User.id, allowing the parent and child rows to be inserted under the new identity. Candidate A writes user_ptr_id instead while leaving User.id unchanged, so saving updates the original parent and then attempts to reuse its one-to-one link, typically raising an integrity error. This exercises the general pk setter contract rather than only the benchmark's None-copy recipe.",
  "test_patch": "diff --git a/tests/model_inheritance_regress/tests.py b/tests/model_inheritance_regress/tests.py\n--- a/tests/model_inheritance_regress/tests.py\n+++ b/tests/model_inheritance_regress/tests.py\n@@ -558,3 +558,21 @@ class ModelInheritanceTest(TestCase):\n         italian_restaurant.restaurant_ptr = None\n         self.assertIsNone(italian_restaurant.pk)\n         self.assertIsNone(italian_restaurant.id)\n+\n+    def test_save_with_explicit_new_pk(self):\n+        profile = Profile.objects.create(username='john')\n+        old_pk = profile.pk\n+        new_pk = old_pk + 100\n+\n+        profile.pk = new_pk\n+        profile.username = 'bill'\n+        profile.save()\n+\n+        self.assertEqual(Profile.objects.count(), 2)\n+        self.assertEqual(\n+            Profile.objects.get(pk=old_pk).username,\n+            'john',\n+        )\n+        copied = Profile.objects.get(pk=new_pk)\n+        self.assertEqual(copied.username, 'bill')\n+        self.assertEqual(copied.user_ptr_id, new_pk)\n",
  "test_command": "./tests/runtests.py model_inheritance_regress.tests.ModelInheritanceTest.test_save_with_explicit_new_pk"
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
ETesting against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).

======================================================================
ERROR: test_save_with_explicit_new_pk (model_inheritance_regress.tests.ModelInheritanceTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/django/db/backends/utils.py", line 84, in _execute
    return self.cursor.execute(sql, params)
  File "/testbed/django/db/backends/sqlite3/base.py", line 401, in execute
    return Database.Cursor.execute(self, query, params)
sqlite3.IntegrityError: UNIQUE constraint failed: model_inheritance_regress_profile.user_ptr_id

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
  File "/testbed/tests/model_inheritance_regress/tests.py", line 569, in test_save_with_explicit_new_pk
    profile.save()
  File "/testbed/django/db/models/base.py", line 753, in save
    force_update=force_update, update_fields=update_fields)
  File "/testbed/django/db/models/base.py", line 791, in save_base
    force_update, using, update_fields,
  File "/testbed/django/db/models/base.py", line 896, in _save_table
    results = self._do_insert(cls._base_manager, using, fields, returning_fields, raw)
  File "/testbed/django/db/models/base.py", line 935, in _do_insert
    using=using, raw=raw,
  File "/testbed/django/db/models/manager.py", line 82, in manager_method
    return getattr(self.get_queryset(), name)(*args, **kwargs)
  File "/testbed/django/db/models/query.py", line 1228, in _insert
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
django.db.utils.IntegrityError: UNIQUE constraint failed: model_inheritance_regress_profile.user_ptr_id

----------------------------------------------------------------------
Ran 1 test in 0.002s

FAILED (errors=1)
Destroying test database for alias 'default'...
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
Ran 1 test in 0.002s

OK
Destroying test database for alias 'default'...
[pipeline] test_exit_code=0

```
</validated_execution>
