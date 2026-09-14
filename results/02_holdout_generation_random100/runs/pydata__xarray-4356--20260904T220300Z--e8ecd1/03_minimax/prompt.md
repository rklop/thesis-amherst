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

<original_test_patch>
diff --git a/xarray/tests/test_duck_array_ops.py b/xarray/tests/test_duck_array_ops.py
--- a/xarray/tests/test_duck_array_ops.py
+++ b/xarray/tests/test_duck_array_ops.py
@@ -595,6 +595,24 @@ def test_min_count(dim_num, dtype, dask, func, aggdim):
     assert_dask_array(actual, dask)
 
 
+@pytest.mark.parametrize("dtype", [float, int, np.float32, np.bool_])
+@pytest.mark.parametrize("dask", [False, True])
+@pytest.mark.parametrize("func", ["sum", "prod"])
+def test_min_count_nd(dtype, dask, func):
+    if dask and not has_dask:
+        pytest.skip("requires dask")
+
+    min_count = 3
+    dim_num = 3
+    da = construct_dataarray(dim_num, dtype, contains_nan=True, dask=dask)
+    actual = getattr(da, func)(dim=["x", "y", "z"], skipna=True, min_count=min_count)
+    # Supplying all dims is equivalent to supplying `...` or `None`
+    expected = getattr(da, func)(dim=..., skipna=True, min_count=min_count)
+
+    assert_allclose(actual, expected)
+    assert_dask_array(actual, dask)
+
+
 @pytest.mark.parametrize("func", ["sum", "prod"])
 def test_min_count_dataset(func):
     da = construct_dataarray(2, dtype=float, contains_nan=True, dask=False)
@@ -606,14 +624,15 @@ def test_min_count_dataset(func):
 
 @pytest.mark.parametrize("dtype", [float, int, np.float32, np.bool_])
 @pytest.mark.parametrize("dask", [False, True])
+@pytest.mark.parametrize("skipna", [False, True])
 @pytest.mark.parametrize("func", ["sum", "prod"])
-def test_multiple_dims(dtype, dask, func):
+def test_multiple_dims(dtype, dask, skipna, func):
     if dask and not has_dask:
         pytest.skip("requires dask")
     da = construct_dataarray(3, dtype, contains_nan=True, dask=dask)
 
-    actual = getattr(da, func)(("x", "y"))
-    expected = getattr(getattr(da, func)("x"), func)("y")
+    actual = getattr(da, func)(("x", "y"), skipna=skipna)
+    expected = getattr(getattr(da, func)("x", skipna=skipna), func)("y", skipna=skipna)
     assert_allclose(actual, expected)
 
 

</original_test_patch>

<candidate_a_patch>
diff --git a/xarray/core/nanops.py b/xarray/core/nanops.py
index 41c8d258..fd185a7b 100644
--- a/xarray/core/nanops.py
+++ b/xarray/core/nanops.py
@@ -26,19 +26,20 @@ def _maybe_null_out(result, axis, mask, min_count=1):
     """
     xarray version of pandas.core.nanops._maybe_null_out
     """
-    if hasattr(axis, "__len__"):  # if tuple or list
-        raise ValueError(
-            "min_count is not available for reduction with more than one dimensions."
-        )
 
-    if axis is not None and getattr(result, "ndim", False):
-        null_mask = (mask.shape[axis] - mask.sum(axis) - min_count) < 0
+    if axis is not None and getattr(result, "ndim", 0) > 0:
+        if hasattr(axis, "__len__"):  # if tuple or list
+            null_mask = (
+                np.take(mask.shape, axis).prod() - mask.sum(axis=axis) - min_count
+            ) < 0
+        else:
+            null_mask = (mask.shape[axis] - mask.sum(axis) - min_count) < 0
         if null_mask.any():
             dtype, fill_value = dtypes.maybe_promote(result.dtype)
             result = result.astype(dtype)
             result[null_mask] = fill_value
 
-    elif getattr(result, "dtype", None) not in dtypes.NAT_TYPES:
+    elif hasattr(result, "dtype"):  # excludes scalar-ish types like float, int
         null_mask = mask.size - mask.sum()
         if null_mask < min_count:
             result = np.nan

</candidate_a_patch>

<candidate_b_patch>
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

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_sum_min_count_multiple_dims_object_scalar",
  "specification_gap": "A multidimensional reduction can return a Python scalar when the DataArray has object dtype. The documented min_count rule must still replace that scalar result with NA when too few non-NA values are present.",
  "input_description": "A 2\u00d72 object-dtype DataArray containing one valid Python float and three NaNs is summed over both dimensions with skipna=True and min_count=2.",
  "expected_output": "The zero-dimensional result contains NaN because only one valid value is present. Candidate_b preserves scalar nulling; candidate_a returns the partial sum 2.5 because its added dtype check excludes Python scalar results.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "This covers a public dtype/return-arity boundary absent from the reported numeric example. It verifies that the general min_count invariant applies even when NumPy's object reduction returns a Python scalar rather than an ndarray or NumPy scalar.",
  "test_patch": "diff --git a/xarray/tests/test_sum_min_count_multidim.py b/xarray/tests/test_sum_min_count_multidim.py\nnew file mode 100644\n--- /dev/null\n+++ b/xarray/tests/test_sum_min_count_multidim.py\n@@ -0,0 +1,10 @@\n+import numpy as np\n+\n+import xarray as xr\n+\n+\n+def test_sum_min_count_multiple_dims_object_scalar():\n+    values = np.array([[np.nan, 2.5], [np.nan, np.nan]], dtype=object)\n+    array = xr.DataArray(values, dims=(\"x\", \"y\"))\n+\n+    assert np.isnan(array.sum([\"x\", \"y\"], skipna=True, min_count=2).values.item())\n",
  "test_command": "cd /testbed && python -m pytest -q xarray/tests/test_sum_min_count_multidim.py::test_sum_min_count_multiple_dims_object_scalar"
}
</generated_test_proposal>

<validated_execution>
## candidate_a
returncode=1 passed=False test_ran=True
log=02_execution/attempt_02/candidate_a.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
F                                                                        [100%]
=================================== FAILURES ===================================
________________ test_sum_min_count_multiple_dims_object_scalar ________________

    def test_sum_min_count_multiple_dims_object_scalar():
        values = np.array([[np.nan, 2.5], [np.nan, np.nan]], dtype=object)
        array = xr.DataArray(values, dims=("x", "y"))
    
>       assert np.isnan(array.sum(["x", "y"], skipna=True, min_count=2).values.item())
E       AssertionError: assert False
E        +  where False = <ufunc 'isnan'>(2.5)
E        +    where <ufunc 'isnan'> = np.isnan
E        +    and   2.5 = <built-in method item of numpy.ndarray object at 0x72d935461770>()
E        +      where <built-in method item of numpy.ndarray object at 0x72d935461770> = array(2.5).item
E        +        where array(2.5) = <xarray.DataArray ()>\narray(2.5).values
E        +          where <xarray.DataArray ()>\narray(2.5) = <bound method ImplementsArrayReduce._reduce_method.<locals>.wrapped_func of <xarray.DataArray (x: 2, y: 2)>\narray([[nan, 2.5],\n       [nan, nan]], dtype=object)\nDimensions without coordinates: x, y>(['x', 'y'], skipna=True, min_count=2)
E        +            where <bound method ImplementsArrayReduce._reduce_method.<locals>.wrapped_func of <xarray.DataArray (x: 2, y: 2)>\narray([[nan, 2.5],\n       [nan, nan]], dtype=object)\nDimensions without coordinates: x, y> = <xarray.DataArray (x: 2, y: 2)>\narray([[nan, 2.5],\n       [nan, nan]], dtype=object)\nDimensions without coordinates: x, y.sum

xarray/tests/test_sum_min_count_multidim.py:10: AssertionError
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
FAILED xarray/tests/test_sum_min_count_multidim.py::test_sum_min_count_multiple_dims_object_scalar
1 failed, 11 warnings in 2.64s
[pipeline] test_exit_code=1

```

## candidate_b
returncode=0 passed=True test_ran=True
log=02_execution/attempt_02/candidate_b.log
```
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
1 passed, 11 warnings in 2.13s
[pipeline] test_exit_code=0

```
</validated_execution>
