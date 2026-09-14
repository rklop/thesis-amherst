Act as a careful human software engineer reviewing a potential
negative specification test. Return exactly one JSON object and no Markdown.

The generated test separates two plausible patches, but the actual SWE-bench
gold patch executes and fails it. The question is not whether the assertion
should pass. The question is whether this failure reliably specifies behavior
that a final patch must NOT be required to implement. Extract only a narrow,
implementation-independent negative constraint supported by the issue and gold
evidence. Do not infer that every gold failure is useful.

Use this policy:
- `accept` only when the target test demonstrably ran, its failure is behavioral
  (not collection/setup/infrastructure), and the assertion captures a plausible,
  issue-relevant behavior that the gold evidence specifically declines to require.
- `reject` for infrastructure or collection failures, incidental output, private
  implementation coupling, unrelated behavior, unstable/undefined behavior, or
  when failure does not justify a reusable negative constraint.
- `human_review` only for a consequential ambiguity not resolvable from evidence.

eligible_failure_fingerprint=true. If false, do not accept.
An arbitrary nonzero exit is never sufficient.

Required JSON keys: `decision`, `confidence`, `negative_specification`,
`forbidden_behavior`, `gold_failure_interpretation`, `specification_value`,
`decision_rationale`, `concerns`, `human_review_questions`.

<issue_statement>
sum: min_count is not available for reduction with more than one dimensions
**Is your feature request related to a problem? Please describe.**

`sum` with `min_count` errors when passing more than one dim:

```python
import xarray as xr
da = xr.DataArray([[1., 2, 3], [4, 5, 6]])
da.sum(["dim_0", "dim_1"], min_count=1)
```

**Describe the solution you'd like**
The logic to calculate the number of valid elements is here:
https://github.com/pydata/xarray/blob/1be777fe725a85b8cc0f65a2bc41f4bc2ba18043/xarray/core/nanops.py#L35

I *think* this can be fixed by replacing

`mask.shape[axis]` with `np.take(a.shape, axis).prod()`

**Additional context**
Potentially relevant for #4351


</issue_statement>
<candidate_a_patch>
diff --git a/xarray/core/nanops.py b/xarray/core/nanops.py
index 41c8d258..60d8c8c7 100644
--- a/xarray/core/nanops.py
+++ b/xarray/core/nanops.py
@@ -26,13 +26,16 @@ def _maybe_null_out(result, axis, mask, min_count=1):
     """
     xarray version of pandas.core.nanops._maybe_null_out
     """
-    if hasattr(axis, "__len__"):  # if tuple or list
-        raise ValueError(
-            "min_count is not available for reduction with more than one dimensions."
-        )
-
     if axis is not None and getattr(result, "ndim", False):
-        null_mask = (mask.shape[axis] - mask.sum(axis) - min_count) < 0
+        # Calculate the number of elements along the given axis/axes
+        if hasattr(axis, "__len__"):  # if tuple or list
+            # For multiple axes, compute the product of shapes
+            axis_shape = np.take(mask.shape, axis).prod()
+        else:
+            # For single axis
+            axis_shape = mask.shape[axis]
+
+        null_mask = (axis_shape - mask.sum(axis) - min_count) < 0
         if null_mask.any():
             dtype, fill_value = dtypes.maybe_promote(result.dtype)
             result = result.astype(dtype)

</candidate_a_patch>
<candidate_b_patch>
diff --git a/xarray/core/nanops.py b/xarray/core/nanops.py
index 41c8d258..78fed923 100644
--- a/xarray/core/nanops.py
+++ b/xarray/core/nanops.py
@@ -26,19 +26,34 @@ def _maybe_null_out(result, axis, mask, min_count=1):
     """
     xarray version of pandas.core.nanops._maybe_null_out
     """
-    if hasattr(axis, "__len__"):  # if tuple or list
-        raise ValueError(
-            "min_count is not available for reduction with more than one dimensions."
-        )
-
-    if axis is not None and getattr(result, "ndim", False):
-        null_mask = (mask.shape[axis] - mask.sum(axis) - min_count) < 0
-        if null_mask.any():
-            dtype, fill_value = dtypes.maybe_promote(result.dtype)
-            result = result.astype(dtype)
-            result[null_mask] = fill_value
-
-    elif getattr(result, "dtype", None) not in dtypes.NAT_TYPES:
+
+    if axis is not None and hasattr(result, "ndim"):
+        # np.take and prod work for both single axis and multiple axes
+        null_mask = (np.prod(np.take(mask.shape, axis)) - mask.sum(axis) - min_count) < 0
+        if isinstance(result, np.ndarray):
+            if null_mask.any():
+                dtype, fill_value = dtypes.maybe_promote(result.dtype)
+                result = result.astype(dtype)
+                result[null_mask] = fill_value
+        elif isinstance(result, dask_array_type):
+            # dask array - compute the null_mask check
+            if null_mask.any():
+                dtype, fill_value = dtypes.maybe_promote(result.dtype)
+                result = result.astype(dtype)
+                result[null_mask] = fill_value
+        else:
+            # result is a numpy scalar
+            if null_mask:
+                result = np.nan
+
+    elif hasattr(result, "dtype"):
+        # result has a dtype attribute (numpy scalar or array)
+        if result.dtype not in dtypes.NAT_TYPES:
+            null_mask = mask.size - mask.sum()
+            if null_mask < min_count:
+                result = np.nan
+    else:
+        # result is a Python scalar (e.g., from object array reduction)
         null_mask = mask.size - mask.sum()
         if null_mask < min_count:
             result = np.nan

</candidate_b_patch>
<generated_test_proposal>
{
  "test_name": "test_min_count_multiple_dims_keeps_dask_scalar_lazy",
  "specification_gap": "A multi-dimensional min_count reduction that produces a scalar should preserve the lazy Dask backend, including when too few valid values make the result missing.",
  "input_description": "A Dask-backed 1x2 float DataArray containing 1.0 and NaN, summed over both named dimensions with skipna=True and min_count=2.",
  "expected_output": "A zero-dimensional, Dask-backed DataArray that computes to NaN.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "This exercises the scalar-output boundary of the reported multi-axis behavior. candidate_a bypasses its axis-aware branch when result.ndim is zero and replaces the lazy scalar with eager numpy.nan; candidate_b explicitly handles zero-dimensional Dask results and preserves the backend.",
  "test_patch": "diff --git a/xarray/tests/test_duck_array_ops.py b/xarray/tests/test_duck_array_ops.py\n--- a/xarray/tests/test_duck_array_ops.py\n+++ b/xarray/tests/test_duck_array_ops.py\n@@ -592,7 +592,24 @@ def test_min_count(dim_num, dtype, dask, func, aggdim):\n     actual = getattr(da, func)(dim=aggdim, skipna=True, min_count=min_count)\n     expected = series_reduce(da, func, skipna=True, dim=aggdim, min_count=min_count)\n     assert_allclose(actual, expected)\n     assert_dask_array(actual, dask)\n \n \n+@requires_dask\n+def test_min_count_multiple_dims_keeps_dask_scalar_lazy():\n+    import dask.array as da\n+\n+    source = DataArray(\n+        da.from_array(\n+            np.array([[1.0, np.nan]], dtype=np.float32), chunks=(1, 2)\n+        ),\n+        dims=(\"x\", \"y\"),\n+    )\n+\n+    actual = source.sum((\"x\", \"y\"), skipna=True, min_count=2)\n+\n+    assert isinstance(actual.data, dask_array_type)\n+    assert np.isnan(actual.compute().item())\n+\n+\n @pytest.mark.parametrize(\"func\", [\"sum\", \"prod\"])\n",
  "test_command": "cd /testbed && python -m pytest -q xarray/tests/test_duck_array_ops.py::test_min_count_multiple_dims_keeps_dask_scalar_lazy"
}
</generated_test_proposal>
<candidate_execution_summary>
{
  "attempt": 2,
  "separates": true,
  "required_passing_candidate": "candidate_b",
  "meets_acceptance_rule": true,
  "results": [
    {
      "label": "candidate_a",
      "returncode": 1,
      "passed": false,
      "test_ran": true,
      "duration_seconds": 4.071,
      "log_path": "02_execution/attempt_02/candidate_a.log"
    },
    {
      "label": "candidate_b",
      "returncode": 0,
      "passed": true,
      "test_ran": true,
      "duration_seconds": 3.175,
      "log_path": "02_execution/attempt_02/candidate_b.log"
    }
  ]
}
</candidate_execution_summary>
<candidate_a_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
F                                                                        [100%]
=================================== FAILURES ===================================
_____________ test_min_count_multiple_dims_keeps_dask_scalar_lazy ______________

    @requires_dask
    def test_min_count_multiple_dims_keeps_dask_scalar_lazy():
        import dask.array as da
    
        source = DataArray(
            da.from_array(
                np.array([[1.0, np.nan]], dtype=np.float32), chunks=(1, 2)
            ),
            dims=("x", "y"),
        )
    
        actual = source.sum(("x", "y"), skipna=True, min_count=2)
    
>       assert isinstance(actual.data, dask_array_type)
E       AssertionError: assert False
E        +  where False = isinstance(array(nan), (<class 'dask.array.core.Array'>,))
E        +    where array(nan) = <xarray.DataArray 'array-bfacc857f1b25f890eefc20c40e1470e' ()>\narray(nan).data

xarray/tests/test_duck_array_ops.py:611: AssertionError
=============================== warnings summary ===============================
xarray/__init__.py:1
  /testbed/xarray/__init__.py:1: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
    import pkg_resources

xarray/core/dask_array_compat.py:16
xarray/core/dask_array_compat.py:16
  /testbed/xarray/core/dask_array_compat.py:16: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    if LooseVersion(dask_version) >= LooseVersion("2.0.0"):

xarray/core/dask_array_compat.py:149
xarray/core/dask_array_compat.py:149
  /testbed/xarray/core/dask_array_compat.py:149: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    if LooseVersion(dask_version) >= LooseVersion("2.8.1"):

xarray/core/dask_array_compat.py:186
xarray/core/dask_array_compat.py:186
  /testbed/xarray/core/dask_array_compat.py:186: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    if LooseVersion(dask_version) > LooseVersion("2.9.0"):

xarray/core/pdcompat.py:45
  /testbed/xarray/core/pdcompat.py:45: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    if LooseVersion(pd.__version__) < "0.25.0":

../opt/miniconda3/envs/testbed/lib/python3.10/site-packages/setuptools/_distutils/version.py:345
  /opt/miniconda3/envs/testbed/lib/python3.10/site-packages/setuptools/_distutils/version.py:345: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    other = LooseVersion(other)

xarray/tests/__init__.py:58
xarray/tests/__init__.py:58
  /testbed/xarray/tests/__init__.py:58: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    return version.LooseVersion(vstring)

-- Docs: https://docs.pytest.org/en/stable/how-to/capture-warnings.html
=========================== short test summary info ============================
FAILED xarray/tests/test_duck_array_ops.py::test_min_count_multiple_dims_keeps_dask_scalar_lazy
1 failed, 11 warnings in 2.93s
[pipeline] test_exit_code=1

</candidate_a_execution_log>
<candidate_b_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
.                                                                        [100%]
=============================== warnings summary ===============================
xarray/__init__.py:1
  /testbed/xarray/__init__.py:1: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
    import pkg_resources

xarray/core/dask_array_compat.py:16
xarray/core/dask_array_compat.py:16
  /testbed/xarray/core/dask_array_compat.py:16: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    if LooseVersion(dask_version) >= LooseVersion("2.0.0"):

xarray/core/dask_array_compat.py:149
xarray/core/dask_array_compat.py:149
  /testbed/xarray/core/dask_array_compat.py:149: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    if LooseVersion(dask_version) >= LooseVersion("2.8.1"):

xarray/core/dask_array_compat.py:186
xarray/core/dask_array_compat.py:186
  /testbed/xarray/core/dask_array_compat.py:186: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    if LooseVersion(dask_version) > LooseVersion("2.9.0"):

xarray/core/pdcompat.py:45
  /testbed/xarray/core/pdcompat.py:45: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    if LooseVersion(pd.__version__) < "0.25.0":

../opt/miniconda3/envs/testbed/lib/python3.10/site-packages/setuptools/_distutils/version.py:345
  /opt/miniconda3/envs/testbed/lib/python3.10/site-packages/setuptools/_distutils/version.py:345: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    other = LooseVersion(other)

xarray/tests/__init__.py:58
xarray/tests/__init__.py:58
  /testbed/xarray/tests/__init__.py:58: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    return version.LooseVersion(vstring)

-- Docs: https://docs.pytest.org/en/stable/how-to/capture-warnings.html
1 passed, 11 warnings in 2.14s
[pipeline] test_exit_code=0

</candidate_b_execution_log>
<official_gold_patch>
diff --git a/xarray/core/nanops.py b/xarray/core/nanops.py
--- a/xarray/core/nanops.py
+++ b/xarray/core/nanops.py
@@ -26,13 +26,9 @@ def _maybe_null_out(result, axis, mask, min_count=1):
     """
     xarray version of pandas.core.nanops._maybe_null_out
     """
-    if hasattr(axis, "__len__"):  # if tuple or list
-        raise ValueError(
-            "min_count is not available for reduction with more than one dimensions."
-        )
 
     if axis is not None and getattr(result, "ndim", False):
-        null_mask = (mask.shape[axis] - mask.sum(axis) - min_count) < 0
+        null_mask = (np.take(mask.shape, axis).prod() - mask.sum(axis) - min_count) < 0
         if null_mask.any():
             dtype, fill_value = dtypes.maybe_promote(result.dtype)
             result = result.astype(dtype)

</official_gold_patch>
<gold_execution_summary>
{
  "label": "gold",
  "returncode": 1,
  "passed": false,
  "test_ran": true,
  "duration_seconds": 3.572,
  "log_path": "gold_execution/gold.log"
}
</gold_execution_summary>
<gold_failure_contract>
{
  "kind": "behavioral_test_failure",
  "required_substrings": [
    "test_min_count_multiple_dims_keeps_dask_scalar_lazy"
  ],
  "required_any_substrings": [
    "AssertionError",
    "FAILED",
    " failed,"
  ],
  "forbidden_substrings": [
    "collected 0 items",
    "no tests ran",
    "ERROR collecting",
    "command not found",
    "No such file or directory",
    "Could not find a version that satisfies the requirement",
    "Temporary failure in name resolution"
  ],
  "reference_returncode": 1,
  "reference_log_sha256": "70d8eeec51d23627be95bec697b6a1044cd07357c10103ac1975f12f03fb99d1"
}
</gold_failure_contract>
<gold_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
F                                                                        [100%]
=================================== FAILURES ===================================
_____________ test_min_count_multiple_dims_keeps_dask_scalar_lazy ______________

    @requires_dask
    def test_min_count_multiple_dims_keeps_dask_scalar_lazy():
        import dask.array as da
    
        source = DataArray(
            da.from_array(
                np.array([[1.0, np.nan]], dtype=np.float32), chunks=(1, 2)
            ),
            dims=("x", "y"),
        )
    
        actual = source.sum(("x", "y"), skipna=True, min_count=2)
    
>       assert isinstance(actual.data, dask_array_type)
E       AssertionError: assert False
E        +  where False = isinstance(array(nan), (<class 'dask.array.core.Array'>,))
E        +    where array(nan) = <xarray.DataArray 'array-bfacc857f1b25f890eefc20c40e1470e' ()>\narray(nan).data

xarray/tests/test_duck_array_ops.py:611: AssertionError
=============================== warnings summary ===============================
xarray/__init__.py:1
  /testbed/xarray/__init__.py:1: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
    import pkg_resources

xarray/core/dask_array_compat.py:16
xarray/core/dask_array_compat.py:16
  /testbed/xarray/core/dask_array_compat.py:16: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    if LooseVersion(dask_version) >= LooseVersion("2.0.0"):

xarray/core/dask_array_compat.py:149
xarray/core/dask_array_compat.py:149
  /testbed/xarray/core/dask_array_compat.py:149: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    if LooseVersion(dask_version) >= LooseVersion("2.8.1"):

xarray/core/dask_array_compat.py:186
xarray/core/dask_array_compat.py:186
  /testbed/xarray/core/dask_array_compat.py:186: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    if LooseVersion(dask_version) > LooseVersion("2.9.0"):

xarray/core/pdcompat.py:45
  /testbed/xarray/core/pdcompat.py:45: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    if LooseVersion(pd.__version__) < "0.25.0":

../opt/miniconda3/envs/testbed/lib/python3.10/site-packages/setuptools/_distutils/version.py:345
  /opt/miniconda3/envs/testbed/lib/python3.10/site-packages/setuptools/_distutils/version.py:345: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    other = LooseVersion(other)

xarray/tests/__init__.py:58
xarray/tests/__init__.py:58
  /testbed/xarray/tests/__init__.py:58: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    return version.LooseVersion(vstring)

-- Docs: https://docs.pytest.org/en/stable/how-to/capture-warnings.html
=========================== short test summary info ============================
FAILED xarray/tests/test_duck_array_ops.py::test_min_count_multiple_dims_keeps_dask_scalar_lazy
1 failed, 11 warnings in 2.46s
[pipeline] test_exit_code=1

</gold_execution_log>
