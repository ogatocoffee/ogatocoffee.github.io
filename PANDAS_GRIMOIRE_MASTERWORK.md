# 10 minutes to pandas
This is a short introduction to pandas, geared mainly for new users.
You can see more complex recipes in the `Cookbook<cookbook>`.

Customarily, we import as follows:

```python
import numpy as np
import pandas as pd
```
## Basic data structures in pandas
pandas provides two types of classes for handling data:

1. `Series`: a one-dimensional labeled array holding data of any type
    such as integers, strings, Python objects etc.
2. `DataFrame`: a two-dimensional data structure that holds data like
   a two-dimension array or a table with rows and columns.

## Object creation
See the `Intro to data structures section <dsintro>`.

Creating a `Series` by passing a list of values, letting pandas create
a default `RangeIndex`.

```python
s = pd.Series([1, 3, 5, np.nan, 6, 8])
s
```
Creating a `DataFrame` by passing a NumPy array with a datetime index using `date_range`
and labeled columns:

```python
dates = pd.date_range("20130101", periods=6)
dates
df = pd.DataFrame(np.random.randn(6, 4), index=dates, columns=list("ABCD"))
df
```
Creating a `DataFrame` by passing a dictionary of objects where the keys are the column
labels and the values are the column values.

```python
df2 = pd.DataFrame(
    {
        "A": 1.0,
        "B": pd.Timestamp("20130102"),
        "C": pd.Series(1, index=list(range(4)), dtype="float32"),
        "D": np.array([3] * 4, dtype="int32"),
        "E": pd.Categorical(["test", "train", "test", "train"]),
        "F": "foo",
    }
)
df2
```
The columns of the resulting `DataFrame` have different
`dtypes <basics.dtypes>`:

```python
df2.dtypes
```
If you're using IPython, tab completion for column names (as well as public
attributes) is automatically enabled. Here's a subset of the attributes that
will be completed:



   @verbatim
   In [1]: df2.<TAB>  # noqa: E225, E999
   df2.A                  df2.bool
   df2.abs                df2.boxplot
   df2.add                df2.C
   df2.add_prefix         df2.clip
   df2.add_suffix         df2.columns
   df2.align              df2.copy
   df2.all                df2.count
   df2.any                df2.combine
   df2.append             df2.D
   df2.apply              df2.describe
   df2.B                  df2.duplicated
   df2.diff

As you can see, the columns ``A``, ``B``, ``C``, and ``D`` are automatically
tab completed. ``E`` and ``F`` are there as well; the rest of the attributes have been
truncated for brevity.

## Viewing data
See the `Essential basic functionality section <basics>`.

Use `DataFrame.head` and `DataFrame.tail` to view the top and bottom rows of the frame
respectively:

```python
df.head()
df.tail(3)
```
Display the `DataFrame.index` or `DataFrame.columns`:

```python
df.index
df.columns
```
Return a NumPy representation of the underlying data with `DataFrame.to_numpy`
without the index or column labels:

```python
df.to_numpy()
```
> **note.capitalize():**
   **NumPy arrays have one dtype for the entire array while pandas DataFrames
   have one dtype per column**. When you call `DataFrame.to_numpy`, pandas will
   find the NumPy dtype that can hold *all* of the dtypes in the DataFrame.
   If the common data type is ``object``, `DataFrame.to_numpy` will require
   copying data.

   ```python
df2.dtypes
df2.to_numpy()
```
`~DataFrame.describe` shows a quick statistic summary of your data:

```python
df.describe()
```
Transposing your data:

```python
df.T
```
`DataFrame.sort_index` sorts by an axis:

```python
df.sort_index(axis=1, ascending=False)
```
`DataFrame.sort_values` sorts by values:

```python
df.sort_values(by="B")
```
## Selection
> **note.capitalize():**
   While standard Python / NumPy expressions for selecting and setting are
   intuitive and come in handy for interactive work, for production code, we
   recommend the optimized pandas data access methods, `DataFrame.at`, `DataFrame.iat`,
   `DataFrame.loc` and `DataFrame.iloc`.

See the indexing documentation `Indexing and Selecting Data <indexing>` and `MultiIndex / Advanced Indexing <advanced>`.

### Getitem (``[]``)
For a `DataFrame`, passing a single label selects a column and
yields a `Series`:

```python
df["A"]
```
If the label only contains letters, numbers, and underscores, you can
alternatively use the column name attribute:

```python
df.A
```
Passing a list of column labels selects multiple columns, which can be useful
for getting a subset/rearranging:

```python
df[["B", "A"]]
```
For a `DataFrame`, passing a slice ``:`` selects matching rows:

```python
df[0:3]
df["20130102":"20130104"]
```
### Selection by label
See more in `Selection by Label <indexing.label>` using `DataFrame.loc` or `DataFrame.at`.

Selecting a row matching a label:

```python
df.loc[dates[0]]
```
Selecting all rows (``:``) with a select column labels:

```python
df.loc[:, ["A", "B"]]
```
For label slicing, both endpoints are *included*:

```python
df.loc["20130102":"20130104", ["A", "B"]]
```
Selecting a single row and column label returns a scalar:

```python
df.loc[dates[0], "A"]
```
For getting fast access to a scalar (equivalent to the prior method):

```python
df.at[dates[0], "A"]
```
### Selection by position
See more in `Selection by Position <indexing.integer>` using `DataFrame.iloc` or `DataFrame.iat`.

Select via the position of the passed integers:

```python
df.iloc[3]
```
Integer slices acts similar to NumPy/Python:

```python
df.iloc[3:5, 0:2]
```
Lists of integer position locations:

```python
df.iloc[[1, 2, 4], [0, 2]]
```
For slicing rows explicitly:

```python
df.iloc[1:3, :]
```
For slicing columns explicitly:

```python
df.iloc[:, 1:3]
```
For getting a value explicitly:

```python
df.iloc[1, 1]
```
For getting fast access to a scalar (equivalent to the prior method):

```python
df.iat[1, 1]
```
### Boolean indexing
Select rows where ``df.A`` is greater than ``0``.

```python
df[df["A"] > 0]
```
Selecting values from a `DataFrame` where a boolean condition is met:

```python
df[df > 0]
```
Using `~Series.isin` method for filtering:

```python
df2 = df.copy()
df2["E"] = ["one", "one", "two", "three", "four", "three"]
df2
df2[df2["E"].isin(["two", "four"])]
```
### Setting
Setting a new column automatically aligns the data by the indexes:

```python
s1 = pd.Series([1, 2, 3, 4, 5, 6], index=pd.date_range("20130102", periods=6))
s1
df["F"] = s1
```
Setting values by label:

```python
df.at[dates[0], "A"] = 0
```
Setting values by position:

```python
df.iat[0, 1] = 0
```
Setting by assigning with a NumPy array:

```python
:okwarning:

df.loc[:, "D"] = np.array([5] * len(df))
```
The result of the prior setting operations:

```python
df
```
A ``where`` operation with setting:

```python
df2 = df.copy()
df2[df2 > 0] = -df2
df2
```
## Missing data
For NumPy data types, ``np.nan`` represents missing data. It is by
default not included in computations. See the Missing Data section
.

Reindexing allows you to change/add/delete the index on a specified axis. This
returns a copy of the data:

```python
df1 = df.reindex(index=dates[0:4], columns=list(df.columns) + ["E"])
df1.loc[dates[0] : dates[1], "E"] = 1
df1
```
`DataFrame.dropna` drops any rows that have missing data:

```python
df1.dropna(how="any")
```
`DataFrame.fillna` fills missing data:

```python
df1.fillna(value=5)
```
`isna` gets the boolean mask where values are ``nan``:

```python
pd.isna(df1)
```
## Operations
See the `Basic section on Binary Ops <basics.binop>`.

### Stats
Operations in general *exclude* missing data.

Calculate the mean value for each column:

```python
df.mean()
```
Calculate the mean value for each row:

```python
df.mean(axis=1)
```
Operating with another `Series` or `DataFrame` with a different index or column
will align the result with the union of the index or column labels. In addition, pandas
automatically broadcasts along the specified dimension and will fill unaligned labels with ``np.nan``.

```python
s = pd.Series([1, 3, 5, np.nan, 6, 8], index=dates).shift(2)
s
df.sub(s, axis="index")
```
### User defined functions
`DataFrame.agg` and `DataFrame.transform` applies a user defined function
that reduces or broadcasts its result respectively.

```python
df.agg(lambda x: np.mean(x) * 5.6)
df.transform(lambda x: x * 101.2)
```
### Value Counts
See more at `Histogramming and Discretization <basics.discretization>`.

```python
s = pd.Series(np.random.randint(0, 7, size=10))
s
s.value_counts()
```
### String Methods
`Series` is equipped with a set of string processing methods in the ``str``
attribute that make it easy to operate on each element of the array, as in the
code snippet below. See more at Vectorized String Methods
.

```python
s = pd.Series(["A", "B", "C", "Aaba", "Baca", np.nan, "CABA", "dog", "cat"])
s.str.lower()
```
## Merge
### Concat
pandas provides various facilities for easily combining together `Series` and
`DataFrame` objects with various kinds of set logic for the indexes
and relational algebra functionality in the case of join / merge-type
operations.

See the `Merging section <merging>`.

Concatenating pandas objects together row-wise with `concat`:

```python
df = pd.DataFrame(np.random.randn(10, 4))
df

# break it into pieces
pieces = [df[:3], df[3:7], df[7:]]

pd.concat(pieces)
```
> **note.capitalize():**
   Adding a column to a `DataFrame` is relatively fast. However, adding
   a row requires a copy, and may be expensive. We recommend passing a
   pre-built list of records to the `DataFrame` constructor instead
   of building a `DataFrame` by iteratively appending records to it.

### Join
`merge` enables SQL style join types along specific columns. See the `Database style joining <merging.join>` section.

```python
left = pd.DataFrame({"key": ["foo", "foo"], "lval": [1, 2]})
right = pd.DataFrame({"key": ["foo", "foo"], "rval": [4, 5]})
left
right
pd.merge(left, right, on="key")
```
`merge` on unique keys:

```python
left = pd.DataFrame({"key": ["foo", "bar"], "lval": [1, 2]})
right = pd.DataFrame({"key": ["foo", "bar"], "rval": [4, 5]})
left
right
pd.merge(left, right, on="key")
```
## Grouping
By "group by" we are referring to a process involving one or more of the
following steps:

* **Splitting** the data into groups based on some criteria
* **Applying** a function to each group independently
* **Combining** the results into a data structure

See the `Grouping section <groupby>`.

```python
df = pd.DataFrame(
    {
        "A": ["foo", "bar", "foo", "bar", "foo", "bar", "foo", "foo"],
        "B": ["one", "one", "two", "three", "two", "two", "one", "three"],
        "C": np.random.randn(8),
        "D": np.random.randn(8),
    }
)
df
```
Grouping by a column label, selecting column labels, and then applying the
`.DataFrameGroupBy.sum` function to the resulting
groups:

```python
df.groupby("A")[["C", "D"]].sum()
```
Grouping by multiple columns label forms `MultiIndex`.

```python
df.groupby(["A", "B"]).sum()
```
## Reshaping
See the sections on `Hierarchical Indexing <advanced.hierarchical>` and
`Reshaping <reshaping.stacking>`.

### Stack
```python
arrays = [
   ["bar", "bar", "baz", "baz", "foo", "foo", "qux", "qux"],
   ["one", "two", "one", "two", "one", "two", "one", "two"],
]
index = pd.MultiIndex.from_arrays(arrays, names=["first", "second"])
df = pd.DataFrame(np.random.randn(8, 2), index=index, columns=["A", "B"])
df2 = df[:4]
df2
```
The `~DataFrame.stack` method "compresses" a level in the DataFrame's
columns:

```python
stacked = df2.stack()
stacked
```
With a "stacked" DataFrame or Series (having a `MultiIndex` as the
``index``), the inverse operation of `~DataFrame.stack` is
`~DataFrame.unstack`, which by default unstacks the **last level**:

```python
stacked.unstack()
stacked.unstack(1)
stacked.unstack(0)
```
### Pivot tables
See the section on `Pivot Tables <reshaping.pivot>`.

```python
df = pd.DataFrame(
    {
        "A": ["one", "one", "two", "three"] * 3,
        "B": ["A", "B", "C"] * 4,
        "C": ["foo", "foo", "foo", "bar", "bar", "bar"] * 2,
        "D": np.random.randn(12),
        "E": np.random.randn(12),
    }
)
df
```
`pivot_table` pivots a `DataFrame` specifying the ``values``, ``index`` and ``columns``

```python
pd.pivot_table(df, values="D", index=["A", "B"], columns=["C"])
```
## Time series
pandas has simple, powerful, and efficient functionality for performing
resampling operations during frequency conversion (e.g., converting secondly
data into 5-minutely data). This is extremely common in, but not limited to,
financial applications. See the `Time Series section <timeseries>`.

```python
rng = pd.date_range("1/1/2012", periods=100, freq="s")
ts = pd.Series(np.random.randint(0, 500, len(rng)), index=rng)
ts.resample("5Min").sum()
```
`Series.tz_localize` localizes a time series to a time zone:

```python
rng = pd.date_range("3/6/2012 00:00", periods=5, freq="D")
ts = pd.Series(np.random.randn(len(rng)), rng)
ts
ts_utc = ts.tz_localize("UTC")
ts_utc
```
`Series.tz_convert` converts a timezones aware time series to another time zone:

```python
ts_utc.tz_convert("US/Eastern")
```
Adding a non-fixed duration (`~pandas.tseries.offsets.BusinessDay`) to a time series:

```python
rng
rng + pd.offsets.BusinessDay(5)
```
## Categoricals
pandas can include categorical data in a `DataFrame`. For full docs, see the
`categorical introduction <categorical>` and the `API documentation <api.arrays.categorical>`.

```python
df = pd.DataFrame(
    {"id": [1, 2, 3, 4, 5, 6], "raw_grade": ["a", "b", "b", "a", "a", "e"]}
)
```
Converting the raw grades to a categorical data type:

```python
df["grade"] = df["raw_grade"].astype("category")
df["grade"]
```
Rename the categories to more meaningful names:

```python
new_categories = ["very good", "good", "very bad"]
df["grade"] = df["grade"].cat.rename_categories(new_categories)
```
Reorder the categories and simultaneously add the missing categories (methods under `Series.cat` return a new `Series` by default):

```python
df["grade"] = df["grade"].cat.set_categories(
    ["very bad", "bad", "medium", "good", "very good"]
)
df["grade"]
```
Sorting is per order in the categories, not lexical order:

```python
df.sort_values(by="grade")
```
Grouping by a categorical column with ``observed=False`` also shows empty categories:

```python
df.groupby("grade", observed=False).size()
```
## Plotting
See the `Plotting <visualization>[ docs.

We use the standard convention for referencing the matplotlib API:

```python
import matplotlib.pyplot as plt

plt.close("all")
```
The ``plt.close`` method is used to `close](https://matplotlib.org/stable/api/_as_gen/matplotlib.pyplot.close.html)_ a figure window:

[``python
ts = pd.Series(np.random.randn(1000), index=pd.date_range("1/1/2000", periods=1000))
ts = ts.cumsum()

@savefig series_plot_basic.png
ts.plot();
```
> **note.capitalize():**
   When using Jupyter, the plot will appear using `~Series.plot`.  Otherwise use
   `matplotlib.pyplot.show](https://matplotlib.org/stable/api/_as_gen/matplotlib.pyplot.show.html)_ to show it or
   [matplotlib.pyplot.savefig](https://matplotlib.org/stable/api/_as_gen/matplotlib.pyplot.savefig.html)_ to write it to a file.

`~DataFrame.plot` plots all columns:

```python
df = pd.DataFrame(
    np.random.randn(1000, 4), index=ts.index, columns=["A", "B", "C", "D"]
)

df = df.cumsum()

plt.figure();
df.plot();
@savefig frame_plot_basic.png
plt.legend(loc='best');
```
## Importing and exporting data
See the `IO Tools <io>` section.

### CSV
`Writing to a csv file: <io.store_in_csv>` using `DataFrame.to_csv`

```python
df = pd.DataFrame(np.random.randint(0, 5, (10, 5)))
df.to_csv("foo.csv")
```
`Reading from a csv file: <io.read_csv_table>` using `read_csv`

```python
pd.read_csv("foo.csv")
```
```python
:suppress:

import os

os.remove("foo.csv")
```
### Parquet
Writing to a Parquet file:

```python
df.to_parquet("foo.parquet")
```
Reading from a Parquet file Store using `read_parquet`:

```python
pd.read_parquet("foo.parquet")
```
```python
:suppress:

os.remove("foo.parquet")
```
### Excel
Reading and writing to `Excel <io.excel>`.

Writing to an excel file using `DataFrame.to_excel`:

```python
df.to_excel("foo.xlsx", sheet_name="Sheet1")
```
Reading from an excel file using `read_excel`:

```python
pd.read_excel("foo.xlsx", "Sheet1", index_col=None, na_values=["NA"])
```
```python
:suppress:

os.remove("foo.xlsx")
```
## Gotchas
If you are attempting to perform a boolean operation on a `Series` or `DataFrame`
you might see an exception like:

```python
:okexcept:

 if pd.Series([False, True, False]):
     print("I was true")
```
See `Comparisons<basics.compare>` and `Gotchas<gotchas>` for an explanation and what to do.

---

# Intro to data structures
We'll start with a quick, non-comprehensive overview of the fundamental data
structures in pandas to get you started. The fundamental behavior about data
types, indexing, axis labeling, and alignment apply across all of the
objects. To get started, import NumPy and load pandas into your namespace:

```python
import numpy as np
import pandas as pd
```
Fundamentally, **data alignment is intrinsic**. The link
between labels and data will not be broken unless done so explicitly by you.

We'll give a brief intro to the data structures, then consider all of the broad
categories of functionality and methods in separate sections.



## Series
`Series` is a one-dimensional labeled array capable of holding any data
type (integers, strings, floating point numbers, Python objects, etc.). The axis
labels are collectively referred to as the **index**. The basic method to create a `Series` is to call:

```python
s = pd.Series(data, index=index)
```
Here, ``data`` can be many different things:

* a Python dict
* an ndarray
* a scalar value (like 5)

The passed **index** is a list of axis labels. The constructor's behavior
depends on **data**'s type:

**From ndarray**

If ``data`` is an ndarray, **index** must be the same length as **data**. If no
index is passed, one will be created having values ``[0, ..., len(data) - 1]``.

```python
s = pd.Series(np.random.randn(5), index=["a", "b", "c", "d", "e"])
s
s.index

pd.Series(np.random.randn(5))
```
> **note.capitalize():**
    pandas supports non-unique index values. If an operation
    that does not support duplicate index values is attempted, an exception
    will be raised at that time.

**From dict**

`Series` can be instantiated from dicts:

```python
d = {"b": 1, "a": 0, "c": 2}
pd.Series(d)
```
If an index is passed, the values in data corresponding to the labels in the
index will be pulled out.

```python
d = {"a": 0.0, "b": 1.0, "c": 2.0}
pd.Series(d)
pd.Series(d, index=["b", "c", "d", "a"])
```
> **note.capitalize():**
    NaN (not a number) is the standard missing data marker used in pandas.

**From scalar value**

If ``data`` is a scalar value, the value will be repeated to match
the length of **index**.  If the **index** is not provided, it defaults
to ``RangeIndex(1)``.

```python
pd.Series(5.0, index=["a", "b", "c", "d", "e"])
```
### Series is ndarray-like
`Series` acts very similarly to a `numpy.ndarray` and is a valid argument to most NumPy functions.
However, operations such as slicing will also slice the index.

```python
s.iloc[0]
s.iloc[:3]
s[s > s.median()]
s.iloc[[4, 3, 1]]
np.exp(s)
```
> **note.capitalize():**
   We will address array-based indexing like ``s.iloc[[4, 3, 1]]``
   in the `section on indexing <indexing>`.

Like a NumPy array, a pandas `Series` has a single `~Series.dtype`.

```python
s.dtype
```
This is often a NumPy dtype. However, pandas and 3rd-party libraries
extend NumPy's type system in a few places, in which case the dtype would
be an `~pandas.api.extensions.ExtensionDtype`. Some examples within
pandas are `categorical` and `integer_na`. See `basics.dtypes`
for more.

If you need the actual array backing a `Series`, use `Series.array`.

```python
s.array
```
Accessing the array can be useful when you need to do some operation without the
index (to disable `automatic alignment <dsintro.alignment>`, for example).

`Series.array` will always be an `~pandas.api.extensions.ExtensionArray`.
Briefly, an ExtensionArray is a thin wrapper around one or more *concrete* arrays like a
`numpy.ndarray`. pandas knows how to take an `~pandas.api.extensions.ExtensionArray` and
store it in a `Series` or a column of a `DataFrame`.
See `basics.dtypes` for more.

While `Series` is ndarray-like, if you need an *actual* ndarray, then use
`Series.to_numpy`.

```python
s.to_numpy()
```
Even if the `Series` is backed by a `~pandas.api.extensions.ExtensionArray`,
`Series.to_numpy` will return a NumPy ndarray.

### Series is dict-like
A `Series` is also like a fixed-size dict in that you can get and set values by index
label:

```python
s["a"]
s["e"] = 12.0
s
"e" in s
"f" in s
```
If a label is not contained in the index, an exception is raised:

```python
:okexcept:

s["f"]
```
Using the `Series.get` method, a missing label will return None or specified default:

```python
s.get("f")

s.get("f", np.nan)
```
These labels can also be accessed by `attribute<indexing.attribute_access>[.

### Vectorized operations and label alignment with Series
When working with raw NumPy arrays, looping through value-by-value is usually
not necessary. The same is true when working with `Series` in pandas.
`Series` can also be passed into most NumPy methods expecting an ndarray.

```python
s + s
s * 2
np.exp(s)
```
A key difference between `Series` and ndarray is that operations between `Series`
automatically align the data based on label. Thus, you can write computations
without giving consideration to whether the `Series` involved have the same
labels.

```python
s.iloc[1:] + s.iloc[:-1]
```
The result of an operation between unaligned `Series` will have the **union** of
the indexes involved. If a label is not found in one `Series` or the other, the
result will be marked as missing ``NaN``. Being able to write code without doing
any explicit data alignment grants immense freedom and flexibility in
interactive data analysis and research. The integrated data alignment features
of the pandas data structures set pandas apart from the majority of related
tools for working with labeled data.

> **note.capitalize():**
    In general, we chose to make the default result of operations between
    differently indexed objects yield the **union** of the indexes in order to
    avoid loss of information. Having an index label, though the data is
    missing, is typically important information as part of a computation. You
    of course have the option of dropping labels with missing data via the
    **dropna** function.

### Name attribute


`Series` also has a ``name`` attribute:

```python
s = pd.Series(np.random.randn(5), name="something")
s
s.name
```
The `Series` ``name`` can be assigned automatically in many cases, in particular,
when selecting a single column from a `DataFrame`, the ``name`` will be assigned
the column label.

You can rename a `Series` with the `pandas.Series.rename` method.

```python
s2 = s.rename("different")
s2.name
```
Note that ``s`` and ``s2`` refer to different objects.



## DataFrame
`DataFrame` is a 2-dimensional labeled data structure with columns of
potentially different types. You can think of it like a spreadsheet or SQL
table, or a dict of Series objects. It is generally the most commonly used
pandas object. Like Series, DataFrame accepts many different kinds of input:

* Dict of 1D ndarrays, lists, dicts, or `Series`
* 2-D numpy.ndarray
* `Structured or record
 ](https://numpy.org/doc/stable/user/basics.rec.html)_ ndarray
* A [Series`
* Another `DataFrame`

Along with the data, you can optionally pass **index** (row labels) and
**columns** (column labels) arguments. If you pass an index and / or columns,
you are guaranteeing the index and / or columns of the resulting
DataFrame. Thus, a dict of Series plus a specific index will discard all data
not matching up to the passed index.

If axis labels are not passed, they will be constructed from the input data
based on common sense rules.

### From dict of Series or dicts
The resulting **index** will be the **union** of the indexes of the various
Series. If there are any nested dicts, these will first be converted to
Series. If no columns are passed, the columns will be the ordered list of dict
keys.

```python
d = {
    "one": pd.Series([1.0, 2.0, 3.0], index=["a", "b", "c"]),
    "two": pd.Series([1.0, 2.0, 3.0, 4.0], index=["a", "b", "c", "d"]),
}
df = pd.DataFrame(d)
df

pd.DataFrame(d, index=["d", "b", "a"])
pd.DataFrame(d, index=["d", "b", "a"], columns=["two", "three"])
```
The row and column labels can be accessed respectively by accessing the
**index** and **columns** attributes:

> **note.capitalize():**
   When a particular set of columns is passed along with a dict of data, the
   passed columns override the keys in the dict.

```python
df.index
df.columns
```
### From dict of ndarrays / lists
All ndarrays must share the same length. If an index is passed, it must
also be the same length as the arrays. If no index is passed, the
result will be ``range(n)``, where ``n`` is the array length.

```python
d = {"one": [1.0, 2.0, 3.0, 4.0], "two": [4.0, 3.0, 2.0, 1.0]}
pd.DataFrame(d)
pd.DataFrame(d, index=["a", "b", "c", "d"])
```
### From structured or record array
This case is handled identically to a dict of arrays.

```python
data = np.zeros((2,), dtype=[("A", "i4"), ("B", "f4"), ("C", "S10")])
data[:] = [(1, 2.0, "Hello"), (2, 3.0, "World")]

pd.DataFrame(data)
pd.DataFrame(data, index=["first", "second"])
pd.DataFrame(data, columns=["C", "A", "B"])
```
> **note.capitalize():**
    DataFrame is not intended to work exactly like a 2-dimensional NumPy
    ndarray.



### From a list of dicts
```python
data2 = [{"a": 1, "b": 2}, {"a": 5, "b": 10, "c": 20}]
pd.DataFrame(data2)
pd.DataFrame(data2, index=["first", "second"])
pd.DataFrame(data2, columns=["a", "b"])
```


### From a dict of tuples
You can automatically create a MultiIndexed frame by passing a tuples
dictionary.

```python
pd.DataFrame(
    {
        ("a", "b"): {("A", "B"): 1, ("A", "C"): 2},
        ("a", "a"): {("A", "C"): 3, ("A", "B"): 4},
        ("a", "c"): {("A", "B"): 5, ("A", "C"): 6},
        ("b", "a"): {("A", "C"): 7, ("A", "B"): 8},
        ("b", "b"): {("A", "D"): 9, ("A", "B"): 10},
    }
)
```


### From a Series
The result will be a DataFrame with the same index as the input Series, and
with one column whose name is the original name of the Series (only if no other
column name provided).

```python
ser = pd.Series(range(3), index=list("abc"), name="ser")
pd.DataFrame(ser)
```


### From a list of namedtuples
The field names of the first ``namedtuple`` in the list determine the columns
of the `DataFrame`. The remaining namedtuples (or tuples) are simply unpacked
and their values are fed into the rows of the `DataFrame`. If any of those
tuples is shorter than the first ``namedtuple`` then the later columns in the
corresponding row are marked as missing values. If any are longer than the
first ``namedtuple``, a ``ValueError`` is raised.

```python
from collections import namedtuple

Point = namedtuple("Point", "x y")

pd.DataFrame([Point(0, 0), Point(0, 3), (2, 3)])

Point3D = namedtuple("Point3D", "x y z")

pd.DataFrame([Point3D(0, 0, 0), Point3D(0, 3, 5), Point(2, 3)])
```


### From a list of dataclasses
Data Classes as introduced in `PEP557](https://www.python.org/dev/peps/pep-0557)_,
can be passed into the DataFrame constructor.
Passing a list of dataclasses is equivalent to passing a list of dictionaries.

Please be aware, that all values in the list should be dataclasses, mixing
types in the list would result in a ``TypeError``.

```python
from dataclasses import make_dataclass

Point = make_dataclass("Point", [("x", int), ("y", int)])

pd.DataFrame([Point(0, 0), Point(0, 3), Point(2, 3)])
```
**Missing data**

To construct a DataFrame with missing data, we use ``np.nan`` to
represent missing values. Alternatively, you may pass a ``numpy.MaskedArray``
as the data argument to the DataFrame constructor, and its masked entries will
be considered missing. See `Missing data <missing_data>` for more.

### Alternate constructors


**DataFrame.from_dict**

`DataFrame.from_dict` takes a dict of dicts or a dict of array-like sequences
and returns a DataFrame. It operates like the `DataFrame` constructor except
for the ``orient`` parameter which is ``'columns'`` by default, but which can be
set to ``'index'`` in order to use the dict keys as row labels.


```python
pd.DataFrame.from_dict(dict([("A", [1, 2, 3]), ("B", [4, 5, 6])]))
```
If you pass ``orient='index'``, the keys will be the row labels. In this
case, you can also pass the desired column names:

```python
pd.DataFrame.from_dict(
    dict([("A", [1, 2, 3]), ("B", [4, 5, 6])]),
    orient="index",
    columns=["one", "two", "three"],
)
```


**DataFrame.from_records**

`DataFrame.from_records` takes a list of tuples or an ndarray with structured
dtype. It works analogously to the normal `DataFrame` constructor, except that
the resulting DataFrame index may be a specific field of the structured
dtype.

```python
data
pd.DataFrame.from_records(data, index="C")
```


### Column selection, addition, deletion
You can treat a `DataFrame` semantically like a dict of like-indexed `Series`
objects. Getting, setting, and deleting columns works with the same syntax as
the analogous dict operations:

```python
df["one"]
df["three"] = df["one"] * df["two"]
df["flag"] = df["one"] > 2
df
```
Columns can be deleted or popped like with a dict:

```python
del df["two"]
three = df.pop("three")
df
```
When inserting a scalar value, it will naturally be propagated to fill the
column:

```python
df["foo"] = "bar"
df
```
When inserting a `Series` that does not have the same index as the `DataFrame`, it
will be conformed to the DataFrame's index:

```python
df["one_trunc"] = df["one"][:2]
df
```
You can insert raw ndarrays but their length must match the length of the
DataFrame's index.

By default, columns get inserted at the end. `DataFrame.insert`
inserts at a particular location in the columns:

```python
df.insert(1, "bar", df["one"])
df
```


### Assigning new columns in method chains
Inspired by `dplyr's
<https://dplyr.tidyverse.org/reference/mutate.html>`__
``mutate`` verb, DataFrame has an `~pandas.DataFrame.assign`
method that allows you to easily create new columns that are potentially
derived from existing columns.

```python
iris = pd.read_csv("data/iris.data")
iris.head()
iris.assign(sepal_ratio=iris["SepalWidth"] / iris["SepalLength"]).head()
```
In the example above, we inserted a precomputed value. We can also pass in
a function of one argument to be evaluated on the DataFrame being assigned to.

```python
iris.assign(sepal_ratio=lambda x: (x["SepalWidth"] / x["SepalLength"])).head()
```
or, using `pandas.col`:

```python
iris.assign(sepal_ratio=pd.col("SepalWidth") / pd.col("SepalLength")).head()
```
`~pandas.DataFrame.assign` **always** returns a copy of the data, leaving the original
DataFrame untouched.

Passing a callable, as opposed to an actual value to be inserted, is
useful when you don't have a reference to the DataFrame at hand. This is
common when using `~pandas.DataFrame.assign` in a chain of operations. For example,
we can limit the DataFrame to just those observations with a Sepal Length
greater than 5, calculate the ratio, and plot:

```python
@savefig basics_assign.png
(
    iris.query("SepalLength > 5")
    .assign(
        SepalRatio=lambda x: x.SepalWidth / x.SepalLength,
        PetalRatio=lambda x: x.PetalWidth / x.PetalLength,
    )
    .plot(kind="scatter", x="SepalRatio", y="PetalRatio")
)
```
Since a function is passed in, the function is computed on the DataFrame
being assigned to. Importantly, this is the DataFrame that's been filtered
to those rows with sepal length greater than 5. The filtering happens first,
and then the ratio calculations. This is an example where we didn't
have a reference to the *filtered* DataFrame available.

The function signature for `~pandas.DataFrame.assign` is simply ``**kwargs``. The keys
are the column names for the new fields, and the values are either a value
to be inserted (for example, a `Series` or NumPy array), or a function
of one argument to be called on the `DataFrame`. A *copy* of the original
`DataFrame` is returned, with the new values inserted.

The order of ``**kwargs`` is preserved. This allows
for *dependent* assignment, where an expression later in ``**kwargs`` can refer
to a column created earlier in the same `~DataFrame.assign`.

```python
dfa = pd.DataFrame({"A": [1, 2, 3], "B": [4, 5, 6]})
dfa.assign(C=lambda x: x["A"] + x["B"], D=lambda x: x["A"] + x["C"])
```
In the second expression, ``x['C']`` will refer to the newly created column,
that's equal to ``dfa['A'] + dfa['B']``.


### Indexing / selection
The basics of indexing are as follows:


    :header: "Operation", "Syntax", "Result"
    :widths: 30, 20, 10

    Select column, ``df[col]``, Series
    Select row by label, ``df.loc[label]``, Series
    Select row by integer location, ``df.iloc[loc]``, Series
    Slice rows, ``df[5:10]``, DataFrame
    Select rows by boolean vector, ``df[bool_vec]``, DataFrame

Row selection, for example, returns a `Series` whose index is the columns of the
`DataFrame`:

```python
df.loc["b"]
df.iloc[2]
```
For a more exhaustive treatment of sophisticated label-based indexing and
slicing, see the `section on indexing <indexing>`. We will address the
fundamentals of reindexing / conforming to new sets of labels in the
`section on reindexing <basics.reindexing>`.



### Data alignment and arithmetic
Data alignment between `DataFrame` objects automatically align on **both the
columns and the index (row labels)**. Again, the resulting object will have the
union of the column and row labels.

```python
df = pd.DataFrame(np.random.randn(10, 4), columns=["A", "B", "C", "D"])
df2 = pd.DataFrame(np.random.randn(7, 3), columns=["A", "B", "C"])
df + df2
```
When doing an operation between `DataFrame` and `Series`, the default behavior is
to align the `Series` **index** on the `DataFrame` **columns**, thus `broadcasting
<https://numpy.org/doc/stable/user/basics.broadcasting.html>`__
row-wise. For example:

```python
df - df.iloc[0]
```
For explicit control over the matching and broadcasting behavior, see the
section on `flexible binary operations <basics.binop>[.

Arithmetic operations with scalars operate element-wise:

```python
df * 5 + 2
1 / df
df ** 4
```


Boolean operators operate element-wise as well:

```python
df1 = pd.DataFrame({"a": [1, 0, 1], "b": [0, 1, 1]}, dtype=bool)
df2 = pd.DataFrame({"a": [0, 1, 1], "b": [1, 1, 0]}, dtype=bool)
df1 & df2
df1 | df2
df1 ^ df2
-df1
```
### Transposing
To transpose, access the ``T`` attribute or `DataFrame.transpose`,
similar to an ndarray:

```python
# only show the first 5 rows
df[:5].T
```


### DataFrame interoperability with NumPy functions
Most NumPy functions can be called directly on `Series` and `DataFrame`.

```python
np.exp(df)
np.asarray(df)
```
`DataFrame` is not intended to be a drop-in replacement for ndarray as its
indexing semantics and data model are quite different in places from an n-dimensional
array.

`Series` implements ``__array_ufunc__``, which allows it to work with NumPy's
`universal functions](https://numpy.org/doc/stable/reference/ufuncs.html).

The ufunc is applied to the underlying array in a [Series`.

```python
ser = pd.Series([1, 2, 3, 4])
np.exp(ser)
```
When multiple `Series` are passed to a ufunc, they are aligned before
performing the operation.

Like other parts of the library, pandas will automatically align labeled inputs
as part of a ufunc with multiple inputs. For example, using `numpy.remainder`
on two `Series` with differently ordered labels will align before the operation.

```python
ser1 = pd.Series([1, 2, 3], index=["a", "b", "c"])
ser2 = pd.Series([1, 3, 5], index=["b", "a", "c"])
ser1
ser2
np.remainder(ser1, ser2)
```
As usual, the union of the two indices is taken, and non-overlapping values are filled
with missing values.

```python
ser3 = pd.Series([2, 4, 6], index=["b", "c", "d"])
ser3
np.remainder(ser1, ser3)
```
When a binary ufunc is applied to a `Series` and `Index`, the `Series`
implementation takes precedence and a `Series` is returned.

```python
ser = pd.Series([1, 2, 3])
idx = pd.Index([4, 5, 6])

np.maximum(ser, idx)
```
NumPy ufuncs are safe to apply to `Series` backed by non-ndarray arrays,
for example `arrays.SparseArray` (see `sparse.calculation`). If possible,
the ufunc is applied without converting the underlying data to an ndarray.

### Console display
A very large `DataFrame` will be truncated to display them in the console.
You can also get a summary using `~pandas.DataFrame.info`.
(The **baseball** dataset is from the **plyr** R package):

```python
:suppress:

# force a summary to be printed
pd.set_option("display.max_rows", 5)
```
```python
baseball = pd.read_csv("data/baseball.csv")
print(baseball)
baseball.info()
```
```python
:suppress:
:okwarning:

# restore GlobalPrintConfig
pd.reset_option(r"^display\.")
```
However, using `DataFrame.to_string` will return a string representation of the
`DataFrame` in tabular form, though it won't always fit the console width:

```python
print(baseball.iloc[-20:, :12].to_string())
```
Wide DataFrames will be printed across multiple rows by
default:

```python
pd.DataFrame(np.random.randn(3, 12))
```
You can change how much to print on a single row by setting the ``display.width``
option:

```python
pd.set_option("display.width", 40)  # default is 80

pd.DataFrame(np.random.randn(3, 12))
```
You can adjust the max width of the individual columns by setting ``display.max_colwidth``

```python
datafile = {
    "filename": ["filename_01", "filename_02"],
    "path": [
        "media/user_name/storage/folder_01/filename_01",
        "media/user_name/storage/folder_02/filename_02",
    ],
}

pd.set_option("display.max_colwidth", 30)
pd.DataFrame(datafile)

pd.set_option("display.max_colwidth", 100)
pd.DataFrame(datafile)
```
```python
:suppress:

pd.reset_option("display.width")
pd.reset_option("display.max_colwidth")
```
You can also disable this feature via the ``expand_frame_repr`` option.
This will print the table in one block.

### DataFrame column attribute access and IPython completion
If a `DataFrame` column label is a valid Python variable name, the column can be
accessed like an attribute:

```python
df = pd.DataFrame({"foo1": np.random.randn(5), "foo2": np.random.randn(5)})
df
df.foo1
```
The columns are also connected to the `IPython](https://ipython.org)_
completion mechanism so they can be tab-completed:



    In [5]: df.foo<TAB>  # noqa: E225, E999
    df.foo1  df.foo2

---

# 
#  Essential basic functionality
Here we discuss a lot of the essential functionality common to the pandas data
structures. To begin, let's create some example objects like we did in
the `10 minutes to pandas <10min>` section:

```python
index = pd.date_range("1/1/2000", periods=8)
s = pd.Series(np.random.randn(5), index=["a", "b", "c", "d", "e"])
df = pd.DataFrame(np.random.randn(8, 3), index=index, columns=["A", "B", "C"])
```


## Head and tail
To view a small sample of a Series or DataFrame object, use the
`~DataFrame.head` and `~DataFrame.tail` methods. The default number
of elements to display is five, but you may pass a custom number.

```python
long_series = pd.Series(np.random.randn(1000))
long_series.head()
long_series.tail(3)
```


## Attributes and underlying data
pandas objects have a number of attributes enabling you to access the metadata.

* **shape**: gives the axis dimensions of the object, consistent with ndarray
* Axis labels
    * **Series**: *index* (only axis)
    * **DataFrame**: *index* (rows) and *columns*

Note, **these attributes can be safely assigned to**!

```python
df[:2]
df.columns = [x.lower() for x in df.columns]
df
```
pandas objects (`Index`, `Series`, `DataFrame`) can be
thought of as containers for arrays, which hold the actual data and do the
actual computation. For many types, the underlying array is a
`numpy.ndarray`. However, pandas and 3rd party libraries may *extend*
NumPy's type system to add support for custom arrays
(see `basics.dtypes`).

To get the actual data inside a `Index` or `Series`, use
the ``.array`` property.

```python
s.array
s.index.array
```
`~Series.array` will always be an `~pandas.api.extensions.ExtensionArray`.
The exact details of what an `~pandas.api.extensions.ExtensionArray` is and why pandas uses them are a bit
beyond the scope of this introduction. See `basics.dtypes` for more.

If you know you need a NumPy array, use `~Series.to_numpy`
or `numpy.asarray`.

```python
s.to_numpy()
np.asarray(s)
```
When the Series or Index is backed by
an `~pandas.api.extensions.ExtensionArray`, `~Series.to_numpy`
may involve copying data and coercing values. See `basics.dtypes` for more.

`~Series.to_numpy` gives some control over the ``dtype`` of the
resulting `numpy.ndarray`. For example, consider datetimes with timezones.
NumPy doesn't have a dtype to represent timezone-aware datetimes, so there
are two possibly useful representations:

1. An object-dtype `numpy.ndarray` with `Timestamp` objects, each
   with the correct ``tz``.
2. A ``datetime64[ns]`` -dtype `numpy.ndarray`, where the values have
   been converted to UTC and the timezone discarded.

Timezones may be preserved with ``dtype=object``:

```python
ser = pd.Series(pd.date_range("2000", periods=2, tz="CET"))
ser.to_numpy(dtype=object)
```
Or thrown away with ``dtype='datetime64[ns]'``:

```python
ser.to_numpy(dtype="datetime64[ns]")
```
Getting the "raw data" inside a `DataFrame` is possibly a bit more
complex. When your ``DataFrame`` only has a single data type for all the
columns, `DataFrame.to_numpy` will return the underlying data:

```python
df.to_numpy()
```
If a DataFrame contains homogeneously-typed data, the ndarray can
actually be modified in-place, and the changes will be reflected in the data
structure. For heterogeneous data (e.g. some of the DataFrame's columns are not
all the same dtype), this will not be the case. The values attribute itself,
unlike the axis labels, cannot be assigned to.

> **note.capitalize():**
    When working with heterogeneous data, the dtype of the resulting ndarray
    will be chosen to accommodate all of the data involved. For example, if
    strings are involved, the result will be of object dtype. If there are only
    floats and integers, the resulting array will be of float dtype.

In the past, pandas recommended `Series.values` or `DataFrame.values`
for extracting the data from a Series or DataFrame. You'll still find references
to these in old code bases and online. Going forward, we recommend avoiding
``.values`` and using ``.array`` or ``.to_numpy()``. ``.values`` has the following
drawbacks:

1. When your Series contains an `extension type <extending.extension-types>`, it's
   unclear whether `Series.values` returns a NumPy array or the extension array.
   `Series.array` will always return an `~pandas.api.extensions.ExtensionArray`, and will never
   copy data. `Series.to_numpy` will always return a NumPy array,
   potentially at the cost of copying / coercing values.
2. When your DataFrame contains a mixture of data types, `DataFrame.values` may
   involve copying data and coercing values to a common dtype, a relatively expensive
   operation. `DataFrame.to_numpy`, being a method, makes it clearer that the
   returned NumPy array may not be a view on the same data in the DataFrame.



## Accelerated operations
pandas has support for accelerating certain types of binary numerical and boolean operations using
the ``numexpr`` library and the ``bottleneck`` libraries.

These libraries are especially useful when dealing with large data sets, and provide large
speedups. ``numexpr`` uses smart chunking, caching, and multiple cores. ``bottleneck`` is
a set of specialized cython routines that are especially fast when dealing with arrays that have
``nans``.

You are highly encouraged to install both libraries. See the section
`Recommended Dependencies <install.recommended_dependencies>` for more installation info.

These are both enabled to be used by default, you can control this by setting the options:

```python
pd.set_option("compute.use_bottleneck", False)
pd.set_option("compute.use_numexpr", False)
```


## Flexible binary operations
With binary operations between pandas data structures, there are two key points
of interest:

* Broadcasting behavior between higher- (e.g. DataFrame) and
  lower-dimensional (e.g. Series) objects.
* Missing data in computations.

We will demonstrate how to manage these issues independently, though they can
be handled simultaneously.

### Matching / broadcasting behavior
DataFrame has the methods `~DataFrame.add`, `~DataFrame.sub`,
`~DataFrame.mul`, `~DataFrame.div` and related functions
`~DataFrame.radd`, `~DataFrame.rsub`, ...
for carrying out binary operations. For broadcasting behavior,
Series input is of primary interest. Using these functions, you can use to
either match on the *index* or *columns* via the **axis** keyword:

```python
df = pd.DataFrame(
    {
        "one": pd.Series(np.random.randn(3), index=["a", "b", "c"]),
        "two": pd.Series(np.random.randn(4), index=["a", "b", "c", "d"]),
        "three": pd.Series(np.random.randn(3), index=["b", "c", "d"]),
    }
)
df
row = df.iloc[1]
column = df["two"]

df.sub(row, axis="columns")
df.sub(row, axis=1)

df.sub(column, axis="index")
df.sub(column, axis=0)
```
Furthermore you can align a level of a MultiIndexed DataFrame with a Series.

```python
dfmi = df.copy()
dfmi.index = pd.MultiIndex.from_tuples(
    [(1, "a"), (1, "b"), (1, "c"), (2, "a")], names=["first", "second"]
)
dfmi.sub(column, axis=0, level="second")
```
Series and Index also support the `divmod` builtin. This function takes
the floor division and modulo operation at the same time returning a two-tuple
of the same type as the left hand side. For example:

```python
s = pd.Series(np.arange(10))
s
div, rem = divmod(s, 3)
div
rem

idx = pd.Index(np.arange(10))
idx
div, rem = divmod(idx, 3)
div
rem
```
We can also do elementwise `divmod`:

```python
div, rem = divmod(s, [2, 2, 3, 3, 4, 4, 5, 5, 6, 6])
div
rem
```
### Missing data / operations with fill values
In Series and DataFrame, the arithmetic functions have the option of inputting
a *fill_value*, namely a value to substitute when at most one of the values at
a location are missing. For example, when adding two DataFrame objects, you may
wish to treat NaN as 0 unless both DataFrames are missing that value, in which
case the result will be NaN (you can later replace NaN with some other value
using ``fillna`` if you wish).

```python
df2 = df.copy()
df2.loc["a", "three"] = 1.0
df
df2
df + df2
df.add(df2, fill_value=0)
```


### Flexible comparisons
Series and DataFrame have the binary comparison methods ``eq``, ``ne``, ``lt``, ``gt``,
``le``, and ``ge`` whose behavior is analogous to the binary
arithmetic operations described above:

```python
df.gt(df2)
df2.ne(df)
```
These operations produce a pandas object of the same type as the left-hand-side
input that is of dtype ``bool``. These ``boolean`` objects can be used in
indexing operations, see the section on `Boolean indexing<indexing.boolean>`.



### Boolean reductions
You can apply the reductions: `~DataFrame.empty`, `~DataFrame.any`,
`~DataFrame.all`.

```python
(df > 0).all()
(df > 0).any()
```
You can reduce to a final boolean value.

```python
(df > 0).any().any()
```
You can test if a pandas object is empty, via the `~DataFrame.empty` property.

```python
df.empty
pd.DataFrame(columns=list("ABC")).empty
```
> **warning.capitalize():**
   Asserting the truthiness of a pandas object will raise an error, as the testing of the emptiness
   or values is ambiguous.

   ```python
:okexcept:

   if df:
       print(True)


   :okexcept:

   df and df2

See `gotchas<gotchas.truth>` for a more detailed discussion.
```


### Comparing if objects are equivalent
Often you may find that there is more than one way to compute the same
result.  As a simple example, consider ``df + df`` and ``df * 2``. To test
that these two computations produce the same result, given the tools
shown above, you might imagine using ``(df + df == df * 2).all()``. But in
fact, this expression is False:

```python
df + df == df * 2
(df + df == df * 2).all()
```
Notice that the boolean DataFrame ``df + df == df * 2`` contains some False values!
This is because NaNs do not compare as equals:

```python
np.nan == np.nan
```
So, NDFrames (such as Series and DataFrames)
have an `~DataFrame.equals` method for testing equality, with NaNs in
corresponding locations treated as equal.

```python
(df + df).equals(df * 2)
```
Note that the Series or DataFrame index needs to be in the same order for
equality to be True:

```python
df1 = pd.DataFrame({"col": ["foo", 0, np.nan]})
df2 = pd.DataFrame({"col": [np.nan, 0, "foo"]}, index=[2, 1, 0])
df1.equals(df2)
df1.equals(df2.sort_index())
```
### Comparing array-like objects
You can conveniently perform element-wise comparisons when comparing a pandas
data structure with a scalar value:

```python
pd.Series(["foo", "bar", "baz"]) == "foo"
pd.Index(["foo", "bar", "baz"]) == "foo"
```
pandas also handles element-wise comparisons between different array-like
objects of the same length:

```python
pd.Series(["foo", "bar", "baz"]) == pd.Index(["foo", "bar", "qux"])
pd.Series(["foo", "bar", "baz"]) == np.array(["foo", "bar", "qux"])
```
Trying to compare ``Index`` or ``Series`` objects of different lengths will
raise a ValueError:

```python
:okexcept:

 pd.Series(['foo', 'bar', 'baz']) == pd.Series(['foo', 'bar'])

 pd.Series(['foo', 'bar', 'baz']) == pd.Series(['foo'])
```
### Combining overlapping data sets
A problem occasionally arising is the combination of two similar data sets
where values in one are preferred over the other. An example would be two data
series representing a particular economic indicator where one is considered to
be of "higher quality". However, the lower quality series might extend further
back in history or have more complete data coverage. As such, we would like to
combine two DataFrame objects where missing values in one DataFrame are
conditionally filled with like-labeled values from the other DataFrame. The
function implementing this operation is `~DataFrame.combine_first`,
which we illustrate:

```python
df1 = pd.DataFrame(
    {"A": [1.0, np.nan, 3.0, 5.0, np.nan], "B": [np.nan, 2.0, 3.0, np.nan, 6.0]}
)
df2 = pd.DataFrame(
    {
        "A": [5.0, 2.0, 4.0, np.nan, 3.0, 7.0],
        "B": [np.nan, np.nan, 3.0, 4.0, 6.0, 8.0],
    }
)
df1
df2
df1.combine_first(df2)
```
### General DataFrame combine
The `~DataFrame.combine_first` method above calls the more general
`DataFrame.combine`. This method takes another DataFrame
and a combiner function, aligns the input DataFrame and then passes the combiner
function pairs of Series (i.e., columns whose names are the same).

So, for instance, to reproduce `~DataFrame.combine_first` as above:

```python
def combiner(x, y):
    return np.where(pd.isna(x), y, x)


df1.combine(df2, combiner)
```


## Descriptive statistics
There exists a large number of methods for computing descriptive statistics and
other related operations on `Series <api.series.stats>`, DataFrame
. Most of these
are aggregations (hence producing a lower-dimensional result) like
`~DataFrame.sum`, `~DataFrame.mean`, and `~DataFrame.quantile`,
but some of them, like `~DataFrame.cumsum` and `~DataFrame.cumprod`,
produce an object of the same size. Generally speaking, these methods take an
**axis** argument, just like *ndarray.{sum, std, ...}*, but the axis can be
specified by name or integer:

* **Series**: no axis argument needed
* **DataFrame**: "index" (axis=0, default), "columns" (axis=1)

For example:

```python
df
df.mean(axis=0)
df.mean(axis=1)
```
All such methods have a ``skipna`` option signaling whether to exclude missing
data (``True`` by default):

```python
df.sum(axis=0, skipna=False)
df.sum(axis=1, skipna=True)
```
Combined with the broadcasting / arithmetic behavior, one can describe various
statistical procedures, like standardization (rendering data zero mean and
standard deviation of 1), very concisely:

```python
ts_stand = (df - df.mean()) / df.std()
ts_stand.std()
xs_stand = df.sub(df.mean(axis=1), axis=0).div(df.std(axis=1), axis=0)
xs_stand.std(axis=1)
```
Note that methods like `~DataFrame.cumsum` and `~DataFrame.cumprod`
preserve the location of ``NaN`` values. This is somewhat different from
`~DataFrame.expanding` and `~DataFrame.rolling` since ``NaN`` behavior
is furthermore dictated by a ``min_periods`` parameter.

```python
df.cumsum()
```
Here is a quick reference summary table of common functions. Each also takes an
optional ``level`` parameter which applies only if the object has a
`hierarchical index<advanced.hierarchical>`.


    :header: "Function", "Description"
    :widths: 20, 80

    ``count``, Number of non-NA observations
    ``sum``, Sum of values
    ``mean``, Mean of values
    ``median``, Arithmetic median of values
    ``min``, Minimum
    ``max``, Maximum
    ``mode``, Mode
    ``abs``, Absolute Value
    ``prod``, Product of values
    ``std``, Bessel-corrected sample standard deviation
    ``var``, Unbiased variance
    ``sem``, Standard error of the mean
    ``skew``, Sample skewness (3rd moment)
    ``kurt``, Sample kurtosis (4th moment)
    ``quantile``, Sample quantile (value at %)
    ``cumsum``, Cumulative sum
    ``cumprod``, Cumulative product
    ``cummax``, Cumulative maximum
    ``cummin``, Cumulative minimum

Note that by chance some NumPy methods, like ``mean``, ``std``, and ``sum``,
will exclude NAs on Series input by default:

```python
np.mean(df["one"])
np.mean(df["one"].to_numpy())
```
`Series.nunique` will return the number of unique non-NA values in a
Series:

```python
series = pd.Series(np.random.randn(500))
series[20:500] = np.nan
series[10:20] = 5
series.nunique()
```


### Summarizing data: describe
There is a convenient `~DataFrame.describe` function which computes a variety of summary
statistics about a Series or the columns of a DataFrame (excluding NAs of
course):

```python
series = pd.Series(np.random.randn(1000))
series[::2] = np.nan
series.describe()
frame = pd.DataFrame(np.random.randn(1000, 5), columns=["a", "b", "c", "d", "e"])
frame.iloc[::2] = np.nan
frame.describe()
```
You can select specific percentiles to include in the output:

```python
series.describe(percentiles=[0.05, 0.25, 0.75, 0.95])
```
By default, the median is always included.

For a non-numerical Series object, `~Series.describe` will give a simple
summary of the number of unique values and most frequently occurring values:

```python
s = pd.Series(["a", "a", "b", "b", "a", "a", np.nan, "c", "d", "a"])
s.describe()
```
Note that on a mixed-type DataFrame object, `~DataFrame.describe` will
restrict the summary to include only numerical columns or, if none are, only
categorical columns:

```python
frame = pd.DataFrame({"a": ["Yes", "Yes", "No", "No"], "b": range(4)})
frame.describe()
```
This behavior can be controlled by providing a list of types as ``include``/``exclude``
arguments. The special value ``all`` can also be used:

```python
frame.describe(include=["str"])
frame.describe(include=["number"])
frame.describe(include="all")
```
That feature relies on `select_dtypes <basics.selectdtypes>`. Refer to
there for details about accepted inputs.



### Index of min/max values
The `~DataFrame.idxmin` and `~DataFrame.idxmax` functions on Series
and DataFrame compute the index labels with the minimum and maximum
corresponding values:

```python
s1 = pd.Series(np.random.randn(5))
s1
s1.idxmin(), s1.idxmax()

df1 = pd.DataFrame(np.random.randn(5, 3), columns=["A", "B", "C"])
df1
df1.idxmin(axis=0)
df1.idxmax(axis=1)
```
When there are multiple rows (or columns) matching the minimum or maximum
value, `~DataFrame.idxmin` and `~DataFrame.idxmax` return the first
matching index:

```python
df3 = pd.DataFrame([2, 1, 1, 3, np.nan], columns=["A"], index=list("edcba"))
df3
df3["A"].idxmin()
```
> **note.capitalize():**
   ``idxmin`` and ``idxmax`` are called ``argmin`` and ``argmax`` in NumPy.



### Value counts (histogramming) / mode
The `~Series.value_counts` Series method computes a histogram
of a 1D array of values. It can also be used as a function on regular arrays:

```python
data = np.random.randint(0, 7, size=50)
data
s = pd.Series(data)
s.value_counts()
```
The `~DataFrame.value_counts` method can be used to count combinations across multiple columns.
By default all columns are used but a subset can be selected using the ``subset`` argument.

```python
data = {"a": [1, 2, 3, 4], "b": ["x", "x", "y", "y"]}
frame = pd.DataFrame(data)
frame.value_counts()
```
Similarly, you can get the most frequently occurring value(s), i.e. the mode, of the values in a Series or DataFrame:

```python
s5 = pd.Series([1, 1, 3, 3, 3, 5, 5, 7, 7, 7])
s5.mode()
df5 = pd.DataFrame(
    {
        "A": np.random.randint(0, 7, size=50),
        "B": np.random.randint(-10, 15, size=50),
    }
)
df5.mode()
```
### Discretization and quantiling
Continuous values can be discretized using the `cut` (bins based on values)
and `qcut` (bins based on sample quantiles) functions:

```python
arr = np.random.randn(20)
factor = pd.cut(arr, 4)
factor

factor = pd.cut(arr, [-5, -1, 0, 1, 5])
factor
```
`qcut` computes sample quantiles. For example, we could slice up some
normally distributed data into equal-size quartiles like so:

```python
arr = np.random.randn(30)
factor = pd.qcut(arr, [0, 0.25, 0.5, 0.75, 1])
factor
```
We can also pass infinite values to define the bins:

```python
arr = np.random.randn(20)
factor = pd.cut(arr, [-np.inf, 0, np.inf])
factor
```


## Function application
To apply your own or another library's functions to pandas objects,
you should be aware of the three methods below. The appropriate
method to use depends on whether your function expects to operate
on an entire ``DataFrame`` or ``Series``, row- or column-wise, or elementwise.

1. `Tablewise Function Application`_: `~DataFrame.pipe`
2. `Row or Column-wise Function Application`_: `~DataFrame.apply`
3. `Aggregation API`_: `~DataFrame.agg` and `~DataFrame.transform`
4. `Applying Elementwise Functions`_: `~DataFrame.map`



### Tablewise function application
``DataFrames`` and ``Series`` can be passed into functions.
However, if the function needs to be called in a chain, consider using the `~DataFrame.pipe` method.

First some setup:

```python
def extract_city_name(df):
    """
    Chicago, IL -> Chicago for city_name column
    """
    df["city_name"] = df["city_and_code"].str.split(",").str.get(0)
    return df


def add_country_name(df, country_name=None):
    """
    Chicago -> Chicago-US for city_name column
    """
    col = "city_name"
    df["city_and_country"] = df[col] + country_name
    return df


df_p = pd.DataFrame({"city_and_code": ["Chicago, IL"]})
```
``extract_city_name`` and ``add_country_name`` are functions taking and returning ``DataFrames``.

Now compare the following:

```python
add_country_name(extract_city_name(df_p), country_name="US")
```
Is equivalent to:

```python
df_p.pipe(extract_city_name).pipe(add_country_name, country_name="US")
```
pandas encourages the second style, which is known as method chaining.
``pipe`` makes it easy to use your own or another library's functions
in method chains, alongside pandas' methods.

In the example above, the functions ``extract_city_name`` and ``add_country_name`` each expected a ``DataFrame`` as the first positional argument.
What if the function you wish to apply takes its data as, say, the second argument?
In this case, provide ``pipe`` with a tuple of ``(callable, data_keyword)``.
``.pipe`` will route the ``DataFrame`` to the argument specified in the tuple.

For example, we can fit a regression using statsmodels. Their API expects a formula first and a ``DataFrame`` as the second argument, ``data``. We pass in the function, keyword pair ``(sm.ols, 'data')`` to ``pipe``:



   In [147]: import statsmodels.formula.api as sm

   In [148]: bb = pd.read_csv("data/baseball.csv", index_col="id")

   In [149]: (
      .....:     bb.query("h > 0")
      .....:     .assign(ln_h=lambda df: np.log(df.h))
      .....:     .pipe((sm.ols, "data"), "hr ~ ln_h + year + g + C(lg)")
      .....:     .fit()
      .....:     .summary()
      .....: )
      .....:
   Out[149]:
   <class 'statsmodels.iolib.summary.Summary'>
   """
                              OLS Regression Results
   ==============================================================================
   Dep. Variable:                     hr   R-squared:                       0.685
   Model:                            OLS   Adj. R-squared:                  0.665
   Method:                 Least Squares   F-statistic:                     34.28
   Date:                Tue, 22 Nov 2022   Prob (F-statistic):           3.48e-15
   Time:                        05:34:17   Log-Likelihood:                -205.92
   No. Observations:                  68   AIC:                             421.8
   Df Residuals:                      63   BIC:                             432.9
   Df Model:                           4
   Covariance Type:            nonrobust
   ===============================================================================
                     coef    std err          t      P>|t|      [0.025      0.975]
   -------------------------------------------------------------------------------
   Intercept   -8484.7720   4664.146     -1.819      0.074   -1.78e+04     835.780
   C(lg)[T.NL]    -2.2736      1.325     -1.716      0.091      -4.922       0.375
   ln_h           -1.3542      0.875     -1.547      0.127      -3.103       0.395
   year            4.2277      2.324      1.819      0.074      -0.417       8.872
   g               0.1841      0.029      6.258      0.000       0.125       0.243
   ==============================================================================
   Omnibus:                       10.875   Durbin-Watson:                   1.999
   Prob(Omnibus):                  0.004   Jarque-Bera (JB):               17.298
   Skew:                           0.537   Prob(JB):                     0.000175
   Kurtosis:                       5.225   Cond. No.                     1.49e+07
   ==============================================================================

   Notes:
   [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
   [2] The condition number is large, 1.49e+07. This might indicate that there are
   strong multicollinearity or other numerical problems.
   """

The pipe method is inspired by unix pipes and more recently dplyr_ and magrittr_, which
have introduced the popular ``(%>%)`` (read pipe) operator for R_.
The implementation of ``pipe`` here is quite clean and feels right at home in Python.
We encourage you to view the source code of `~DataFrame.pipe`.

.. _dplyr: https://github.com/tidyverse/dplyr
.. _magrittr: https://github.com/tidyverse/magrittr
.. _R: https://www.r-project.org


### Row or column-wise function application
Arbitrary functions can be applied along the axes of a DataFrame
using the `~DataFrame.apply` method, which, like the descriptive
statistics methods, takes an optional ``axis`` argument:

```python
df.apply(lambda x: np.mean(x))
df.apply(lambda x: np.mean(x), axis=1)
df.apply(lambda x: x.max() - x.min())
df.apply(np.cumsum)
df.apply(np.exp)
```
The `~DataFrame.apply` method will also dispatch on a string method name.

```python
df.apply("mean")
df.apply("mean", axis=1)
```
The return type of the function passed to `~DataFrame.apply` affects the
type of the final output from ``DataFrame.apply`` for the default behaviour:

* If the applied function returns a ``Series``, the final output is a ``DataFrame``.
  The columns match the index of the ``Series`` returned by the applied function.
* If the applied function returns any other type, the final output is a ``Series``.

This default behaviour can be overridden using the ``result_type``, which
accepts three options: ``reduce``, ``broadcast``, and ``expand``.
These will determine how list-likes return values expand (or not) to a ``DataFrame``.

`~DataFrame.apply` combined with some cleverness can be used to answer many questions
about a data set. For example, suppose we wanted to extract the date where the
maximum value for each column occurred:

```python
tsdf = pd.DataFrame(
    np.random.randn(1000, 3),
    columns=["A", "B", "C"],
    index=pd.date_range("1/1/2000", periods=1000),
)
tsdf.apply(lambda x: x.idxmax())
```
You may also pass additional arguments and keyword arguments to the `~DataFrame.apply`
method.

```python
def subtract_and_divide(x, sub, divide=1):
    return (x - sub) / divide

df_udf = pd.DataFrame(np.ones((2, 2)))
df_udf.apply(subtract_and_divide, args=(5,), divide=3)
```
Another useful feature is the ability to pass Series methods to carry out some
Series operation on each column or row:

```python
tsdf = pd.DataFrame(
    np.random.randn(10, 3),
    columns=["A", "B", "C"],
    index=pd.date_range("1/1/2000", periods=10),
)
tsdf.iloc[3:7] = np.nan
tsdf
tsdf.apply(pd.Series.interpolate)
```
Finally, `~DataFrame.apply` takes an argument ``raw`` which is False by default, which
converts each row or column into a Series before applying the function. When
set to True, the passed function will instead receive an ndarray object, which
has positive performance implications if you do not need the indexing
functionality.



### Aggregation API
The aggregation API allows one to express possibly multiple aggregation operations in a single concise way.
This API is similar across pandas objects, see `groupby API <groupby.aggregate>`, the
`window API <window.overview>`, and the `resample API <timeseries.aggregate>`.
The entry point for aggregation is `DataFrame.aggregate`, or the alias
`DataFrame.agg`.

We will use a similar starting frame from above:

```python
tsdf = pd.DataFrame(
    np.random.randn(10, 3),
    columns=["A", "B", "C"],
    index=pd.date_range("1/1/2000", periods=10),
)
tsdf.iloc[3:7] = np.nan
tsdf
```
Using a single function is equivalent to `~DataFrame.apply`. You can also
pass named methods as strings. These will return a ``Series`` of the aggregated
output:

```python
tsdf.agg(lambda x: np.sum(x))

tsdf.agg("sum")

# these are equivalent to a ``.sum()`` because we are aggregating
# on a single function
tsdf.sum()
```
Single aggregations on a ``Series`` this will return a scalar value:

```python
tsdf["A"].agg("sum")
```
Aggregating with multiple functions
+++++++++++++++++++++++++++++++++++

You can pass multiple aggregation arguments as a list.
The results of each of the passed functions will be a row in the resulting ``DataFrame``.
These are naturally named from the aggregation function.

```python
tsdf.agg(["sum"])
```
Multiple functions yield multiple rows:

```python
tsdf.agg(["sum", "mean"])
```
On a ``Series``, multiple functions return a ``Series``, indexed by the function names:

```python
tsdf["A"].agg(["sum", "mean"])
```
Passing a ``lambda`` function will yield a ``<lambda>`` named row:

```python
tsdf["A"].agg(["sum", lambda x: x.mean()])
```
Passing a named function will yield that name for the row:

```python
def mymean(x):
    return x.mean()


tsdf["A"].agg(["sum", mymean])
```
Aggregating with a dict
+++++++++++++++++++++++

Passing a dictionary of column names to a scalar or a list of scalars, to ``DataFrame.agg``
allows you to customize which functions are applied to which columns. Note that the results
are not in any particular order, you can use an ``OrderedDict`` instead to guarantee ordering.

```python
tsdf.agg({"A": "mean", "B": "sum"})
```
Passing a list-like will generate a ``DataFrame`` output. You will get a matrix-like output
of all of the aggregators. The output will consist of all unique functions. Those that are
not noted for a particular column will be ``NaN``:

```python
tsdf.agg({"A": ["mean", "min"], "B": "sum"})
```


Custom describe
+++++++++++++++

With ``.agg()`` it is possible to easily create a custom describe function, similar
to the built in `describe function <basics.describe>`.

```python
from functools import partial

q_25 = partial(pd.Series.quantile, q=0.25)
q_25.__name__ = "25%"
q_75 = partial(pd.Series.quantile, q=0.75)
q_75.__name__ = "75%"

tsdf.agg(["count", "mean", "std", "min", q_25, "median", q_75, "max"])
```


### Transform API
The `~DataFrame.transform` method returns an object that is indexed the same (same size)
as the original. This API allows you to provide *multiple* operations at the same
time rather than one-by-one. Its API is quite similar to the ``.agg`` API.

We create a frame similar to the one used in the above sections.

```python
tsdf = pd.DataFrame(
    np.random.randn(10, 3),
    columns=["A", "B", "C"],
    index=pd.date_range("1/1/2000", periods=10),
)
tsdf.iloc[3:7] = np.nan
tsdf
```
Transform the entire frame. ``.transform()`` allows input functions as: a NumPy function, a string
function name or a user defined function.

```python
:okwarning:

tsdf.transform(np.abs)
tsdf.transform("abs")
tsdf.transform(lambda x: x.abs())
```
Here `~DataFrame.transform` received a single function; this is equivalent to a `ufunc
<https://numpy.org/doc/stable/reference/ufuncs.html>`__ application.

```python
np.abs(tsdf)
```
Passing a single function to ``.transform()`` with a ``Series`` will yield a single ``Series`` in return.

```python
tsdf["A"].transform(np.abs)
```
Transform with multiple functions
+++++++++++++++++++++++++++++++++

Passing multiple functions will yield a column MultiIndexed DataFrame.
The first level will be the original frame column names; the second level
will be the names of the transforming functions.

```python
tsdf.transform([np.abs, lambda x: x + 1])
```
Passing multiple functions to a Series will yield a DataFrame. The
resulting column names will be the transforming functions.

```python
tsdf["A"].transform([np.abs, lambda x: x + 1])
```
Transforming with a dict
++++++++++++++++++++++++


Passing a dict of functions will allow selective transforming per column.

```python
tsdf.transform({"A": np.abs, "B": lambda x: x + 1})
```
Passing a dict of lists will generate a MultiIndexed DataFrame with these
selective transforms.

```python
:okwarning:

tsdf.transform({"A": np.abs, "B": [lambda x: x + 1, "sqrt"]})
```


### Applying elementwise functions
Since not all functions can be vectorized (accept NumPy arrays and return
another array or value), the methods `~DataFrame.map` on DataFrame
and analogously `~Series.map` on Series accept any Python function taking
a single value and returning a single value. For example:

```python
df4 = df.copy()
df4

def f(x):
    return len(str(x))

df4["one"].map(f)
df4.map(f)
```
`Series.map` has an additional feature; it can be used to easily
"link" or "map" values defined by a secondary series. This is closely related
to `merging/joining functionality <merging>`:

```python
s = pd.Series(
    ["six", "seven", "six", "seven", "six"], index=["a", "b", "c", "d", "e"]
)
t = pd.Series({"six": 6.0, "seven": 7.0})
s
s.map(t)
```


## Reindexing and altering labels
`~Series.reindex` is the fundamental data alignment method in pandas.
It is used to implement nearly all other features relying on label-alignment
functionality. To *reindex* means to conform the data to match a given set of
labels along a particular axis. This accomplishes several things:

* Reorders the existing data to match a new set of labels
* Inserts missing value (NA) markers in label locations where no data for
  that label existed
* If specified, **fill** data for missing labels using logic (highly relevant
  to working with time series data)

Here is a simple example:

```python
s = pd.Series(np.random.randn(5), index=["a", "b", "c", "d", "e"])
s
s.reindex(["e", "b", "f", "d"])
```
Here, the ``f`` label was not contained in the Series and hence appears as
``NaN`` in the result.

With a DataFrame, you can simultaneously reindex the index and columns:

```python
df
df.reindex(index=["c", "f", "b"], columns=["three", "two", "one"])
```
Note that the ``Index`` objects containing the actual axis labels can be
**shared** between objects. So if we have a Series and a DataFrame, the
following can be done:

```python
rs = s.reindex(df.index)
rs
rs.index is df.index
```
This means that the reindexed Series's index is the same Python object as the
DataFrame's index.

`DataFrame.reindex` also supports an "axis-style" calling convention,
where you specify a single ``labels`` argument and the ``axis`` it applies to.

```python
df.reindex(["c", "f", "b"], axis="index")
df.reindex(["three", "two", "one"], axis="columns")
```
> **seealso.capitalize():**
   `MultiIndex / Advanced Indexing <advanced>` is an even more concise way of
   doing reindexing.

> **note.capitalize():**
    When writing performance-sensitive code, there is a good reason to spend
    some time becoming a reindexing ninja: **many operations are faster on
    pre-aligned data**. Adding two unaligned DataFrames internally triggers a
    reindexing step. For exploratory analysis you will hardly notice the
    difference (because ``reindex`` has been heavily optimized), but when CPU
    cycles matter sprinkling a few explicit ``reindex`` calls here and there can
    have an impact.



### Reindexing to align with another object
You may wish to take an object and reindex its axes to be labeled the same as
another object. While the syntax for this is straightforward albeit verbose, it
is a common enough operation that the `~DataFrame.reindex_like` method is
available to make this simpler:

```python
df2 = df.reindex(["a", "b", "c"], columns=["one", "two"])
df3 = df2 - df2.mean()
df2
df3
df.reindex_like(df2)
```


### Aligning objects with each other with ``align``
The `~Series.align` method is the fastest way to simultaneously align two objects. It
supports a ``join`` argument (related to `joining and merging <merging>`):

  - ``join='outer'``: take the union of the indexes (default)
  - ``join='left'``: use the calling object's index
  - ``join='right'``: use the passed object's index
  - ``join='inner'``: intersect the indexes

It returns a tuple with both of the reindexed Series:

```python
s = pd.Series(np.random.randn(5), index=["a", "b", "c", "d", "e"])
s1 = s[:4]
s2 = s[1:]
s1.align(s2)
s1.align(s2, join="inner")
s1.align(s2, join="left")
```


For DataFrames, the join method will be applied to both the index and the
columns by default:

```python
df.align(df2, join="inner")
```
You can also pass an ``axis`` option to only align on the specified axis:

```python
df.align(df2, join="inner", axis=0)
```


If you pass a Series to `DataFrame.align`, you can choose to align both
objects either on the DataFrame's index or columns using the ``axis`` argument:

```python
df.align(df2.iloc[0], axis=1)
```


### Filling while reindexing
`~Series.reindex` takes an optional parameter ``method`` which is a
filling method chosen from the following table:


    :header: "Method", "Action"
    :widths: 30, 50

    ffill, Fill values forward
    bfill, Fill values backward
    nearest, Fill from the nearest index value

We illustrate these fill methods on a simple Series:

```python
rng = pd.date_range("1/3/2000", periods=8)
ts = pd.Series(np.random.randn(8), index=rng)
ts2 = ts.iloc[[0, 3, 6]]
ts
ts2

ts2.reindex(ts.index)
ts2.reindex(ts.index, method="ffill")
ts2.reindex(ts.index, method="bfill")
ts2.reindex(ts.index, method="nearest")
```
These methods require that the indexes are **ordered** increasing or
decreasing.

Note that the same result could have been achieved using
`ffill <missing_data.fillna>` (except for ``method='nearest'``) or
`interpolate <missing_data.interpolate>`:

```python
ts2.reindex(ts.index).ffill()
```
`~Series.reindex` will raise a ValueError if the index is not monotonically
increasing or decreasing. `~Series.fillna` and `~Series.interpolate`
will not perform any checks on the order of the index.



### Limits on filling while reindexing
The ``limit`` and ``tolerance`` arguments provide additional control over
filling while reindexing. Limit specifies the maximum count of consecutive
matches:

```python
ts2.reindex(ts.index, method="ffill", limit=1)
```
In contrast, tolerance specifies the maximum distance between the index and
indexer values:

```python
ts2.reindex(ts.index, method="ffill", tolerance="1 day")
```
Notice that when used on a ``DatetimeIndex``, ``TimedeltaIndex`` or
``PeriodIndex``, ``tolerance`` will coerced into a ``Timedelta`` if possible.
This allows you to specify tolerance with appropriate strings.



### Dropping labels from an axis
A method closely related to ``reindex`` is the `~DataFrame.drop` function.
It removes a set of labels from an axis:

```python
df
df.drop(["a", "d"], axis=0)
df.drop(["one"], axis=1)
```
Note that the following also works, but is a bit less obvious / clean:

```python
df.reindex(df.index.difference(["a", "d"]))
```


### Renaming / mapping labels
The `~DataFrame.rename` method allows you to relabel an axis based on some
mapping (a dict or Series) or an arbitrary function.

```python
s
s.rename(str.upper)
```
If you pass a function, it must return a value when called with any of the
labels (and must produce a set of unique values). A dict or
Series can also be used:

```python
df.rename(
    columns={"one": "foo", "two": "bar"},
    index={"a": "apple", "b": "banana", "d": "durian"},
)
```
If the mapping doesn't include a column/index label, it isn't renamed. Note that
extra labels in the mapping don't throw an error.

`DataFrame.rename` also supports an "axis-style" calling convention, where
you specify a single ``mapper`` and the ``axis`` to apply that mapping to.

```python
df.rename({"one": "foo", "two": "bar"}, axis="columns")
df.rename({"a": "apple", "b": "banana", "d": "durian"}, axis="index")
```
Finally, `~Series.rename` also accepts a scalar or list-like
for altering the ``Series.name`` attribute.

```python
s.rename("scalar-name")
```


The methods `DataFrame.rename_axis` and `Series.rename_axis`
allow specific names of a ``MultiIndex`` to be changed (as opposed to the
labels).

```python
df = pd.DataFrame(
    {"x": [1, 2, 3, 4, 5, 6], "y": [10, 20, 30, 40, 50, 60]},
    index=pd.MultiIndex.from_product(
        [["a", "b", "c"], [1, 2]], names=["let", "num"]
    ),
)
df
df.rename_axis(index={"let": "abc"})
df.rename_axis(index=str.upper)
```


## Iteration
The behavior of basic iteration over pandas objects depends on the type.
When iterating over a Series, it is regarded as array-like, and basic iteration
produces the values. DataFrames follow the dict-like convention of iterating
over the "keys" of the objects.

In short, basic iteration (``for i in object``) produces:

* **Series**: values
* **DataFrame**: column labels

Thus, for example, iterating over a DataFrame gives you the column names:

```python
df = pd.DataFrame(
    {"col1": np.random.randn(3), "col2": np.random.randn(3)}, index=["a", "b", "c"]
)

for col in df:
    print(col)
```
pandas objects also have the dict-like `~DataFrame.items` method to
iterate over the (key, value) pairs.

To iterate over the rows of a DataFrame, you can use the following methods:

* `~DataFrame.iterrows`: Iterate over the rows of a DataFrame as (index, Series) pairs.
  This converts the rows to Series objects, which can change the dtypes and has some
  performance implications.
* `~DataFrame.itertuples`: Iterate over the rows of a DataFrame
  as namedtuples of the values.  This is a lot faster than
  `~DataFrame.iterrows`, and is in most cases preferable to use
  to iterate over the values of a DataFrame.

> **warning.capitalize():**
  Iterating through pandas objects is generally **slow**. In many cases,
  iterating manually over the rows is not needed and can be avoided with
  one of the following approaches:

  * Look for a *vectorized* solution: many operations can be performed using
    built-in methods or NumPy functions, (boolean) indexing, ...

  * When you have a function that cannot work on the full DataFrame/Series
    at once, it is better to use `~DataFrame.apply` instead of iterating
    over the values. See the docs on `function application <basics.apply>`.

  * If you need to do iterative manipulations on the values but performance is
    important, consider writing the inner loop with cython or numba.
    See the `enhancing performance <enhancingperf>` section for some
    examples of this approach.

> **warning.capitalize():**
  You should **never modify** something you are iterating over.
  This is not guaranteed to work in all cases. Depending on the
  data types, the iterator returns a copy and not a view, and writing
  to it will have no effect!

  For example, in the following case setting the value has no effect:

  ```python
df = pd.DataFrame({"a": [1, 2, 3], "b": ["a", "b", "c"]})

for index, row in df.iterrows():
    row["a"] = 10

df
```
### items
Consistent with the dict-like interface, `~DataFrame.items` iterates
through key-value pairs:

* **Series**: (index, scalar value) pairs
* **DataFrame**: (column, Series) pairs

For example:

```python
for label, ser in df.items():
    print(label)
    print(ser)
```


### iterrows
`~DataFrame.iterrows` allows you to iterate through the rows of a
DataFrame as Series objects. It returns an iterator yielding each
index value along with a Series containing the data in each row:

```python
for row_index, row in df.iterrows():
    print(row_index, row, sep="\n")
```
> **note.capitalize():**
   Because `~DataFrame.iterrows` returns a Series for each row,
   it does **not** preserve dtypes across the rows (dtypes are
   preserved across columns for DataFrames). For example,

   ```python
df_orig = pd.DataFrame([[1, 1.5]], columns=["int", "float"])
   df_orig.dtypes
   row = next(df_orig.iterrows())[1]
   row

All values in ``row``, returned as a Series, are now upcasted
to floats, also the original integer value in column ``x``:



   row["int"].dtype
   df_orig["int"].dtype

To preserve dtypes while iterating over the rows, it is better
to use `~DataFrame.itertuples` which returns namedtuples of the values
and which is generally much faster than `~DataFrame.iterrows`.
```
For instance, a contrived way to transpose the DataFrame would be:

```python
df2 = pd.DataFrame({"x": [1, 2, 3], "y": [4, 5, 6]})
print(df2)
print(df2.T)

df2_t = pd.DataFrame({idx: values for idx, values in df2.iterrows()})
print(df2_t)
```
### itertuples
The `~DataFrame.itertuples` method will return an iterator
yielding a namedtuple for each row in the DataFrame. The first element
of the tuple will be the row's corresponding index value, while the
remaining values are the row values.

For instance:

```python
for row in df.itertuples():
    print(row)
```
This method does not convert the row to a Series object; it merely
returns the values inside a namedtuple. Therefore,
`~DataFrame.itertuples` preserves the data type of the values
and is generally faster than `~DataFrame.iterrows`.

> **note.capitalize():**
   The column names will be renamed to positional names if they are
   invalid Python identifiers, repeated, or start with an underscore.
   With a large number of columns (>255), regular tuples are returned.



## .dt accessor
``Series`` has an accessor to succinctly return datetime like properties for the
*values* of the Series, if it is a datetime/period like Series.
This will return a Series, indexed like the existing Series.

```python
# datetime
s = pd.Series(pd.date_range("20130101 09:10:12", periods=4))
s
s.dt.hour
s.dt.second
s.dt.day
```
This enables nice expressions like this:

```python
s[s.dt.day == 2]
```
You can easily produces tz aware transformations:

```python
stz = s.dt.tz_localize("US/Eastern")
stz
stz.dt.tz
```
You can also chain these types of operations:

```python
s.dt.tz_localize("UTC").dt.tz_convert("US/Eastern")
```
You can also format datetime values as strings with `Series.dt.strftime` which
supports the same format as the standard `~datetime.datetime.strftime`.

```python
# DatetimeIndex
s = pd.Series(pd.date_range("20130101", periods=4))
s
s.dt.strftime("%Y/%m/%d")
```
```python
# PeriodIndex
s = pd.Series(pd.period_range("20130101", periods=4))
s
s.dt.strftime("%Y/%m/%d")
```
The ``.dt`` accessor works for period and timedelta dtypes.

```python
# period
s = pd.Series(pd.period_range("20130101", periods=4, freq="D"))
s
s.dt.year
s.dt.day
```
```python
# timedelta
s = pd.Series(pd.timedelta_range("1 day 00:00:05", periods=4, freq="s"))
s
s.dt.days
s.dt.seconds
s.dt.components
```
> **note.capitalize():**
   ``Series.dt`` will raise a ``TypeError`` if you access with a non-datetime-like values.

## Vectorized string methods
Series is equipped with a set of string processing methods that make it easy to
operate on each element of the array. Perhaps most importantly, these methods
exclude missing/NA values automatically. These are accessed via the Series's
``str`` attribute and generally have names matching the equivalent (scalar)
built-in string methods. For example:

 ```python
s = pd.Series(
    ["A", "B", "C", "Aaba", "Baca", np.nan, "CABA", "dog", "cat"], dtype="string"
)
s.str.lower()
```
Powerful pattern-matching methods are provided as well, but note that
pattern-matching generally uses `regular expressions
<https://docs.python.org/3/library/re.html>`__ by default (and in some cases
always uses them).

> **note.capitalize():**
   Prior to pandas 1.0, string methods were only available on ``object`` -dtype
   ``Series``. pandas 1.0 added the `StringDtype` which is dedicated
   to strings. See `text.types` for more.

Please see `Vectorized String Methods <text.string_methods>[ for a complete
description.



## Sorting
pandas supports three kinds of sorting: sorting by index labels,
sorting by column values, and sorting by a combination of both.



### By index
The `Series.sort_index` and `DataFrame.sort_index` methods are
used to sort a pandas object by its index levels.

```python
df = pd.DataFrame(
    {
        "one": pd.Series(np.random.randn(3), index=["a", "b", "c"]),
        "two": pd.Series(np.random.randn(4), index=["a", "b", "c", "d"]),
        "three": pd.Series(np.random.randn(3), index=["b", "c", "d"]),
    }
)

unsorted_df = df.reindex(
    index=["a", "d", "c", "b"], columns=["three", "two", "one"]
)
unsorted_df

# DataFrame
unsorted_df.sort_index()
unsorted_df.sort_index(ascending=False)
unsorted_df.sort_index(axis=1)

# Series
unsorted_df["three"].sort_index()
```


Sorting by index also supports a ``key`` parameter that takes a callable
function to apply to the index being sorted. For ``MultiIndex`` objects,
the key is applied per-level to the levels specified by ``level``.

```python
s1 = pd.DataFrame({"a": ["B", "a", "C"], "b": [1, 2, 3], "c": [2, 3, 4]}).set_index(
    list("ab")
)
s1
```
```python
s1.sort_index(level="a")
s1.sort_index(level="a", key=lambda idx: idx.str.lower())
```
For information on key sorting by value, see value sorting
.



### By values
The `Series.sort_values` method is used to sort a ``Series`` by its values. The
`DataFrame.sort_values` method is used to sort a ``DataFrame`` by its column or row values.
The optional ``by`` parameter to `DataFrame.sort_values` may used to specify one or more columns
to use to determine the sorted order.

```python
df1 = pd.DataFrame(
    {"one": [2, 1, 1, 1], "two": [1, 3, 2, 4], "three": [5, 4, 3, 2]}
)
df1.sort_values(by="two")
```
The ``by`` parameter can take a list of column names, e.g.:

```python
df1[["one", "two", "three"]].sort_values(by=["one", "two"])
```
These methods have special treatment of NA values via the ``na_position``
argument:

```python
s[2] = np.nan
s.sort_values()
s.sort_values(na_position="first")
```


Sorting also supports a ``key`` parameter that takes a callable function
to apply to the values being sorted.

```python
s1 = pd.Series(["B", "a", "C"])
```
```python
s1.sort_values()
s1.sort_values(key=lambda x: x.str.lower())
```
``key`` will be given the `Series` of values and should return a ``Series``
or array of the same shape with the transformed values. For ``DataFrame`` objects,
the key is applied per column, so the key should still expect a Series and return
a Series, e.g.

```python
df = pd.DataFrame({"a": ["B", "a", "C"], "b": [1, 2, 3]})
```
```python
df.sort_values(by="a")
df.sort_values(by="a", key=lambda col: col.str.lower())
```
The name or type of each column can be used to apply different functions to
different columns.



### By indexes and values
Strings passed as the ``by`` parameter to `DataFrame.sort_values` may
refer to either columns or index level names.

```python
# Build MultiIndex
idx = pd.MultiIndex.from_tuples(
    [("a", 1), ("a", 2), ("a", 2), ("b", 2), ("b", 1), ("b", 1)]
)
idx.names = ["first", "second"]

# Build DataFrame
df_multi = pd.DataFrame({"A": np.arange(6, 0, -1)}, index=idx)
df_multi
```
Sort by 'second' (index) and 'A' (column)

```python
df_multi.sort_values(by=["second", "A"])
```
> **note.capitalize():**
   If a string matches both a column name and an index level name then a
   warning is issued and the column takes precedence. This will result in an
   ambiguity error in a future version.



### searchsorted
Series has the `~Series.searchsorted` method, which works similarly to
`numpy.ndarray.searchsorted`.

```python
ser = pd.Series([1, 2, 3])
ser.searchsorted([0, 3])
ser.searchsorted([0, 4])
ser.searchsorted([1, 3], side="right")
ser.searchsorted([1, 3], side="left")
ser = pd.Series([3, 1, 2])
ser.searchsorted([0, 3], sorter=np.argsort(ser))
```


### smallest / largest values
``Series`` has the `~Series.nsmallest` and `~Series.nlargest` methods which return the
smallest or largest `n` values. For a large ``Series`` this can be much
faster than sorting the entire Series and calling ``head(n)`` on the result.

```python
s = pd.Series(np.random.permutation(10))
s
s.sort_values()
s.nsmallest(3)
s.nlargest(3)
```
``DataFrame`` also has the ``nlargest`` and ``nsmallest`` methods.

```python
df = pd.DataFrame(
    {
        "a": [-2, -1, 1, 10, 8, 11, -1],
        "b": list("abdceff"),
        "c": [1.0, 2.0, 4.0, 3.2, np.nan, 3.0, 4.0],
    }
)
df.nlargest(3, "a")
df.nlargest(5, ["a", "c"])
df.nsmallest(3, "a")
df.nsmallest(5, ["a", "c"])
```


### Sorting by a MultiIndex column
You must be explicit about sorting when the column is a MultiIndex, and fully specify
all levels to ``by``.

```python
df1.columns = pd.MultiIndex.from_tuples(
    [("a", "one"), ("a", "two"), ("b", "three")]
)
df1.sort_values(by=("a", "two"))
```
## Copying
The `~DataFrame.copy` method on pandas objects copies the underlying data (though not
the axis indexes, since they are immutable) and returns a new object. Note that
**it is seldom necessary to copy objects**. For example, there are only a
handful of ways to alter a DataFrame *in-place*:

* Inserting, deleting, or modifying a column.
* Assigning to the ``index`` or ``columns`` attributes.
* For homogeneous data, directly modifying the values via the ``values``
  attribute or advanced indexing.

To be clear, no pandas method has the side effect of modifying your data;
almost every method returns a new object, leaving the original object
untouched. If the data is modified, it is because you did so explicitly.



## dtypes
For the most part, pandas uses NumPy arrays and dtypes for Series or individual
columns of a DataFrame. NumPy provides support for ``float``,
``int``, ``bool``, ``timedelta64[ns]`` and ``datetime64[ns]`` (note that NumPy
does not support timezone-aware datetimes).

pandas and third-party libraries *extend* NumPy's type system in a few places.
This section describes the extensions pandas has made internally.
See `extending.extension-types` for how to write your own extension that
works with pandas. See `the ecosystem page](https://pandas.pydata.org/community/ecosystem.html) for a list of third-party
libraries that have implemented an extension.

The following table lists all of pandas extension types. For methods requiring ``dtype``
arguments, strings can be specified as indicated. See the respective
documentation sections for more on each type.

+-------------------------------------------------+---------------------------+--------------------+-------------------------------+----------------------------------------+
| Kind of Data                                    | Data Type                 | Scalar             | Array                         | String Aliases                         |
+=================================================+===============+===========+========+===========+===============================+========================================+
| `tz-aware datetime <timeseries.timezone>`  | `DatetimeTZDtype`  | `Timestamp` | `arrays.DatetimeArray` | ``'datetime64[ns, <tz>]'``             |
|                                                 |                           |                    |                               |                                        |
+-------------------------------------------------+---------------+-----------+--------------------+-------------------------------+----------------------------------------+
| `Categorical <categorical>`                | `CategoricalDtype` | (none)             | `Categorical`          | ``'category'``                         |
+-------------------------------------------------+---------------------------+--------------------+-------------------------------+----------------------------------------+
| `period (time spans) <timeseries.periods>` | `PeriodDtype`      | `Period`    | `arrays.PeriodArray`   | ``'period[<freq>]'``,                  |
|                                                 |                           |                    | ``'Period[<freq>]'``          |                                        |
+-------------------------------------------------+---------------------------+--------------------+-------------------------------+----------------------------------------+
| `sparse <sparse>`                          | `SparseDtype`      | (none)             | `arrays.SparseArray`   | ``'Sparse'``, ``'Sparse[int]'``,       |
|                                                 |                           |                    |                               | ``'Sparse[float]'``                    |
+-------------------------------------------------+---------------------------+--------------------+-------------------------------+----------------------------------------+
| `intervals <advanced.intervalindex>`       | `IntervalDtype`    | `Interval`  | `arrays.IntervalArray` | ``'interval'``, ``'Interval'``,        |
|                                                 |                           |                    |                               | ``'Interval[<numpy_dtype>]'``,         |
|                                                 |                           |                    |                               | ``'Interval[datetime64[ns, <tz>]]'``,  |
|                                                 |                           |                    |                               | ``'Interval[timedelta64[<freq>]]'``    |
+-------------------------------------------------+---------------------------+--------------------+-------------------------------+----------------------------------------+
| `nullable integer <integer_na>`            | `Int64Dtype`, ...  | (none)             | `arrays.IntegerArray`  | ``'Int8'``, ``'Int16'``, ``'Int32'``,  |
|                                                 |                           |                    |                               | ``'Int64'``, ``'UInt8'``, ``'UInt16'``,|
|                                                 |                           |                    |                               | ``'UInt32'``, ``'UInt64'``             |
+-------------------------------------------------+---------------------------+--------------------+-------------------------------+----------------------------------------+
| `nullable float <api.arrays.float_na>`     | `Float64Dtype`, ...| (none)             | `arrays.FloatingArray` | ``'Float32'``, ``'Float64'``           |
+-------------------------------------------------+---------------------------+--------------------+-------------------------------+----------------------------------------+
| `Strings <text>`                           | `StringDtype`      | `str`       | `arrays.StringArray`   | ``'string'``                           |
+-------------------------------------------------+---------------------------+--------------------+-------------------------------+----------------------------------------+
| `Boolean (with NA) <api.arrays.bool>`      | `BooleanDtype`     | `bool`      | `arrays.BooleanArray`  | ``'boolean'``                          |
+-------------------------------------------------+---------------------------+--------------------+-------------------------------+----------------------------------------+

pandas has two ways to store strings.

1. ``object`` dtype, which can hold any Python object, including strings.
2. `StringDtype`, which is dedicated to strings.

Generally, we recommend using `StringDtype`. See `text.types` for more.

Finally, arbitrary objects may be stored using the ``object`` dtype, but should
be avoided to the extent possible (for performance and interoperability with
other libraries and methods. See `basics.object_conversion`).

A convenient `~DataFrame.dtypes` attribute for DataFrame returns a Series
with the data type of each column.

```python
dft = pd.DataFrame(
    {
        "A": np.random.rand(3),
        "B": 1,
        "C": "foo",
        "D": pd.Timestamp("20010102"),
        "E": pd.Series([1.0] * 3).astype("float32"),
        "F": False,
        "G": pd.Series([1] * 3, dtype="int8"),
    }
)
dft
dft.dtypes
```
On a ``Series`` object, use the `~Series.dtype` attribute.

```python
dft["A"].dtype
```
If a pandas object contains data with multiple dtypes *in a single column*, the
dtype of the column will be chosen to accommodate all of the data types
(``object`` is the most general).

```python
# these ints are coerced to floats
pd.Series([1, 2, 3, 4, 5, 6.0])

# string data forces an ``object`` dtype
pd.Series([1, 2, 3, 6.0, "foo"])
```
The number of columns of each type in a ``DataFrame`` can be found by calling
``DataFrame.dtypes.value_counts()``.

```python
dft.dtypes.value_counts()
```
Numeric dtypes will propagate and can coexist in DataFrames.
If a dtype is passed (either directly via the ``dtype`` keyword, a passed ``ndarray``,
or a passed ``Series``), then it will be preserved in DataFrame operations. Furthermore,
different numeric dtypes will **NOT** be combined. The following example will give you a taste.

```python
df1 = pd.DataFrame(np.random.randn(8, 1), columns=["A"], dtype="float64")
df1
df1.dtypes
df2 = pd.DataFrame(
    {
        "A": pd.Series(np.random.randn(8), dtype="float32"),
        "B": pd.Series(np.random.randn(8)),
        "C": pd.Series(np.random.randint(0, 255, size=8), dtype="uint8"),  # [0,255] (range of uint8)
    }
)
df2
df2.dtypes
```
### defaults
By default integer types are ``int64`` and float types are ``float64``,
*regardless* of platform (32-bit or 64-bit).
The following will all result in ``int64`` dtypes.

```python
pd.DataFrame([1, 2], columns=["a"]).dtypes
pd.DataFrame({"a": [1, 2]}).dtypes
pd.DataFrame({"a": 1}, index=list(range(2))).dtypes
```
Note that Numpy will choose *platform-dependent* types when creating arrays.
The following **WILL** result in ``int32`` on 32-bit platform.

```python
frame = pd.DataFrame(np.array([1, 2]))
```
### upcasting
Types can potentially be *upcasted* when combined with other types, meaning they are promoted
from the current type (e.g. ``int`` to ``float``).

```python
df3 = df1.reindex_like(df2).fillna(value=0.0) + df2
df3
df3.dtypes
```
`DataFrame.to_numpy` will return the *lower-common-denominator* of the dtypes, meaning
the dtype that can accommodate **ALL** of the types in the resulting homogeneous dtyped NumPy array. This can
force some *upcasting*.

```python
df3.to_numpy().dtype
```
### astype


You can use the `~DataFrame.astype` method to explicitly convert dtypes from one to another. These will by default return a copy,
even if the dtype was unchanged (pass ``copy=False`` to change this behavior). In addition, they will raise an
exception if the astype operation is invalid.

Upcasting is always according to the **NumPy** rules. If two different dtypes are involved in an operation,
then the more *general* one will be used as the result of the operation.

```python
df3
df3.dtypes

# conversion of dtypes
df3.astype("float32").dtypes
```
Convert a subset of columns to a specified type using `~DataFrame.astype`.

```python
dft = pd.DataFrame({"a": [1, 2, 3], "b": [4, 5, 6], "c": [7, 8, 9]})
dft[["a", "b"]] = dft[["a", "b"]].astype(np.uint8)
dft
dft.dtypes
```
Convert certain columns to a specific dtype by passing a dict to `~DataFrame.astype`.

```python
dft1 = pd.DataFrame({"a": [1, 0, 1], "b": [4, 5, 6], "c": [7, 8, 9]})
dft1 = dft1.astype({"a": np.bool_, "c": np.float64})
dft1
dft1.dtypes
```
> **note.capitalize():**
    When trying to convert a subset of columns to a specified type using `~DataFrame.astype`  and `~DataFrame.loc`, upcasting occurs.

    `~DataFrame.loc` tries to fit in what we are assigning to the current dtypes, while ``[]`` will overwrite them taking the dtype from the right hand side. Therefore the following piece of code produces the unintended result.

    ```python
dft = pd.DataFrame({"a": [1, 2, 3], "b": [4, 5, 6], "c": [7, 8, 9]})
dft.loc[:, ["a", "b"]].astype(np.uint8).dtypes
dft.loc[:, ["a", "b"]] = dft.loc[:, ["a", "b"]].astype(np.uint8)
dft.dtypes
```


### object conversion
pandas offers various functions to try to force conversion of types from the ``object`` dtype to other types.
In cases where the data is already of the correct type, but stored in an ``object`` array, the
`DataFrame.infer_objects` and `Series.infer_objects` methods can be used to soft convert
to the correct type.

  ```python
import datetime

df = pd.DataFrame(
    [
        [1, 2],
        ["a", "b"],
        [datetime.datetime(2016, 3, 2), datetime.datetime(2016, 3, 2)],
    ]
)
df = df.T
df
df.dtypes
```
Because the data was transposed the original inference stored all columns as object, which
``infer_objects`` will correct.

  ```python
df.infer_objects().dtypes
```
The following functions are available for one dimensional object arrays or scalars to perform
hard conversion of objects to a specified type:

* `~pandas.to_numeric` (conversion to numeric dtypes)

  ```python
m = ["1.1", 2, 3]
pd.to_numeric(m)
```
* `~pandas.to_datetime` (conversion to datetime objects)

  ```python
import datetime

m = ["2016-07-09", datetime.datetime(2016, 3, 2)]
pd.to_datetime(m)
```
* `~pandas.to_timedelta` (conversion to timedelta objects)

  ```python
m = ["5us", pd.Timedelta("1day")]
pd.to_timedelta(m)
```
To force a conversion, we can pass in an ``errors`` argument, which specifies how pandas should deal with elements
that cannot be converted to desired dtype or object. By default, ``errors='raise'``, meaning that any errors encountered
will be raised during the conversion process. However, if ``errors='coerce'``, these errors will be ignored and pandas
will convert problematic elements to ``pd.NaT`` (for datetime and timedelta) or ``np.nan`` (for numeric). This might be
useful if you are reading in data which is mostly of the desired dtype (e.g. numeric, datetime), but occasionally has
non-conforming elements intermixed that you want to represent as missing:

```python
:okwarning:

 import datetime

 m = ["apple", datetime.datetime(2016, 3, 2)]
 pd.to_datetime(m, errors="coerce")

 m = ["apple", 2, 3]
 pd.to_numeric(m, errors="coerce")

 m = ["apple", pd.Timedelta("1day")]
 pd.to_timedelta(m, errors="coerce")
```
In addition to object conversion, `~pandas.to_numeric` provides another argument ``downcast``, which gives the
option of downcasting the newly (or already) numeric data to a smaller dtype, which can conserve memory:

```python
m = ["1", 2, 3]
pd.to_numeric(m, downcast="integer")  # smallest signed int dtype
pd.to_numeric(m, downcast="signed")  # same as 'integer'
pd.to_numeric(m, downcast="unsigned")  # smallest unsigned int dtype
pd.to_numeric(m, downcast="float")  # smallest float dtype
```
As these methods apply only to one-dimensional arrays, lists or scalars; they cannot be used directly on multi-dimensional objects such
as DataFrames. However, with `~pandas.DataFrame.apply`, we can "apply" the function over each column efficiently:

```python
import datetime

df = pd.DataFrame([["2016-07-09", datetime.datetime(2016, 3, 2)]] * 2, dtype="O")
df
df.apply(pd.to_datetime)

df = pd.DataFrame([["1.1", 2, 3]] * 2, dtype="O")
df
df.apply(pd.to_numeric)

df = pd.DataFrame([["5us", pd.Timedelta("1day")]] * 2, dtype="O")
df
df.apply(pd.to_timedelta)
```
### gotchas
Performing selection operations on ``integer`` type data can easily upcast the data to ``floating``.
The dtype of the input data will be preserved in cases where ``nans`` are not introduced.
See also `Support for integer NA <gotchas.intna>`.

```python
dfi = df3.astype("int32")
dfi["E"] = 1
dfi
dfi.dtypes

casted = dfi[dfi > 0]
casted
casted.dtypes
```
While float dtypes are unchanged.

```python
dfa = df3.copy()
dfa["A"] = dfa["A"].astype("float32")
dfa.dtypes

casted = dfa[df2 > 0]
casted
casted.dtypes
```
## Selecting columns based on ``dtype``


The `~DataFrame.select_dtypes` method implements subsetting of columns
based on their ``dtype``.

First, let's create a `DataFrame` with a slew of different
dtypes:

```python
df = pd.DataFrame(
    {
        "string": list("abc"),
        "int64": list(range(1, 4)),
        "uint8": np.arange(3, 6).astype("u1"),
        "float64": np.arange(4.0, 7.0),
        "bool1": [True, False, True],
        "bool2": [False, True, False],
        "dates": pd.date_range("now", periods=3),
        "category": pd.Series(list("ABC")).astype("category"),
    }
)
df["tdeltas"] = df.dates.diff()
df["uint64"] = np.arange(3, 6).astype("u8")
df["other_dates"] = pd.date_range("20130101", periods=3)
df["tz_aware_dates"] = pd.date_range("20130101", periods=3, tz="US/Eastern")
df
```
And the dtypes:

```python
df.dtypes
```
`~DataFrame.select_dtypes` has two parameters ``include`` and ``exclude`` that allow you to
say "give me the columns *with* these dtypes" (``include``) and/or "give the
columns *without* these dtypes" (``exclude``).

For example, to select ``bool`` columns:

```python
df.select_dtypes(include=[bool])
```
You can also pass the name of a dtype in the `NumPy dtype hierarchy
<https://numpy.org/doc/stable/reference/arrays.scalars.html>`__:

```python
df.select_dtypes(include=["bool"])
```
`~pandas.DataFrame.select_dtypes` also works with generic dtypes as well.

For example, to select all numeric and boolean columns while excluding unsigned
integers:

```python
df.select_dtypes(include=["number", "bool"], exclude=["unsignedinteger"])
```
To select string columns you must use the ``object`` dtype:

```python
df.select_dtypes(include=["object"])
```
To see all the child dtypes of a generic ``dtype`` like ``numpy.number`` you
can define a function that returns a tree of child dtypes:

```python
def subdtypes(dtype):
    subs = dtype.__subclasses__()
    if not subs:
        return dtype
    return [dtype, [subdtypes(dt) for dt in subs]]
```
All NumPy dtypes are subclasses of ``numpy.generic``:

```python
subdtypes(np.generic)
```
> **note.capitalize():**
    pandas also defines the types ``category``, and ``datetime64[ns, tz]``, which are not integrated into the normal
    NumPy hierarchy and won't show up with the above function.

---

# 
# IO tools (text, CSV, HDF5, ...)
The pandas I/O API is a set of top level [`reader`` functions accessed like
`pandas.read_csv` that generally return a pandas object. The corresponding
``writer`` functions are object methods that are accessed like
`DataFrame.to_csv`. Below is a table containing available ``readers`` and
``writers``.


    :header: "Format Type", "Data Description", "Reader", "Writer"
    :widths: 30, 100, 60, 60

    text,`CSV](https://en.wikipedia.org/wiki/Comma-separated_values)_, `read_csv<io.read_csv_table>`, `to_csv<io.store_in_csv>`
    text,Fixed-Width Text File, `read_fwf<io.fwf_reader>[, NA
    text,`JSON](https://www.json.org/)_, `read_json<io.json_reader>`, `to_json<io.json_writer>[
    text,`HTML](https://en.wikipedia.org/wiki/HTML)_, `read_html<io.read_html>`, `to_html<io.html>[
    text,`LaTeX](https://en.wikipedia.org/wiki/LaTeX)_, NA, `Styler.to_latex<io.latex>[
    text,`XML](https://www.w3.org/standards/xml/core)_, `read_xml<io.read_xml>`, `to_xml<io.xml>`
    text, Local clipboard, `read_clipboard<io.clipboard>`, `to_clipboard<io.clipboard>[
    binary,`MS Excel](https://en.wikipedia.org/wiki/Microsoft_Excel)_ , `read_excel<io.excel_reader>`, `to_excel<io.excel_writer>[
    binary,`OpenDocument](http://opendocumentformat.org)_, `read_excel<io.ods>[, NA
    binary,`HDF5 Format](https://support.hdfgroup.org/documentation/hdf5/latest/_intro_h_d_f5.html)_, `read_hdf<io.hdf5>`, `to_hdf<io.hdf5>[
    binary,`Feather Format](https://github.com/wesm/feather)_, `read_feather<io.feather>`, `to_feather<io.feather>[
    binary,`Parquet Format](https://parquet.apache.org/)_, `read_parquet<io.parquet>`, `to_parquet<io.parquet>[
    binary,`Apache Iceberg](https://iceberg.apache.org/)_, `read_iceberg<io.iceberg>` , `to_iceberg<io.iceberg>[
    binary,`ORC Format](https://orc.apache.org/)_, `read_orc<io.orc>`, `to_orc<io.orc>[
    binary,`Stata](https://en.wikipedia.org/wiki/Stata)_, `read_stata<io.stata_reader>`, `to_stata<io.stata_writer>[
    binary,`SAS](https://en.wikipedia.org/wiki/SAS_(software))_, `read_sas<io.sas_reader>[ , NA
    binary,`SPSS](https://en.wikipedia.org/wiki/SPSS)_, `read_spss<io.spss_reader>[ , NA
    binary,`Python Pickle Format](https://docs.python.org/3/library/pickle.html)_, `read_pickle<io.pickle>`, `to_pickle<io.pickle>[
    SQL,`SQL](https://en.wikipedia.org/wiki/SQL)_, `read_sql<io.sql>`,`to_sql<io.sql>`

`Here <io.perf>` is an informal performance comparison for some of these IO methods.

> **note.capitalize():**
   For examples that use the ``StringIO`` class, make sure you import it
   with ``from io import StringIO`` for Python 3.



## CSV & text files
The workhorse function for reading text files (a.k.a. flat files) is
`read_csv`. See the `cookbook<cookbook.csv>` for some advanced strategies.

Parsing options
'''''''''''''''

`read_csv` accepts the following common arguments:

Basic
+++++

filepath_or_buffer : various
  Either a path to a file (a `python:str`, `python:pathlib.Path`)
  URL (including http, ftp, and S3
  locations), or any object with a ``read()`` method (such as an open file or
  `~python:io.StringIO`).
sep : str, defaults to ``','`` for `read_csv`, ``\t`` for `read_table`
  Delimiter to use. If sep is ``None``, the C engine cannot automatically detect
  the separator, but the Python parsing engine can, meaning the latter will be
  used and automatically detect the separator by Python's builtin sniffer tool,
  `python:csv.Sniffer`. In addition, separators longer than 1 character and
  different from ``'\s+'`` will be interpreted as regular expressions and
  will also force the use of the Python parsing engine. Note that regex
  delimiters are prone to ignoring quoted data. Regex example: ``'\\r\\t'``.
delimiter : str, default ``None``
  Alternative argument name for sep.

Column and index locations and names
++++++++++++++++++++++++++++++++++++

header : int or list of ints, default ``'infer'``
  Row number(s) to use as the column names, and the start of the
  data. Default behavior is to infer the column names: if no names are
  passed the behavior is identical to ``header=0`` and column names
  are inferred from the first line of the file, if column names are
  passed explicitly then the behavior is identical to
  ``header=None``. Explicitly pass ``header=0`` to be able to replace
  existing names.

  The header can be a list of ints that specify row locations
  for a MultiIndex on the columns e.g. ``[0,1,3]``. Intervening rows
  that are not specified will be skipped (e.g. 2 in this example is
  skipped). Note that this parameter ignores commented lines and empty
  lines if ``skip_blank_lines=True``, so header=0 denotes the first
  line of data rather than the first line of the file.
names : array-like, default ``None``
  List of column names to use. If file contains no header row, then you should
  explicitly pass ``header=None``. Duplicates in this list are not allowed.
index_col : int, str, sequence of int / str, or False, optional, default ``None``
  Column(s) to use as the row labels of the ``DataFrame``, either given as
  string name or column index. If a sequence of int / str is given, a
  MultiIndex is used.

  .. note::
     ``index_col=False`` can be used to force pandas to *not* use the first
     column as the index, e.g. when you have a malformed file with delimiters at
     the end of each line.

  The default value of ``None`` instructs pandas to guess. If the number of
  fields in the column header row is equal to the number of fields in the body
  of the data file, then a default index is used.  If it is larger, then
  the first columns are used as index so that the remaining number of fields in
  the body are equal to the number of fields in the header.

  The first row after the header is used to determine the number of columns,
  which will go into the index. If the subsequent rows contain less columns
  than the first row, they are filled with ``NaN``.

  This can be avoided through ``usecols``. This ensures that the columns are
  taken as is and the trailing data are ignored.
usecols : list-like or callable, default ``None``
  Return a subset of the columns. If list-like, all elements must either
  be positional (i.e. integer indices into the document columns) or strings
  that correspond to column names provided either by the user in ``names`` or
  inferred from the document header row(s). If ``names`` are given, the document
  header row(s) are not taken into account. For example, a valid list-like
  ``usecols`` parameter would be ``[0, 1, 2]`` or ``['foo', 'bar', 'baz']``.

  Element order is ignored, so ``usecols=[0, 1]`` is the same as ``[1, 0]``. To
  instantiate a DataFrame from ``data`` with element order preserved use
  ``pd.read_csv(data, usecols=['foo', 'bar'])[['foo', 'bar']]`` for columns
  in ``['foo', 'bar']`` order or
  ``pd.read_csv(data, usecols=['foo', 'bar'])[['bar', 'foo']]`` for
  ``['bar', 'foo']`` order.

  If callable, the callable function will be evaluated against the column names,
  returning names where the callable function evaluates to True:

  ```python
import pandas as pd
   from io import StringIO

   data = "col1,col2,col3\na,b,1\na,b,2\nc,d,3"
   pd.read_csv(StringIO(data))
   pd.read_csv(StringIO(data), usecols=lambda x: x.upper() in ["COL1", "COL3"])

Using this parameter results in much faster parsing time and lower memory usage
when using the c engine. The Python engine loads the data first before deciding
which columns to drop.
```
General parsing configuration
+++++++++++++++++++++++++++++

dtype : Type name or dict of column -> type, default ``None``
  Data type for data or columns. E.g. ``{'a': np.float64, 'b': np.int32, 'c': 'Int64'}``
  Use ``str`` or ``object`` together with suitable ``na_values`` settings to preserve
  and not interpret dtype. If converters are specified, they will be applied INSTEAD
  of dtype conversion.

  .. versionadded:: 1.5.0

     Support for defaultdict was added. Specify a defaultdict as input where
     the default determines the dtype of the columns which are not explicitly
     listed.

dtype_backend : {"numpy_nullable", "pyarrow"}, defaults to NumPy backed DataFrames
  Which dtype_backend to use, e.g. whether a DataFrame should have NumPy
  arrays, nullable dtypes are used for all dtypes that have a nullable
  implementation when "numpy_nullable" is set, pyarrow is used for all
  dtypes if "pyarrow" is set.

  The dtype_backends are still experimental.

  .. versionadded:: 2.0

engine : {``'c'``, ``'python'``, ``'pyarrow'``}
  Parser engine to use. The C and pyarrow engines are faster, while the python engine
  is currently more feature-complete. Multithreading is currently only supported by
  the pyarrow engine.

  .. versionadded:: 1.4.0

     The "pyarrow" engine was added as an *experimental* engine, and some features
     are unsupported, or may not work correctly, with this engine.
converters : dict, default ``None``
  Dict of functions for converting values in certain columns. Keys can either be
  integers or column labels.
true_values : list, default ``None``
  Values to consider as ``True``.
false_values : list, default ``None``
  Values to consider as ``False``.
skipinitialspace : boolean, default ``False``
  Skip spaces after delimiter.
skiprows : list-like or integer, default ``None``
  Line numbers to skip (0-indexed) or number of lines to skip (int) at the start
  of the file.

  If callable, the callable function will be evaluated against the row
  indices, returning True if the row should be skipped and False otherwise:

  ```python
data = "col1,col2,col3\na,b,1\na,b,2\nc,d,3"
pd.read_csv(StringIO(data))
pd.read_csv(StringIO(data), skiprows=lambda x: x % 2 != 0)
```
skipfooter : int, default ``0``
  Number of lines at bottom of file to skip (unsupported with engine='c').

nrows : int, default ``None``
  Number of rows of file to read. Useful for reading pieces of large files.
low_memory : boolean, default ``True``
  Internally process the file in chunks, resulting in lower memory use
  while parsing, but possibly mixed type inference.  To ensure no mixed
  types either set ``False``, or specify the type with the ``dtype`` parameter.
  Note that the entire file is read into a single ``DataFrame`` regardless,
  use the ``chunksize`` or ``iterator`` parameter to return the data in chunks.
  (Only valid with C parser)
memory_map : boolean, default False
  If a filepath is provided for ``filepath_or_buffer``, map the file object
  directly onto memory and access the data directly from there. Using this
  option can improve performance because there is no longer any I/O overhead.

NA and missing data handling
++++++++++++++++++++++++++++

na_values : scalar, str, list-like, or dict, default ``None``
  Additional strings to recognize as NA/NaN. If dict passed, specific per-column
  NA values. See `na values const <io.navaluesconst>[ below
  for a list of the values interpreted as NaN by default.

keep_default_na : boolean, default ``True``
  Whether or not to include the default NaN values when parsing the data.
  Depending on whether ``na_values`` is passed in, the behavior is as follows:

  * If ``keep_default_na`` is ``True``, and ``na_values`` are specified, ``na_values``
    is appended to the default NaN values used for parsing.
  * If ``keep_default_na`` is ``True``, and ``na_values`` are not specified, only
    the default NaN values are used for parsing.
  * If ``keep_default_na`` is ``False``, and ``na_values`` are specified, only
    the NaN values specified ``na_values`` are used for parsing.
  * If ``keep_default_na`` is ``False``, and ``na_values`` are not specified, no
    strings will be parsed as NaN.

  Note that if ``na_filter`` is passed in as ``False``, the ``keep_default_na`` and
  ``na_values`` parameters will be ignored.
na_filter : boolean, default ``True``
  Detect missing value markers (empty strings and the value of na_values). In
  data without any NAs, passing ``na_filter=False`` can improve the performance
  of reading a large file.
verbose : boolean, default ``False``
  Indicate number of NA values placed in non-numeric columns.
skip_blank_lines : boolean, default ``True``
  If ``True``, skip over blank lines rather than interpreting as NaN values.



Datetime handling
+++++++++++++++++

parse_dates : boolean or list of ints or names or list of lists or dict, default ``False``.
  * If ``True`` -> try parsing the index.
  * If ``[1, 2, 3]`` ->  try parsing columns 1, 2, 3 each as a separate date
    column.

  .. note::
     A fast-path exists for iso8601-formatted dates.
date_format : str or dict of column -> format, default ``None``
   If used in conjunction with ``parse_dates``, will parse dates according to this
   format. For anything more complex,
   please read in as ``object`` and then apply `to_datetime` as-needed.

   .. versionadded:: 2.0.0
dayfirst : boolean, default ``False``
  DD/MM format dates, international and European format.
cache_dates : boolean, default True
  If True, use a cache of unique, converted dates to apply the datetime
  conversion. May produce significant speed-up when parsing duplicate
  date strings, especially ones with timezone offsets.

Iteration
+++++++++

iterator : boolean, default ``False``
  Return ``TextFileReader`` object for iteration or getting chunks with
  ``get_chunk()``.
chunksize : int, default ``None``
  Return ``TextFileReader`` object for iteration. See iterating and chunking
   below.

Quoting, compression, and file format
+++++++++++++++++++++++++++++++++++++

compression : {``'infer'``, ``'gzip'``, ``'bz2'``, ``'zip'``, ``'xz'``, ``'zstd'``, ``None``, ``dict``}, default ``'infer'``
  For on-the-fly decompression of on-disk data. If 'infer', then use gzip,
  bz2, zip, xz, or zstandard if ``filepath_or_buffer`` is path-like ending in '.gz', '.bz2',
  '.zip', '.xz', '.zst', respectively, and no decompression otherwise. If using 'zip',
  the ZIP file must contain only one data file to be read in.
  Set to ``None`` for no decompression. Can also be a dict with key ``'method'``
  set to one of {``'zip'``, ``'gzip'``, ``'bz2'``, ``'zstd'``} and other key-value pairs are
  forwarded to ``zipfile.ZipFile``, ``gzip.GzipFile``, ``bz2.BZ2File``, or ``zstandard.ZstdDecompressor``.
  As an example, the following could be passed for faster compression and to
  create a reproducible gzip archive:
  ``compression={'method': 'gzip', 'compresslevel': 1, 'mtime': 1}``.

  .. versionchanged:: 1.2.0 Previous versions forwarded dict entries for 'gzip' to ``gzip.open``.
thousands : str, default ``None``
  Thousands separator.
decimal : str, default ``'.'``
  Character to recognize as decimal point. E.g. use ``','`` for European data.
float_precision : string, default None
  Specifies which converter the C engine should use for floating-point values.
  The options are ``None`` for the ordinary converter, ``high`` for the
  high-precision converter, and ``round_trip`` for the round-trip converter.
lineterminator : str (length 1), default ``None``
  Character to break file into lines. Only valid with C parser.
quotechar : str (length 1)
  The character used to denote the start and end of a quoted item. Quoted items
  can include the delimiter and it will be ignored.
quoting : int or ``csv.QUOTE_*`` instance, default ``0``
  Control field quoting behavior per ``csv.QUOTE_*`` constants. Use one of
  ``QUOTE_MINIMAL`` (0), ``QUOTE_ALL`` (1), ``QUOTE_NONNUMERIC`` (2) or
  ``QUOTE_NONE`` (3).
doublequote : boolean, default ``True``
   When ``quotechar`` is specified and ``quoting`` is not ``QUOTE_NONE``,
   indicate whether or not to interpret two consecutive ``quotechar`` elements
   **inside** a field as a single ``quotechar`` element.
escapechar : str (length 1), default ``None``
  One-character string used to escape delimiter when quoting is ``QUOTE_NONE``.
comment : str, default ``None``
  Indicates remainder of line should not be parsed. If found at the beginning of
  a line, the line will be ignored altogether. This parameter must be a single
  character. Like empty lines (as long as ``skip_blank_lines=True``), fully
  commented lines are ignored by the parameter ``header`` but not by ``skiprows``.
  For example, if ``comment='#'``, parsing '#empty\\na,b,c\\n1,2,3' with
  ``header=0`` will result in 'a,b,c' being treated as the header.
encoding : str, default ``None``
  Encoding to use for UTF when reading/writing (e.g. ``'utf-8'``). `List of
  Python standard encodings
 ](https://docs.python.org/3/library/codecs.html#standard-encodings).
dialect : str or `python:csv.Dialect` instance, default ``None``
  If provided, this parameter will override values (default or not) for the
  following parameters: ``delimiter``, ``doublequote``, ``escapechar``,
  ``skipinitialspace``, ``quotechar``, and ``quoting``. If it is necessary to
  override values, a ParserWarning will be issued. See `python:csv.Dialect`
  documentation for more details.

Error handling
++++++++++++++

on_bad_lines : {{'error', 'warn', 'skip'}}, default 'error'
    Specifies what to do upon encountering a bad line (a line with too many fields).
    Allowed values are :

    - 'error', raise an ParserError when a bad line is encountered.
    - 'warn', print a warning when a bad line is encountered and skip that line.
    - 'skip', skip bad lines without raising or warning when they are encountered.

    .. versionadded:: 1.3.0



Specifying column data types
''''''''''''''''''''''''''''

You can indicate the data type for the whole ``DataFrame`` or individual
columns:

```python
import numpy as np

data = "a,b,c,d\n1,2,3,4\n5,6,7,8\n9,10,11"
print(data)

df = pd.read_csv(StringIO(data), dtype=object)
df
df["a"][0]
df = pd.read_csv(StringIO(data), dtype={"b": object, "c": np.float64, "d": "Int64"})
df.dtypes
```
Fortunately, pandas offers more than one way to ensure that your column(s)
contain only one ``dtype``. If you're unfamiliar with these concepts, you can
see `here<basics.dtypes>` to learn more about dtypes, and
`here<basics.object_conversion>` to learn more about ``object`` conversion in
pandas.


For instance, you can use the ``converters`` argument
of `~pandas.read_csv`:

```python
data = "col_1\n1\n2\n'A'\n4.22"
df = pd.read_csv(StringIO(data), converters={"col_1": str})
df
df["col_1"].apply(type).value_counts()
```
Or you can use the `~pandas.to_numeric` function to coerce the
dtypes after reading in the data,

```python
df2 = pd.read_csv(StringIO(data))
df2["col_1"] = pd.to_numeric(df2["col_1"], errors="coerce")
df2
df2["col_1"].apply(type).value_counts()
```
which will convert all valid parsing to floats, leaving the invalid parsing
as ``NaN``.

Ultimately, how you deal with reading in columns containing mixed dtypes
depends on your specific needs. In the case above, if you wanted to ``NaN`` out
the data anomalies, then `~pandas.to_numeric` is probably your best option.
However, if you wanted for all the data to be coerced, no matter the type, then
using the ``converters`` argument of `~pandas.read_csv` would certainly be
worth trying.

> **note.capitalize():**
   In some cases, reading in abnormal data with columns containing mixed dtypes
   will result in an inconsistent dataset. If you rely on pandas to infer the
   dtypes of your columns, the parsing engine will go and infer the dtypes for
   different chunks of the data, rather than the whole dataset at once. Consequently,
   you can end up with column(s) with mixed dtypes. For example,

   ```python
:okwarning:

     col_1 = list(range(500000)) + ["a", "b"] + list(range(500000))
     df = pd.DataFrame({"col_1": col_1})
     df.to_csv("foo.csv")
     mixed_df = pd.read_csv("foo.csv")
     mixed_df["col_1"].apply(type).value_counts()
     mixed_df["col_1"].dtype

will result with ``mixed_df`` containing an ``int`` dtype for certain chunks
of the column, and ``str`` for others due to the mixed dtypes from the
data that was read in. It is important to note that the overall column will be
marked with a ``dtype`` of ``object``, which is used for columns with mixed dtypes.
```
```python
:suppress:

import os

os.remove("foo.csv")
```
Setting ``dtype_backend="numpy_nullable"`` will result in nullable dtypes for every column.

```python
data = """a,b,c,d,e,f,g,h,i,j
1,2.5,True,a,,,,,12-31-2019,
3,4.5,False,b,6,7.5,True,a,12-31-2019,
"""

df = pd.read_csv(StringIO(data), dtype_backend="numpy_nullable", parse_dates=["i"])
df
df.dtypes
```


Specifying categorical dtype
''''''''''''''''''''''''''''

``Categorical`` columns can be parsed directly by specifying ``dtype='category'`` or
``dtype=CategoricalDtype(categories, ordered)``.

```python
data = "col1,col2,col3\na,b,1\na,b,2\nc,d,3"

pd.read_csv(StringIO(data))
pd.read_csv(StringIO(data)).dtypes
pd.read_csv(StringIO(data), dtype="category").dtypes
```
Individual columns can be parsed as a ``Categorical`` using a dict
specification:

```python
pd.read_csv(StringIO(data), dtype={"col1": "category"}).dtypes
```
Specifying ``dtype='category'`` will result in an unordered ``Categorical``
whose ``categories`` are the unique values observed in the data. For more
control on the categories and order, create a
`~pandas.api.types.CategoricalDtype` ahead of time, and pass that for
that column's ``dtype``.

```python
from pandas.api.types import CategoricalDtype

dtype = CategoricalDtype(["d", "c", "b", "a"], ordered=True)
pd.read_csv(StringIO(data), dtype={"col1": dtype}).dtypes
```
When using ``dtype=CategoricalDtype``, "unexpected" values outside of
``dtype.categories`` are treated as missing values.

```python
:okwarning:

dtype = CategoricalDtype(["a", "b", "d"])  # No 'c'
pd.read_csv(StringIO(data), dtype={"col1": dtype}).col1
```
This matches the behavior of `Categorical.set_categories`. This behavior is
deprecated. In a future version, the presence of non-NA values that are not
among the specified categories will raise.

> **note.capitalize():**
   With ``dtype='category'``, the resulting categories will always be parsed
   as strings (object dtype). If the categories are numeric they can be
   converted using the `to_numeric` function, or as appropriate, another
   converter such as `to_datetime`.

   When ``dtype`` is a ``CategoricalDtype`` with homogeneous ``categories`` (
   all numeric, all datetimes, etc.), the conversion is done automatically.

   ```python
df = pd.read_csv(StringIO(data), dtype="category")
df.dtypes
df["col3"]
new_categories = pd.to_numeric(df["col3"].cat.categories)
df["col3"] = df["col3"].cat.rename_categories(new_categories)
df["col3"]
```
Naming and using columns
''''''''''''''''''''''''



Handling column names
+++++++++++++++++++++

A file may or may not have a header row. pandas assumes the first row should be
used as the column names:

```python
data = "a,b,c\n1,2,3\n4,5,6\n7,8,9"
print(data)
pd.read_csv(StringIO(data))
```
By specifying the ``names`` argument in conjunction with ``header`` you can
indicate other names to use and whether or not to throw away the header row (if
any):

```python
print(data)
pd.read_csv(StringIO(data), names=["foo", "bar", "baz"], header=0)
pd.read_csv(StringIO(data), names=["foo", "bar", "baz"], header=None)
```
If the header is in a row other than the first, pass the row number to
``header``. This will skip the preceding rows:

```python
data = "skip this skip it\na,b,c\n1,2,3\n4,5,6\n7,8,9"
pd.read_csv(StringIO(data), header=1)
```
> **note.capitalize():**
  Default behavior is to infer the column names: if no names are
  passed the behavior is identical to ``header=0`` and column names
  are inferred from the first non-blank line of the file, if column
  names are passed explicitly then the behavior is identical to
  ``header=None``.



Duplicate names parsing
'''''''''''''''''''''''

If the file or header contains duplicate names, pandas will by default
distinguish between them so as to prevent overwriting data:

```python
data = "a,b,a\n0,1,2\n3,4,5"
pd.read_csv(StringIO(data))
```
There is no more duplicate data because duplicate columns 'X', ..., 'X' become
'X', 'X.1', ..., 'X.N'.



Filtering columns (``usecols``)
+++++++++++++++++++++++++++++++

The ``usecols`` argument allows you to select any subset of the columns in a
file, either using the column names, position numbers or a callable:

```python
data = "a,b,c,d\n1,2,3,foo\n4,5,6,bar\n7,8,9,baz"
pd.read_csv(StringIO(data))
pd.read_csv(StringIO(data), usecols=["b", "d"])
pd.read_csv(StringIO(data), usecols=[0, 2, 3])
pd.read_csv(StringIO(data), usecols=lambda x: x.upper() in ["A", "C"])
```
The ``usecols`` argument can also be used to specify which columns not to
use in the final result:

```python
pd.read_csv(StringIO(data), usecols=lambda x: x not in ["a", "c"])
```
In this case, the callable is specifying that we exclude the "a" and "c"
columns from the output.

Comments and empty lines
''''''''''''''''''''''''



Ignoring line comments and empty lines
++++++++++++++++++++++++++++++++++++++

If the ``comment`` parameter is specified, then completely commented lines will
be ignored. By default, completely blank lines will be ignored as well.

```python
data = "\na,b,c\n  \n# commented line\n1,2,3\n\n4,5,6"
print(data)
pd.read_csv(StringIO(data), comment="#")
```
If ``skip_blank_lines=False``, then ``read_csv`` will not ignore blank lines:

```python
data = "a,b,c\n\n1,2,3\n\n\n4,5,6"
pd.read_csv(StringIO(data), skip_blank_lines=False)
```
> **warning.capitalize():**
   The presence of ignored lines might create ambiguities involving line numbers;
   the parameter ``header`` uses row numbers (ignoring commented/empty
   lines), while ``skiprows`` uses line numbers (including commented/empty lines):

   ```python
data = "#comment\na,b,c\nA,B,C\n1,2,3"
   pd.read_csv(StringIO(data), comment="#", header=1)
   data = "A,B,C\n#comment\na,b,c\n1,2,3"
   pd.read_csv(StringIO(data), comment="#", skiprows=2)

If both ``header`` and ``skiprows`` are specified, ``header`` will be
relative to the end of ``skiprows``. For example:
```
```python
data = (
    "# empty\n"
    "# second empty line\n"
    "# third emptyline\n"
    "X,Y,Z\n"
    "1,2,3\n"
    "A,B,C\n"
    "1,2.,4.\n"
    "5.,NaN,10.0\n"
)
print(data)
pd.read_csv(StringIO(data), comment="#", skiprows=4, header=1)
```


Comments
++++++++

Sometimes comments or meta data may be included in a file:

```python
data = (
    "ID,level,category\n"
    "Patient1,123000,x # really unpleasant\n"
    "Patient2,23000,y # wouldn't take his medicine\n"
    "Patient3,1234018,z # awesome"
)
with open("tmp.csv", "w") as fh:
    fh.write(data)

print(open("tmp.csv").read())
```
By default, the parser includes the comments in the output:

```python
df = pd.read_csv("tmp.csv")
df
```
We can suppress the comments using the ``comment`` keyword:

```python
df = pd.read_csv("tmp.csv", comment="#")
df
```
```python
:suppress:

os.remove("tmp.csv")
```


Dealing with Unicode data
'''''''''''''''''''''''''

The ``encoding`` argument should be used for encoded unicode data, which will
result in byte strings being decoded to unicode in the result:

```python
from io import BytesIO

data = b"word,length\n" b"Tr\xc3\xa4umen,7\n" b"Gr\xc3\xbc\xc3\x9fe,5"
data = data.decode("utf8").encode("latin-1")
df = pd.read_csv(BytesIO(data), encoding="latin-1")
df
df["word"][1]
```
Some formats which encode all characters as multiple bytes, like UTF-16, won't
parse correctly at all without specifying the encoding. `Full list of Python
standard encodings
<https://docs.python.org/3/library/codecs.html#standard-encodings>`_.



Index columns and trailing delimiters
'''''''''''''''''''''''''''''''''''''

If a file has one more column of data than the number of column names, the
first column will be used as the ``DataFrame``'s row names:

```python
data = "a,b,c\n4,apple,bat,5.7\n8,orange,cow,10"
pd.read_csv(StringIO(data))
```
```python
data = "index,a,b,c\n4,apple,bat,5.7\n8,orange,cow,10"
pd.read_csv(StringIO(data), index_col=0)
```
Ordinarily, you can achieve this behavior using the ``index_col`` option.

There are some exception cases when a file has been prepared with delimiters at
the end of each data line, confusing the parser. To explicitly disable the
index column inference and discard the last column, pass ``index_col=False``:

```python
data = "a,b,c\n4,apple,bat,\n8,orange,cow,"
print(data)
pd.read_csv(StringIO(data))
pd.read_csv(StringIO(data), index_col=False)
```
If a subset of data is being parsed using the ``usecols`` option, the
``index_col`` specification is based on that subset, not the original data.

```python
data = "a,b,c\n4,apple,bat,\n8,orange,cow,"
print(data)
pd.read_csv(StringIO(data), usecols=["b", "c"])
pd.read_csv(StringIO(data), usecols=["b", "c"], index_col=0)
```


Date Handling
'''''''''''''

Specifying date columns
+++++++++++++++++++++++

To better facilitate working with datetime data, `read_csv`
uses the keyword arguments ``parse_dates`` and ``date_format``
to allow users to specify a variety of columns and date/time formats to turn the
input text data into ``datetime`` objects.

The simplest case is to just pass in ``parse_dates=True``:

```python
with open("foo.csv", mode="w") as f:
    f.write("date,A,B,C\n20090101,a,1,2\n20090102,b,3,4\n20090103,c,4,5")

# Use a column as an index, and parse it as dates.
df = pd.read_csv("foo.csv", index_col=0, parse_dates=True)
df

# These are Python datetime objects
df.index
```
It is often the case that we may want to store date and time data separately,
or store various date fields separately. the ``parse_dates`` keyword can be
used to specify columns to parse the dates and/or times.


> **note.capitalize():**
   If a column or index contains an unparsable date, the entire column or
   index will be returned unaltered as an object data type. For non-standard
   datetime parsing, use `to_datetime` after ``pd.read_csv``.


> **note.capitalize():**
   read_csv has a fast_path for parsing datetime strings in iso8601 format,
   e.g "2000-01-01T00:01:02+00:00" and similar variations. If you can arrange
   for your data to store datetimes in this format, load times will be
   significantly faster, ~20x has been observed.


Date parsing functions
++++++++++++++++++++++

Finally, the parser allows you to specify a custom ``date_format``.
Performance-wise, you should try these methods of parsing dates in order:

1. If you know the format, use ``date_format``, e.g.:
   ``date_format="%d/%m/%Y"`` or ``date_format={column_name: "%d/%m/%Y"}``.

2. If you different formats for different columns, or want to pass any extra options (such
   as ``utc``) to ``to_datetime``, then you should read in your data as ``object`` dtype, and
   then use ``to_datetime``.




Parsing a CSV with mixed timezones
++++++++++++++++++++++++++++++++++

pandas cannot natively represent a column or index with mixed timezones. If your CSV
file contains columns with a mixture of timezones, the default result will be
an object-dtype column with strings, even with ``parse_dates``.
To parse the mixed-timezone values as a datetime column, read in as ``object`` dtype and
then call `to_datetime` with ``utc=True``.


```python
content = """\
a
2000-01-01T00:00:00+05:00
2000-01-01T00:00:00+06:00"""
df = pd.read_csv(StringIO(content))
df["a"] = pd.to_datetime(df["a"], utc=True)
df["a"]
```



Inferring datetime format
+++++++++++++++++++++++++

Here are some examples of datetime strings that can be guessed (all
representing December 30th, 2011 at 00:00:00):

* "20111230"
* "2011/12/30"
* "20111230 00:00:00"
* "12/30/2011 00:00:00"
* "30/Dec/2011 00:00:00"
* "30/December/2011 00:00:00"

Note that format inference is sensitive to ``dayfirst``.  With
``dayfirst=True``, it will guess "01/12/2011" to be December 1st. With
``dayfirst=False`` (default) it will guess "01/12/2011" to be January 12th.

If you try to parse a column of date strings, pandas will attempt to guess the format
from the first non-NaN element, and will then parse the rest of the column with that
format. If pandas fails to guess the format (for example if your first string is
``'01 December US/Pacific 2000'``), then a warning will be raised and each
row will be parsed individually by ``dateutil.parser.parse``. The safest
way to parse dates is to explicitly set ``format=``.

```python
df = pd.read_csv(
    "foo.csv",
    index_col=0,
    parse_dates=True,
)
df
```
In the case that you have mixed datetime formats within the same column, you can
pass  ``format='mixed'``

```python
data = StringIO("date\n12 Jan 2000\n2000-01-13\n")
df = pd.read_csv(data)
df['date'] = pd.to_datetime(df['date'], format='mixed')
df
```
or, if your datetime formats are all ISO8601 (possibly not identically-formatted):

```python
data = StringIO("date\n2020-01-01\n2020-01-01 03:00\n")
df = pd.read_csv(data)
df['date'] = pd.to_datetime(df['date'], format='ISO8601')
df
```
```python
:suppress:

os.remove("foo.csv")
```
International date formats
++++++++++++++++++++++++++

While US date formats tend to be MM/DD/YYYY, many international formats use
DD/MM/YYYY instead. For convenience, a ``dayfirst`` keyword is provided:

```python
data = "date,value,cat\n1/6/2000,5,a\n2/6/2000,10,b\n3/6/2000,15,c"
print(data)
with open("tmp.csv", "w") as fh:
    fh.write(data)

pd.read_csv("tmp.csv", parse_dates=[0])
pd.read_csv("tmp.csv", dayfirst=True, parse_dates=[0])
```
```python
:suppress:

os.remove("tmp.csv")
```
Writing CSVs to binary file objects
+++++++++++++++++++++++++++++++++++



``df.to_csv(..., mode="wb")`` allows writing a CSV to a file object
opened binary mode. In most cases, it is not necessary to specify
``mode`` as pandas will auto-detect whether the file object is
opened in text or binary mode.

```python
import io

data = pd.DataFrame([0, 1, 2])
buffer = io.BytesIO()
data.to_csv(buffer, encoding="utf-8", compression="gzip")
```


Specifying method for floating-point conversion
'''''''''''''''''''''''''''''''''''''''''''''''

The parameter ``float_precision`` can be specified in order to use
a specific floating-point converter during parsing with the C engine.
The options are the ordinary converter, the high-precision converter, and
the round-trip converter (which is guaranteed to round-trip values after
writing to a file). For example:

```python
val = "0.3066101993807095471566981359501369297504425048828125"
data = "a,b,c\n1,2,{0}".format(val)
abs(
    pd.read_csv(
        StringIO(data),
        engine="c",
        float_precision=None,
    )["c"][0] - float(val)
)
abs(
    pd.read_csv(
        StringIO(data),
        engine="c",
        float_precision="high",
    )["c"][0] - float(val)
)
abs(
    pd.read_csv(StringIO(data), engine="c", float_precision="round_trip")["c"][0]
    - float(val)
)
```


Thousand separators
'''''''''''''''''''

For large numbers that have been written with a thousands separator, you can
set the ``thousands`` keyword to a string of length 1 so that integers will be parsed
correctly.

By default, numbers with a thousands separator will be parsed as strings:

```python
data = (
    "ID|level|category\n"
    "Patient1|123,000|x\n"
    "Patient2|23,000|y\n"
    "Patient3|1,234,018|z"
)

with open("tmp.csv", "w") as fh:
    fh.write(data)

df = pd.read_csv("tmp.csv", sep="|")
df

df.level.dtype
```
The ``thousands`` keyword allows integers to be parsed correctly:

```python
df = pd.read_csv("tmp.csv", sep="|", thousands=",")
df

df.level.dtype
```
```python
:suppress:

os.remove("tmp.csv")
```


NA values
'''''''''

To control which values are parsed as missing values (which are signified by
``NaN``), specify a string in ``na_values``. If you specify a list of strings,
then all values in it are considered to be missing values. If you specify a
number (a ``float``, like ``5.0`` or an ``integer`` like ``5``), the
corresponding equivalent values will also imply a missing value (in this case
effectively ``[5.0, 5]`` are recognized as ``NaN``).

To completely override the default values that are recognized as missing, specify ``keep_default_na=False``.



The default ``NaN`` recognized values are ``['-1.#IND', '1.#QNAN', '1.#IND', '-1.#QNAN', '#N/A N/A', '#N/A', 'N/A',
'n/a', 'NA', '<NA>', '#NA', 'NULL', 'null', 'NaN', '-NaN', 'nan', '-nan', 'None', '']``.

Let us consider some examples:

```python
pd.read_csv("path_to_file.csv", na_values=[5])
```
In the example above ``5`` and ``5.0`` will be recognized as ``NaN``, in
addition to the defaults. A string will first be interpreted as a numerical
``5``, then as a ``NaN``.

```python
pd.read_csv("path_to_file.csv", keep_default_na=False, na_values=[""])
```
Above, only an empty field will be recognized as ``NaN``.

```python
pd.read_csv("path_to_file.csv", keep_default_na=False, na_values=["NA", "0"])
```
Above, both ``NA`` and ``0`` as strings are ``NaN``.

```python
pd.read_csv("path_to_file.csv", na_values=["Nope"])
```
The default values, in addition to the string ``"Nope"`` are recognized as
``NaN``.



Infinity
''''''''

``inf`` like values will be parsed as ``np.inf`` (positive infinity), and ``-inf`` as ``-np.inf`` (negative infinity).
These will ignore the case of the value, meaning ``Inf``, will also be parsed as ``np.inf``.



Boolean values
''''''''''''''

The common values ``True``, ``False``, ``TRUE``, and ``FALSE`` are all
recognized as boolean. Occasionally you might want to recognize other values
as being boolean. To do this, use the ``true_values`` and ``false_values``
options as follows:

```python
data = "a,b,c\n1,Yes,2\n3,No,4"
print(data)
pd.read_csv(StringIO(data))
pd.read_csv(StringIO(data), true_values=["Yes"], false_values=["No"])
```


Handling "bad" lines
''''''''''''''''''''

Some files may have malformed lines with too few fields or too many. Lines with
too few fields will have NA values filled in the trailing fields. Lines with
too many fields will raise an error by default:

```python
:okexcept:

data = "a,b,c\n1,2,3\n4,5,6,7\n8,9,10"
pd.read_csv(StringIO(data))
```
You can elect to skip bad lines:

```python
data = "a,b,c\n1,2,3\n4,5,6,7\n8,9,10"
pd.read_csv(StringIO(data), on_bad_lines="skip")
```


Or pass a callable function to handle the bad line if ``engine="python"``.
The bad line will be a list of strings that was split by the ``sep``:

```python
external_list = []
def bad_lines_func(line):
    external_list.append(line)
    return line[-3:]
pd.read_csv(StringIO(data), on_bad_lines=bad_lines_func, engine="python")
external_list
```
> **note.capitalize():**
   The callable function will handle only a line with too many fields.
   Bad lines caused by other errors will be silently skipped.

   ```python
bad_lines_func = lambda line: print(line)

   data = 'name,type\nname a,a is of type a\nname b,"b\" is of type b"'
   data
   pd.read_csv(StringIO(data), on_bad_lines=bad_lines_func, engine="python")

The line was not processed in this case, as a "bad line" here is caused by an escape character.
```
You can also use the ``usecols`` parameter to eliminate extraneous column
data that appear in some lines but not others:

```python
:okexcept:

pd.read_csv(StringIO(data), usecols=[0, 1, 2])
```
In case you want to keep all data including the lines with too many fields, you can
specify a sufficient number of ``names``. This ensures that lines with not enough
fields are filled with ``NaN``.

```python
pd.read_csv(StringIO(data), names=['a', 'b', 'c', 'd'])
```


Dialect
'''''''

The ``dialect`` keyword gives greater flexibility in specifying the file format.
By default it uses the Excel dialect but you can specify either the dialect name
or a `python:csv.Dialect` instance.

Suppose you had data with unenclosed quotes:

```python
data = "label1,label2,label3\n" 'index1,"a,c,e\n' "index2,b,d,f"
print(data)
```
By default, ``read_csv`` uses the Excel dialect and treats the double quote as
the quote character, which causes it to fail when it finds a newline before it
finds the closing double quote.

We can get around this using ``dialect``:

```python
:okwarning:

import csv

dia = csv.excel()
dia.quoting = csv.QUOTE_NONE
pd.read_csv(StringIO(data), dialect=dia)
```
All of the dialect options can be specified separately by keyword arguments:

```python
data = "a,b,c~1,2,3~4,5,6"
pd.read_csv(StringIO(data), lineterminator="~")
```
Another common dialect option is ``skipinitialspace``, to skip any whitespace
after a delimiter:

```python
data = "a, b, c\n1, 2, 3\n4, 5, 6"
print(data)
pd.read_csv(StringIO(data), skipinitialspace=True)
```
The parsers make every attempt to "do the right thing" and not be fragile. Type
inference is a pretty big deal. If a column can be coerced to integer dtype
without altering the contents, the parser will do so. Any non-numeric
columns will come through as object dtype as with the rest of pandas objects.



Quoting and Escape Characters
'''''''''''''''''''''''''''''

Quotes (and other escape characters) in embedded fields can be handled in any
number of ways. One way is to use backslashes; to properly parse this data, you
should pass the ``escapechar`` option:

```python
data = 'a,b\n"hello, \\"Bob\\", nice to see you",5'
print(data)
pd.read_csv(StringIO(data), escapechar="\\")
```



Files with fixed width columns
''''''''''''''''''''''''''''''

While `read_csv` reads delimited data, the `read_fwf` function works
with data files that have known and fixed column widths. The function parameters
to ``read_fwf`` are largely the same as ``read_csv`` with two extra parameters, and
a different usage of the ``delimiter`` parameter:

* ``colspecs``: A list of pairs (tuples) giving the extents of the
  fixed-width fields of each line as half-open intervals (i.e.,  [from, to[ ).
  String value 'infer' can be used to instruct the parser to try detecting
  the column specifications from the first 100 rows of the data. Default
  behavior, if not specified, is to infer.
* ``widths``: A list of field widths which can be used instead of 'colspecs'
  if the intervals are contiguous.
* ``delimiter``: Characters to consider as filler characters in the fixed-width file.
  Can be used to specify the filler character of the fields
  if it is not spaces (e.g., '~').

Consider a typical fixed-width data file:

```python
data1 = (
    "id8141    360.242940   149.910199   11950.7\n"
    "id1594    444.953632   166.985655   11788.4\n"
    "id1849    364.136849   183.628767   11806.2\n"
    "id1230    413.836124   184.375703   11916.8\n"
    "id1948    502.953953   173.237159   12468.3"
)
with open("bar.csv", "w") as f:
    f.write(data1)
```
In order to parse this file into a ``DataFrame``, we simply need to supply the
column specifications to the ``read_fwf`` function along with the file name:

```python
# Column specifications are a list of half-intervals
colspecs = [(0, 6), (8, 20), (21, 33), (34, 43)]
df = pd.read_fwf("bar.csv", colspecs=colspecs, header=None, index_col=0)
df
```
Note how the parser automatically picks column names X.<column number> when
``header=None`` argument is specified. Alternatively, you can supply just the
column widths for contiguous columns:

```python
# Widths are a list of integers
widths = [6, 14, 13, 10]
df = pd.read_fwf("bar.csv", widths=widths, header=None)
df
```
The parser will take care of extra white spaces around the columns
so it's ok to have extra separation between the columns in the file.

By default, ``read_fwf`` will try to infer the file's ``colspecs`` by using the
first 100 rows of the file. It can do it only in cases when the columns are
aligned and correctly separated by the provided ``delimiter`` (default delimiter
is whitespace).

```python
df = pd.read_fwf("bar.csv", header=None, index_col=0)
df
```
``read_fwf`` supports the ``dtype`` parameter for specifying the types of
parsed columns to be different from the inferred type.

```python
pd.read_fwf("bar.csv", header=None, index_col=0).dtypes
pd.read_fwf("bar.csv", header=None, dtype={2: "object"}).dtypes
```
```python
:suppress:

os.remove("bar.csv")
```
Indexes
'''''''

Files with an "implicit" index column
+++++++++++++++++++++++++++++++++++++

Consider a file with one less entry in the header than the number of data
column:

```python
data = "A,B,C\n20090101,a,1,2\n20090102,b,3,4\n20090103,c,4,5"
print(data)
with open("foo.csv", "w") as f:
    f.write(data)
```
In this special case, ``read_csv`` assumes that the first column is to be used
as the index of the ``DataFrame``:

```python
pd.read_csv("foo.csv")
```
Note that the dates weren't automatically parsed. In that case you would need
to do as before:

```python
df = pd.read_csv("foo.csv", parse_dates=True)
df.index
```
```python
:suppress:

os.remove("foo.csv")
```
Reading an index with a ``MultiIndex``
++++++++++++++++++++++++++++++++++++++



Suppose you have data indexed by two columns:

```python
data = 'year,indiv,zit,xit\n1977,"A",1.2,.6\n1977,"B",1.5,.5'
print(data)
with open("mindex_ex.csv", mode="w") as f:
    f.write(data)
```
The ``index_col`` argument to ``read_csv`` can take a list of
column numbers to turn multiple columns into a ``MultiIndex`` for the index of the
returned object:

```python
df = pd.read_csv("mindex_ex.csv", index_col=[0, 1])
df
df.loc[1977]
```
```python
:suppress:

os.remove("mindex_ex.csv")
```


Reading columns with a ``MultiIndex``
+++++++++++++++++++++++++++++++++++++

By specifying list of row locations for the ``header`` argument, you
can read in a ``MultiIndex`` for the columns. Specifying non-consecutive
rows will skip the intervening rows.

```python
mi_idx = pd.MultiIndex.from_arrays([[1, 2, 3, 4], list("abcd")], names=list("ab"))
mi_col = pd.MultiIndex.from_arrays([[1, 2], list("ab")], names=list("cd"))
df = pd.DataFrame(np.ones((4, 2)), index=mi_idx, columns=mi_col)
df.to_csv("mi.csv")
print(open("mi.csv").read())
pd.read_csv("mi.csv", header=[0, 1, 2, 3], index_col=[0, 1])
```
``read_csv`` is also able to interpret a more common format
of multi-columns indices.

```python
data = ",a,a,a,b,c,c\n,q,r,s,t,u,v\none,1,2,3,4,5,6\ntwo,7,8,9,10,11,12"
print(data)
with open("mi2.csv", "w") as fh:
    fh.write(data)

pd.read_csv("mi2.csv", header=[0, 1], index_col=0)
```
> **note.capitalize():**
   If an ``index_col`` is not specified (e.g. you don't have an index, or wrote it
   with ``df.to_csv(..., index=False)``), then any ``names`` on the columns index will
   be *lost*.

```python
:suppress:

os.remove("mi.csv")
os.remove("mi2.csv")
```


Automatically "sniffing" the delimiter
''''''''''''''''''''''''''''''''''''''

``read_csv`` is capable of inferring delimited (not necessarily
comma-separated) files, as pandas uses the `python:csv.Sniffer`
class of the csv module. For this, you have to specify ``sep=None``.

```python
df = pd.DataFrame(np.random.randn(10, 4))
df.to_csv("tmp2.csv", sep=":", index=False)
pd.read_csv("tmp2.csv", sep=None, engine="python")
```
```python
:suppress:

os.remove("tmp2.csv")
```


Reading multiple files to create a single DataFrame
'''''''''''''''''''''''''''''''''''''''''''''''''''

It's best to use `~pandas.concat` to combine multiple files.
See the `cookbook<cookbook.csv.multiple_files>` for an example.



Iterating through files chunk by chunk
''''''''''''''''''''''''''''''''''''''

Suppose you wish to iterate through a (potentially very large) file lazily
rather than reading the entire file into memory, such as the following:


```python
df = pd.DataFrame(np.random.randn(10, 4))
df.to_csv("tmp.csv", index=False)
table = pd.read_csv("tmp.csv")
table
```
By specifying a ``chunksize`` to ``read_csv``, the return
value will be an iterable object of type ``TextFileReader``:

```python
with pd.read_csv("tmp.csv", chunksize=4) as reader:
    print(reader)
    for chunk in reader:
        print(chunk)
```


  ``read_csv/json/sas`` return a context-manager when iterating through a file.

Specifying ``iterator=True`` will also return the ``TextFileReader`` object:

```python
with pd.read_csv("tmp.csv", iterator=True) as reader:
    print(reader.get_chunk(5))
```
```python
:suppress:

os.remove("tmp.csv")
```
Specifying the parser engine
''''''''''''''''''''''''''''

pandas currently supports three engines, the C engine, the python engine, and an experimental
pyarrow engine (requires the ``pyarrow`` package). In general, the pyarrow engine is fastest
on larger workloads and is equivalent in speed to the C engine on most other workloads.
The python engine tends to be slower than the pyarrow and C engines on most workloads. However,
the pyarrow engine is much less robust than the C engine, which lacks a few features compared to the
Python engine.

Where possible, pandas uses the C parser (specified as ``engine='c'``), but it may fall
back to Python if C-unsupported options are specified.

Currently, options unsupported by the C and pyarrow engines include:

* ``sep`` other than a single character (e.g. regex separators)
* ``skipfooter``

Specifying any of the above options will produce a ``ParserWarning`` unless the
python engine is selected explicitly using ``engine='python'``.

Options that are unsupported by the pyarrow engine which are not covered by the list above include:

* ``float_precision``
* ``chunksize``
* ``comment``
* ``nrows``
* ``thousands``
* ``memory_map``
* ``dialect``
* ``on_bad_lines``
* ``quoting``
* ``lineterminator``
* ``converters``
* ``decimal``
* ``iterator``
* ``dayfirst``
* ``verbose``
* ``skipinitialspace``
* ``low_memory``

Specifying these options with ``engine='pyarrow'`` will raise a ``ValueError``.



Reading/writing remote files
''''''''''''''''''''''''''''

You can pass in a URL to read or write remote files to many of pandas' IO
functions - the following example shows reading a CSV file:

```python
df = pd.read_csv("https://download.bls.gov/pub/time.series/cu/cu.item", sep="\t")
```


A custom header can be sent alongside HTTP(s) requests by passing a dictionary
of header key value mappings to the ``storage_options`` keyword argument as shown below:

```python
headers = {"User-Agent": "pandas"}
df = pd.read_csv(
    "https://download.bls.gov/pub/time.series/cu/cu.item",
    sep="\t",
    storage_options=headers
)
```
All URLs which are not local files or HTTP(s) are handled by
`fsspec`_, if installed, and its various filesystem implementations
(including Amazon S3, Google Cloud, SSH, FTP, webHDFS...).
Some of these implementations will require additional packages to be
installed, for example
S3 URLs require the `s3fs
<https://pypi.org/project/s3fs/>`_ library:

```python
df = pd.read_json("s3://pandas-test/adatafile.json")
```
When dealing with remote storage systems, you might need
extra configuration with environment variables or config files in
special locations. For example, to access data in your S3 bucket,
you will need to define credentials in one of the several ways listed in
the `S3Fs documentation
<https://s3fs.readthedocs.io/en/latest/#credentials>`_. The same is true
for several of the storage backends, and you should follow the links
at `fsimpl1`_ for implementations built into ``fsspec`` and `fsimpl2`_
for those not included in the main ``fsspec``
distribution.

You can also pass parameters directly to the backend driver. Since ``fsspec`` does not
utilize the ``AWS_S3_HOST`` environment variable, we can directly define a
dictionary containing the endpoint_url and pass the object into the storage
option parameter:

```python
storage_options = {"client_kwargs": {"endpoint_url": "http://127.0.0.1:5555"}}
df = pd.read_json("s3://pandas-test/test-1", storage_options=storage_options)
```
More sample configurations and documentation can be found at `S3Fs documentation
<https://s3fs.readthedocs.io/en/latest/index.html?highlight=host#s3-compatible-storage>`__.

If you do *not* have S3 credentials, you can still access public
data by specifying an anonymous connection, such as



```python
pd.read_csv(
    "s3://ncei-wcsd-archive/data/processed/SH1305/18kHz/SaKe2013"
    "-D20130523-T080854_to_SaKe2013-D20130523-T085643.csv",
    storage_options={"anon": True},
)
```
``fsspec`` also allows complex URLs, for accessing data in compressed
archives, local caching of files, and more. To locally cache the above
example, you would modify the call to

```python
pd.read_csv(
    "simplecache::s3://ncei-wcsd-archive/data/processed/SH1305/18kHz/"
    "SaKe2013-D20130523-T080854_to_SaKe2013-D20130523-T085643.csv",
    storage_options={"s3": {"anon": True}},
)
```
where we specify that the "anon" parameter is meant for the "s3" part of
the implementation, not to the caching implementation. Note that this caches to a temporary
directory for the duration of the session only, but you can also specify
a permanent store.

.. _fsspec: https://filesystem-spec.readthedocs.io/en/latest/
.. _fsimpl1: https://filesystem-spec.readthedocs.io/en/latest/api.html#built-in-implementations
.. _fsimpl2: https://filesystem-spec.readthedocs.io/en/latest/api.html#other-known-implementations

Writing out data
''''''''''''''''



Writing to CSV format
+++++++++++++++++++++

The ``Series`` and ``DataFrame`` objects have an instance method ``to_csv`` which
allows storing the contents of the object as a comma-separated-values file. The
function takes a number of arguments. Only the first is required.

* ``path_or_buf``: A string path to the file to write or a file object.  If a file object it must be opened with ``newline=''``
* ``sep`` : Field delimiter for the output file (default ",")
* ``na_rep``: A string representation of a missing value (default '')
* ``float_format``: Format string for floating point numbers
* ``columns``: Columns to write (default None)
* ``header``: Whether to write out the column names (default True)
* ``index``: whether to write row (index) names (default True)
* ``index_label``: Column label(s) for index column(s) if desired. If None
  (default), and ``header`` and ``index`` are True, then the index names are
  used. (A sequence should be given if the ``DataFrame`` uses MultiIndex).
* ``mode`` : Python write mode, default 'w'
* ``encoding``: a string representing the encoding to use if the contents are
  non-ASCII, for Python versions prior to 3
* ``lineterminator``: Character sequence denoting line end (default ``os.linesep``)
* ``quoting``: Set quoting rules as in csv module (default csv.QUOTE_MINIMAL). Note that if you have set a ``float_format`` then floats are converted to strings and csv.QUOTE_NONNUMERIC will treat them as non-numeric
* ``quotechar``: Character used to quote fields (default '"')
* ``doublequote``: Control quoting of ``quotechar`` in fields (default True)
* ``escapechar``: Character used to escape ``sep`` and ``quotechar`` when
  appropriate (default None)
* ``chunksize``: Number of rows to write at a time
* ``date_format``: Format string for datetime objects

Writing a formatted string
++++++++++++++++++++++++++



The ``DataFrame`` object has an instance method ``to_string`` which allows control
over the string representation of the object. All arguments are optional:

* ``buf`` default None, for example a StringIO object
* ``columns`` default None, which columns to write
* ``col_space`` default None, minimum width of each column.
* ``na_rep`` default ``NaN``, representation of NA value
* ``formatters`` default None, a dictionary (by column) of functions each of
  which takes a single argument and returns a formatted string
* ``float_format`` default None, a function which takes a single (float)
  argument and returns a formatted string; to be applied to floats in the
  ``DataFrame``.
* ``sparsify`` default True, set to False for a ``DataFrame`` with a hierarchical
  index to print every MultiIndex key at each row.
* ``index_names`` default True, will print the names of the indices
* ``index`` default True, will print the index (ie, row labels)
* ``header`` default True, will print the column labels
* ``justify`` default ``left``, will print column headers left- or
  right-justified

The ``Series`` object also has a ``to_string`` method, but with only the ``buf``,
``na_rep``, ``float_format`` arguments. There is also a ``length`` argument
which, if set to ``True``, will additionally output the length of the Series.



## JSON
Read and write ``JSON`` format files and strings.



Writing JSON
''''''''''''

A ``Series`` or ``DataFrame`` can be converted to a valid JSON string. Use ``to_json``
with optional parameters:

* ``path_or_buf`` : the pathname or buffer to write the output.
  This can be ``None`` in which case a JSON string is returned.
* ``orient`` :

  ``Series``:
      * default is ``index``
      * allowed values are {``split``, ``records``, ``index``}

  ``DataFrame``:
      * default is ``columns``
      * allowed values are {``split``, ``records``, ``index``, ``columns``, ``values``, ``table``}

  The format of the JSON string

  .. csv-table::
     :widths: 20, 150

     ``split``, dict like {index -> [index]; columns -> [columns]; data -> [values]}
     ``records``, list like [{column -> value}; ... ]
     ``index``, dict like {index -> {column -> value}}
     ``columns``, dict like {column -> {index -> value}}
     ``values``, just the values array
     ``table``, adhering to the JSON `Table Schema`_

* ``date_format`` : string, type of date conversion, 'epoch' for timestamp, 'iso' for ISO8601.
* ``double_precision`` : The number of decimal places to use when encoding floating point values, default 10.
* ``force_ascii`` : force encoded string to be ASCII, default True.
* ``date_unit`` : The time unit to encode to, governs timestamp and ISO8601 precision. One of 's', 'ms', 'us' or 'ns' for seconds, milliseconds, microseconds and nanoseconds respectively. Default 'ms'.
* ``default_handler`` : The handler to call if an object cannot otherwise be converted to a suitable format for JSON. Takes a single argument, which is the object to convert, and returns a serializable object.
* ``lines`` : If ``records`` orient, then will write each record per line as json.
* ``mode`` : string, writer mode when writing to path. 'w' for write, 'a' for append. Default 'w'

Note ``NaN``'s, ``NaT``'s and ``None`` will be converted to ``null`` and ``datetime`` objects will be converted based on the ``date_format`` and ``date_unit`` parameters.

```python
dfj = pd.DataFrame(np.random.randn(5, 2), columns=list("AB"))
json = dfj.to_json()
json
```
Orient options
++++++++++++++

There are a number of different options for the format of the resulting JSON
file / string. Consider the following ``DataFrame`` and ``Series``:

```python
dfjo = pd.DataFrame(
    dict(A=range(1, 4), B=range(4, 7), C=range(7, 10)),
    columns=list("ABC"),
    index=list("xyz"),
)
dfjo
sjo = pd.Series(dict(x=15, y=16, z=17), name="D")
sjo
```
**Column oriented** (the default for ``DataFrame``) serializes the data as
nested JSON objects with column labels acting as the primary index:

```python
dfjo.to_json(orient="columns")
# Not available for Series
```
**Index oriented** (the default for ``Series``) similar to column oriented
but the index labels are now primary:

```python
dfjo.to_json(orient="index")
sjo.to_json(orient="index")
```
**Record oriented** serializes the data to a JSON array of column -> value records,
index labels are not included. This is useful for passing ``DataFrame`` data to plotting
libraries, for example the JavaScript library ``d3.js``:

```python
dfjo.to_json(orient="records")
sjo.to_json(orient="records")
```
**Value oriented** is a bare-bones option which serializes to nested JSON arrays of
values only, column and index labels are not included:

```python
dfjo.to_json(orient="values")
# Not available for Series
```
**Split oriented** serializes to a JSON object containing separate entries for
values, index and columns. Name is also included for ``Series``:

```python
dfjo.to_json(orient="split")
sjo.to_json(orient="split")
```
**Table oriented** serializes to the JSON `Table Schema`_, allowing for the
preservation of metadata including but not limited to dtypes and index names.

> **note.capitalize():**
  Any orient option that encodes to a JSON object will not preserve the ordering of
  index and column labels during round-trip serialization. If you wish to preserve
  label ordering use the ``split`` option as it uses ordered containers.

Date handling
+++++++++++++

Writing in ISO date format:

```python
dfd = pd.DataFrame(np.random.randn(5, 2), columns=list("AB"))
dfd["date"] = pd.Timestamp("20130101")
dfd = dfd.sort_index(axis=1, ascending=False)
json = dfd.to_json(date_format="iso")
json
```
Writing in ISO date format, with microseconds:

```python
json = dfd.to_json(date_format="iso", date_unit="us")
json
```
Writing to a file, with a date index and a date column:

```python
dfj2 = dfj.copy()
dfj2["date"] = pd.Timestamp("20130101")
dfj2["ints"] = list(range(5))
dfj2["bools"] = True
dfj2.index = pd.date_range("20130101", periods=5)
dfj2.to_json("test.json", date_format="iso")

with open("test.json") as fh:
    print(fh.read())
```
Fallback behavior
+++++++++++++++++

If the JSON serializer cannot handle the container contents directly it will
fall back in the following manner:

* if the dtype is unsupported (e.g. ``np.complex_``) then the ``default_handler``, if provided, will be called
  for each value, otherwise an exception is raised.

* if an object is unsupported it will attempt the following:


    - check if the object has defined a ``toDict`` method and call it.
      A ``toDict`` method should return a ``dict`` which will then be JSON serialized.

    - invoke the ``default_handler`` if one was provided.

    - convert the object to a ``dict`` by traversing its contents. However this will often fail
      with an ``OverflowError`` or give unexpected results.

In general the best approach for unsupported objects or dtypes is to provide a ``default_handler``.
For example:

```python
>>> DataFrame([1.0, 2.0, complex(1.0, 2.0)]).to_json()  # raises
RuntimeError: Unhandled numpy dtype 15
```
can be dealt with by specifying a simple ``default_handler``:

```python
pd.DataFrame([1.0, 2.0, complex(1.0, 2.0)]).to_json(default_handler=str)
```


Reading JSON
''''''''''''

Reading a JSON string to pandas object can take a number of parameters.
The parser will try to parse a ``DataFrame`` if ``typ`` is not supplied or
is ``None``. To explicitly force ``Series`` parsing, pass ``typ=series``

* ``filepath_or_buffer`` : a **VALID** JSON string or file handle / StringIO. The string could be
  a URL. Valid URL schemes include http, ftp, S3, and file. For file URLs, a host
  is expected. For instance, a local file could be
  file ://localhost/path/to/table.json
* ``typ``    : type of object to recover (series or frame), default 'frame'
* ``orient`` :

  Series :
      * default is ``index``
      * allowed values are {``split``, ``records``, ``index``}

  DataFrame
      * default is ``columns``
      * allowed values are {``split``, ``records``, ``index``, ``columns``, ``values``, ``table``}

  The format of the JSON string

  .. csv-table::
     :widths: 20, 150

     ``split``, dict like {index -> [index]; columns -> [columns]; data -> [values]}
     ``records``, list like [{column -> value} ...]
     ``index``, dict like {index -> {column -> value}}
     ``columns``, dict like {column -> {index -> value}}
     ``values``, just the values array
     ``table``, adhering to the JSON `Table Schema`_


* ``dtype`` : if True, infer dtypes, if a dict of column to dtype, then use those, if ``False``, then don't infer dtypes at all, default is True, apply only to the data.
* ``convert_axes`` : boolean, try to convert the axes to the proper dtypes, default is ``True``
* ``convert_dates`` : a list of columns to parse for dates; If ``True``, then try to parse date-like columns, default is ``True``.
* ``keep_default_dates`` : boolean, default ``True``. If parsing dates, then parse the default date-like columns.
* ``precise_float`` : boolean, default ``False``. Set to enable usage of higher precision (strtod) function when decoding string to double values. Default (``False``) is to use fast but less precise builtin functionality.
* ``date_unit`` : string, the timestamp unit to detect if converting dates. Default
  None. By default the timestamp precision will be detected, if this is not desired
  then pass one of 's', 'ms', 'us' or 'ns' to force timestamp precision to
  seconds, milliseconds, microseconds or nanoseconds respectively.
* ``lines`` : reads file as one json object per line.
* ``encoding`` : The encoding to use to decode py3 bytes.
* ``chunksize`` : when used in combination with ``lines=True``, return a ``pandas.api.typing.JsonReader`` which reads in ``chunksize`` lines per iteration.
* ``engine``: Either ``"ujson"``, the built-in JSON parser, or ``"pyarrow"`` which dispatches to pyarrow's ``pyarrow.json.read_json``.
  The ``"pyarrow"`` is only available when ``lines=True``

The parser will raise one of ``ValueError/TypeError/AssertionError`` if the JSON is not parseable.

If a non-default ``orient`` was used when encoding to JSON be sure to pass the same
option here so that decoding produces sensible results, see `Orient Options`_ for an
overview.

Data conversion
+++++++++++++++

The default of ``convert_axes=True``, ``dtype=True``, and ``convert_dates=True``
will try to parse the axes, and all of the data into appropriate types,
including dates. If you need to override specific dtypes, pass a dict to
``dtype``. ``convert_axes`` should only be set to ``False`` if you need to
preserve string-like numbers (e.g. '1', '2') in an axes.

> **note.capitalize():**
  Large integer values may be converted to dates if ``convert_dates=True`` and the data and / or column labels appear 'date-like'. The exact threshold depends on the ``date_unit`` specified. 'date-like' means that the column label meets one of the following criteria:

  * it ends with ``'_at'``
  * it ends with ``'_time'``
  * it begins with ``'timestamp'``
  * it is ``'modified'``
  * it is ``'date'``

> **warning.capitalize():**
   When reading JSON data, automatic coercing into dtypes has some quirks:

   * an index can be reconstructed in a different order from serialization, that is, the returned order is not guaranteed to be the same as before serialization
   * a column that was ``float`` data will be converted to ``integer`` if it can be done safely, e.g. a column of ``1.``
   * bool columns will be converted to ``integer`` on reconstruction

   Thus there are times where you may want to specify specific dtypes via the ``dtype`` keyword argument.

Reading from a JSON string:

```python
from io import StringIO
pd.read_json(StringIO(json))
```
Reading from a file:

```python
pd.read_json("test.json")
```
Don't convert any data (but still convert axes and dates):

```python
pd.read_json("test.json", dtype=object).dtypes
```
Specify dtypes for conversion:

```python
pd.read_json("test.json", dtype={"A": "float32", "bools": "int8"}).dtypes
```
Preserve string indices:

```python
from io import StringIO
si = pd.DataFrame(
    np.zeros((4, 4)), columns=list(range(4)), index=[str(i) for i in range(4)]
)
si
si.index
si.columns
json = si.to_json()

sij = pd.read_json(StringIO(json), convert_axes=False)
sij
sij.index
sij.columns
```
Dates written in nanoseconds need to be read back in nanoseconds:

```python
from io import StringIO
json = dfj2.to_json(date_format="iso", date_unit="ns")

# Try to parse timestamps as milliseconds -> Won't Work
dfju = pd.read_json(StringIO(json), date_unit="ms")
dfju

# Let pandas detect the correct precision
dfju = pd.read_json(StringIO(json))
dfju

# Or specify that all timestamps are in nanoseconds
dfju = pd.read_json(StringIO(json), date_unit="ns")
dfju
```
By setting the ``dtype_backend`` argument you can control the default dtypes used for the resulting DataFrame.

```python
data = (
 '{"a":{"0":1,"1":3},"b":{"0":2.5,"1":4.5},"c":{"0":true,"1":false},"d":{"0":"a","1":"b"},'
 '"e":{"0":null,"1":6.0},"f":{"0":null,"1":7.5},"g":{"0":null,"1":true},"h":{"0":null,"1":"a"},'
 '"i":{"0":"12-31-2019","1":"12-31-2019"},"j":{"0":null,"1":null}}'
)
df = pd.read_json(StringIO(data), dtype_backend="pyarrow")
df
df.dtypes
```


Normalization
'''''''''''''

pandas provides a utility function to take a dict or list of dicts and *normalize* this semi-structured data
into a flat table.

```python
data = [
    {"id": 1, "name": {"first": "Coleen", "last": "Volk"}},
    {"name": {"given": "Mark", "family": "Regner"}},
    {"id": 2, "name": "Faye Raker"},
]
pd.json_normalize(data)
```
```python
data = [
    {
        "state": "Florida",
        "shortname": "FL",
        "info": {"governor": "Rick Scott"},
        "county": [
            {"name": "Dade", "population": 12345},
            {"name": "Broward", "population": 40000},
            {"name": "Palm Beach", "population": 60000},
        ],
    },
    {
        "state": "Ohio",
        "shortname": "OH",
        "info": {"governor": "John Kasich"},
        "county": [
            {"name": "Summit", "population": 1234},
            {"name": "Cuyahoga", "population": 1337},
        ],
    },
]

pd.json_normalize(data, "county", ["state", "shortname", ["info", "governor"]])
```
The max_level parameter provides more control over which level to end normalization.
With max_level=1 the following snippet normalizes until 1st nesting level of the provided dict.

```python
data = [
    {
        "CreatedBy": {"Name": "User001"},
        "Lookup": {
            "TextField": "Some text",
            "UserField": {"Id": "ID001", "Name": "Name001"},
        },
        "Image": {"a": "b"},
    }
]
pd.json_normalize(data, max_level=1)
```


Line delimited json
'''''''''''''''''''

pandas is able to read and write line-delimited json files that are common in data processing pipelines
using Hadoop or Spark.

For line-delimited json files, pandas can also return an iterator which reads in ``chunksize`` lines at a time. This can be useful for large files or to read from a stream.

```python
from io import StringIO
jsonl = """
    {"a": 1, "b": 2}
    {"a": 3, "b": 4}
"""
df = pd.read_json(StringIO(jsonl), lines=True)
df
df.to_json(orient="records", lines=True)

# reader is an iterator that returns ``chunksize`` lines each iteration
with pd.read_json(StringIO(jsonl), lines=True, chunksize=1) as reader:
    reader
    for chunk in reader:
        print(chunk)
```
Line-limited json can also be read using the pyarrow reader by specifying ``engine="pyarrow"``.

```python
from io import BytesIO
df = pd.read_json(BytesIO(jsonl.encode()), lines=True, engine="pyarrow")
df
```




Table schema
''''''''''''

`Table Schema`_ is a spec for describing tabular datasets as a JSON
object. The JSON includes information on the field names, types, and
other attributes. You can use the orient ``table`` to build
a JSON string with two fields, ``schema`` and ``data``.

```python
df = pd.DataFrame(
    {
        "A": [1, 2, 3],
        "B": ["a", "b", "c"],
        "C": pd.date_range("2016-01-01", freq="D", periods=3),
    },
    index=pd.Index(range(3), name="idx"),
)
df
df.to_json(orient="table", date_format="iso")
```
The ``schema`` field contains the ``fields`` key, which itself contains
a list of column name to type pairs, including the ``Index`` or ``MultiIndex``
(see below for a list of types).
The ``schema`` field also contains a ``primaryKey`` field if the (Multi)index
is unique.

The second field, ``data``, contains the serialized data with the ``records``
orient.
The index is included, and any datetimes are ISO 8601 formatted, as required
by the Table Schema spec.

The full list of types supported are described in the Table Schema
spec. This table shows the mapping from pandas types:

=============== =================
pandas type     Table Schema type
=============== =================
int64           integer
float64         number
bool            boolean
datetime64[ns]  datetime
timedelta64[ns] duration
categorical     any
object          str
=============== =================

A few notes on the generated table schema:

* The ``schema`` object contains a ``pandas_version`` field. This contains
  the version of pandas' dialect of the schema, and will be incremented
  with each revision.
* All dates are converted to UTC when serializing. Even timezone naive values,
  which are treated as UTC with an offset of 0.

  ```python
from pandas.io.json import build_table_schema

s = pd.Series(pd.date_range("2016", periods=4))
build_table_schema(s)
```
* datetimes with a timezone (before serializing), include an additional field
  ``tz`` with the time zone name (e.g. ``'US/Central'``).

  ```python
s_tz = pd.Series(pd.date_range("2016", periods=12, tz="US/Central"))
build_table_schema(s_tz)
```
* Periods are converted to timestamps before serialization, and so have the
  same behavior of being converted to UTC. In addition, periods will contain
  and additional field ``freq`` with the period's frequency, e.g. ``'A-DEC'``.

  ```python
s_per = pd.Series(1, index=pd.period_range("2016", freq="Y-DEC", periods=4))
build_table_schema(s_per)
```
* Categoricals use the ``any`` type and an ``enum`` constraint listing
  the set of possible values. Additionally, an ``ordered`` field is included:

  ```python
s_cat = pd.Series(pd.Categorical(["a", "b", "a"]))
build_table_schema(s_cat)
```
* A ``primaryKey`` field, containing an array of labels, is included
  *if the index is unique*:

  ```python
s_dupe = pd.Series([1, 2], index=[1, 1])
build_table_schema(s_dupe)
```
* The ``primaryKey`` behavior is the same with MultiIndexes, but in this
  case the ``primaryKey`` is an array:

  ```python
s_multi = pd.Series(1, index=pd.MultiIndex.from_product([("a", "b"), (0, 1)]))
build_table_schema(s_multi)
```
* The default naming roughly follows these rules:

    - For series, the ``object.name`` is used. If that's none, then the
      name is ``values``
    - For ``DataFrames``, the stringified version of the column name is used
    - For ``Index`` (not ``MultiIndex``), ``index.name`` is used, with a
      fallback to ``index`` if that is None.
    - For ``MultiIndex``, ``mi.names`` is used. If any level has no name,
      then ``level_<i>`` is used.

``read_json`` also accepts ``orient='table'`` as an argument. This allows for
the preservation of metadata such as dtypes and index names in a
round-trippable manner.

```python
df = pd.DataFrame(
    {
        "foo": [1, 2, 3, 4],
        "bar": ["a", "b", "c", "d"],
        "baz": pd.date_range("2018-01-01", freq="D", periods=4),
        "qux": pd.Categorical(["a", "b", "c", "c"]),
    },
    index=pd.Index(range(4), name="idx"),
)
df
df.dtypes

df.to_json("test.json", orient="table")
new_df = pd.read_json("test.json", orient="table")
new_df
new_df.dtypes
```
Please note that the literal string 'index' as the name of an `Index`
is not round-trippable, nor are any names beginning with ``'level_'`` within a
`MultiIndex`. These are used by default in `DataFrame.to_json` to
indicate missing values and the subsequent read cannot distinguish the intent.

```python
:okwarning:

df.index.name = "index"
df.to_json("test.json", orient="table")
new_df = pd.read_json("test.json", orient="table")
print(new_df.index.name)
```
```python
:suppress:

os.remove("test.json")
```
When using ``orient='table'`` along with user-defined ``ExtensionArray``,
the generated schema will contain an additional ``extDtype`` key in the respective
``fields`` element. This extra key is not standard but does enable JSON roundtrips
for extension types (e.g. ``read_json(df.to_json(orient="table"), orient="table")``).

The ``extDtype`` key carries the name of the extension, if you have properly registered
the ``ExtensionDtype``, pandas will use said name to perform a lookup into the registry
and re-convert the serialized data into your custom dtype.

.. _Table Schema: https://specs.frictionlessdata.io/table-schema/


## HTML


Reading HTML content
''''''''''''''''''''''

> **warning.capitalize():**
   We **highly encourage** you to read the `HTML Table Parsing gotchas <io.html.gotchas>`
   below regarding the issues surrounding the BeautifulSoup4/html5lib/lxml parsers.

The top-level `~pandas.io.html.read_html` function can accept an HTML
string/file/URL and will parse HTML tables into list of pandas ``DataFrames``.
Let's look at a few examples.

> **note.capitalize():**
   ``read_html`` returns a ``list`` of ``DataFrame`` objects, even if there is
   only a single table contained in the HTML content.

Read a URL with no options:



   In [320]: url = "https://www.fdic.gov/resources/resolutions/bank-failures/failed-bank-list"

   In [321]: pd.read_html(url)
   Out[321]:
   [                         Bank NameBank           CityCity StateSt  ...              Acquiring InstitutionAI Closing DateClosing FundFund
    0                    Almena State Bank             Almena      KS  ...                          Equity Bank    October 23, 2020    10538
    1           First City Bank of Florida  Fort Walton Beach      FL  ...            United Fidelity Bank, fsb    October 16, 2020    10537
    2                 The First State Bank      Barboursville      WV  ...                       MVB Bank, Inc.       April 3, 2020    10536
    3                   Ericson State Bank            Ericson      NE  ...           Farmers and Merchants Bank   February 14, 2020    10535
    4     City National Bank of New Jersey             Newark      NJ  ...                      Industrial Bank    November 1, 2019    10534
    ..                                 ...                ...     ...  ...                                  ...                 ...      ...
    558                 Superior Bank, FSB           Hinsdale      IL  ...                Superior Federal, FSB       July 27, 2001     6004
    559                Malta National Bank              Malta      OH  ...                    North Valley Bank         May 3, 2001     4648
    560    First Alliance Bank & Trust Co.         Manchester      NH  ...  Southern New Hampshire Bank & Trust    February 2, 2001     4647
    561  National State Bank of Metropolis         Metropolis      IL  ...              Banterra Bank of Marion   December 14, 2000     4646
    562                   Bank of Honolulu           Honolulu      HI  ...                   Bank of the Orient    October 13, 2000     4645

    [563 rows x 7 columns]]

> **note.capitalize():**
   The data from the above URL changes every Monday so the resulting data above may be slightly different.

Read a URL while passing headers alongside the HTTP request:



   In [322]: url = 'https://www.sump.org/notes/request/' # HTTP request reflector

   In [323]: pd.read_html(url)
   Out[323]:
   [                   0                    1
    0     Remote Socket:  51.15.105.256:51760
    1  Protocol Version:             HTTP/1.1
    2    Request Method:                  GET
    3       Request URI:      /notes/request/
    4     Request Query:                  NaN,
    0   Accept-Encoding:             identity
    1              Host:         www.sump.org
    2        User-Agent:    Python-urllib/3.8
    3        Connection:                close]

   In [324]: headers = {
      .....:    'User-Agent':'Mozilla Firefox v14.0',
      .....:    'Accept':'application/json',
      .....:    'Connection':'keep-alive',
      .....:    'Auth':'Bearer 2*/f3+fe68df*4'
      .....: }

   In [325]: pd.read_html(url, storage_options=headers)
   Out[325]:
   [                   0                    1
    0     Remote Socket:  51.15.105.256:51760
    1  Protocol Version:             HTTP/1.1
    2    Request Method:                  GET
    3       Request URI:      /notes/request/
    4     Request Query:                  NaN,
    0        User-Agent: Mozilla Firefox v14.0
    1    AcceptEncoding:   gzip,  deflate,  br
    2            Accept:      application/json
    3        Connection:             keep-alive
    4              Auth:  Bearer 2*/f3+fe68df*4]

> **note.capitalize():**
   We see above that the headers we passed are reflected in the HTTP request.

Read in the content of the file from the above URL and pass it to ``read_html``
as a string:

```python
html_str = """
         <table>
             <tr>
                 <th>A</th>
                 <th colspan="1">B</th>
                 <th rowspan="1">C</th>
             </tr>
             <tr>
                 <td>a</td>
                 <td>b</td>
                 <td>c</td>
             </tr>
         </table>
     """

with open("tmp.html", "w") as f:
    f.write(html_str)
df = pd.read_html("tmp.html")
df[0]
[``
```python
:suppress:

os.remove("tmp.html")
```
You can even pass in an instance of ``StringIO`` if you so desire:

```python
dfs = pd.read_html(StringIO(html_str))
dfs[0]
```
> **note.capitalize():**
   The following examples are not run by the IPython evaluator due to the fact
   that having so many network-accessing functions slows down the documentation
   build. If you spot an error or an example that doesn't run, please do not
   hesitate to report it over on `pandas GitHub issues page
  ](https://github.com/pandas-dev/pandas/issues)_.


Read a URL and match a table that contains specific text:

```python
match = "Metcalf Bank"
df_list = pd.read_html(url, match=match)
```
Specify a header row (by default ``<th>`` or ``<td>`` elements located within a
``<thead>`` are used to form the column index, if multiple rows are contained within
``<thead>`` then a MultiIndex is created); if specified, the header row is taken
from the data minus the parsed header elements (``<th>`` elements).

```python
dfs = pd.read_html(url, header=0)
```
Specify an index column:

```python
dfs = pd.read_html(url, index_col=0)
```
Specify a number of rows to skip:

```python
dfs = pd.read_html(url, skiprows=0)
```
Specify a number of rows to skip using a list (``range`` works
as well):

```python
dfs = pd.read_html(url, skiprows=range(2))
```
Specify an HTML attribute:

```python
dfs1 = pd.read_html(url, attrs={"id": "table"})
dfs2 = pd.read_html(url, attrs={"class": "sortable"})
print(np.array_equal(dfs1[0], dfs2[0]))  # Should be True
```
Specify values that should be converted to NaN:

```python
dfs = pd.read_html(url, na_values=["No Acquirer"])
```
Specify whether to keep the default set of NaN values:

```python
dfs = pd.read_html(url, keep_default_na=False)
```
Specify converters for columns. This is useful for numerical text data that has
leading zeros.  By default columns that are numerical are cast to numeric
types and the leading zeros are lost. To avoid this, we can convert these
columns to strings.

```python
url_mcc = "https://en.wikipedia.org/wiki/Mobile_country_code?oldid=899173761"
dfs = pd.read_html(
    url_mcc,
    match="Telekom Albania",
    header=0,
    converters={"MNC": str},
)
```
Use some combination of the above:

```python
dfs = pd.read_html(url, match="Metcalf Bank", index_col=0)
```
Read in pandas ``to_html`` output (with some loss of floating point precision):

```python
df = pd.DataFrame(np.random.randn(2, 2))
s = df.to_html(float_format="{0:.40g}".format)
dfin = pd.read_html(s, index_col=0)
```
The ``lxml`` backend will raise an error on a failed parse if that is the only
parser you provide. If you only have a single parser you can provide just a
string, but it is considered good practice to pass a list with one string if,
for example, the function expects a sequence of strings. You may use:

```python
dfs = pd.read_html(url, "Metcalf Bank", index_col=0, flavor=["lxml"])
```
Or you could pass ``flavor='lxml'`` without a list:

```python
dfs = pd.read_html(url, "Metcalf Bank", index_col=0, flavor="lxml")
```
However, if you have bs4 and html5lib installed and pass ``None`` or ``['lxml',
'bs4']`` then the parse will most likely succeed. Note that *as soon as a parse
succeeds, the function will return*.

```python
dfs = pd.read_html(url, "Metcalf Bank", index_col=0, flavor=["lxml", "bs4"])
```
Links can be extracted from cells along with the text using ``extract_links="all"``.

```python
html_table = """
<table>
  <tr>
    <th>GitHub</th>
  </tr>
  <tr>
    <td><a href="https://github.com/pandas-dev/pandas">pandas</a></td>
  </tr>
</table>
"""

df = pd.read_html(
    StringIO(html_table),
    extract_links="all"
)[0]
df
df[("GitHub", None)]
df[("GitHub", None)].str[1]
```




Writing to HTML files
''''''''''''''''''''''

``DataFrame`` objects have an instance method ``to_html`` which renders the
contents of the ``DataFrame`` as an HTML table. The function arguments are as
in the method ``to_string`` described above.

> **note.capitalize():**
   Not all of the possible options for ``DataFrame.to_html`` are shown here for
   brevity's sake. See `.DataFrame.to_html` for the
   full set of options.

> **note.capitalize():**
   In an HTML-rendering supported environment like a Jupyter Notebook, ``display(HTML(...))```
   will render the raw HTML into the environment.

```python
from IPython.display import display, HTML

df = pd.DataFrame(np.random.randn(2, 2))
df
html = df.to_html()
print(html)  # raw html
display(HTML(html))
```
The ``columns`` argument will limit the columns shown:

```python
html = df.to_html(columns=[0])
print(html)
display(HTML(html))
```
``float_format`` takes a Python callable to control the precision of floating
point values:

```python
html = df.to_html(float_format="{0:.10f}".format)
print(html)
display(HTML(html))
```
``bold_rows`` will make the row labels bold by default, but you can turn that
off:

```python
html = df.to_html(bold_rows=False)
print(html)
display(HTML(html))
```
The ``classes`` argument provides the ability to give the resulting HTML
table CSS classes. Note that these classes are *appended* to the existing
``'dataframe'`` class.

```python
print(df.to_html(classes=["awesome_table_class", "even_more_awesome_class"]))
```
The ``render_links`` argument provides the ability to add hyperlinks to cells
that contain URLs.

```python
url_df = pd.DataFrame(
    {
        "name": ["Python", "pandas"],
        "url": ["https://www.python.org/", "https://pandas.pydata.org"],
    }
)
html = url_df.to_html(render_links=True)
print(html)
display(HTML(html))
```
Finally, the ``escape`` argument allows you to control whether the
"<", ">" and "&" characters escaped in the resulting HTML (by default it is
``True``). So to get the HTML without escaped characters pass ``escape=False``

```python
df = pd.DataFrame({"a": list("&<>"), "b": np.random.randn(3)})
[``
Escaped:

```python
html = df.to_html()
print(html)
display(HTML(html))
```
Not escaped:

```python
html = df.to_html(escape=False)
print(html)
display(HTML(html))
```
> **note.capitalize():**
   Some browsers may not show a difference in the rendering of the previous two
   HTML tables.




HTML Table Parsing Gotchas
''''''''''''''''''''''''''

There are some versioning issues surrounding the libraries that are used to
parse HTML tables in the top-level pandas io function ``read_html``.

**Issues with** |lxml|_

* Benefits

    - |lxml|_ is very fast.

    - |lxml|_ requires Cython to install correctly.

* Drawbacks

    - |lxml|_ does *not* make any guarantees about the results of its parse
      *unless* it is given |svm|_.

    - In light of the above, we have chosen to allow you, the user, to use the
      |lxml|_ backend, but **this backend will use** |html5lib|_ if |lxml|_
      fails to parse

    - It is therefore *highly recommended* that you install both
      |BeautifulSoup4|_ and |html5lib|_, so that you will still get a valid
      result (provided everything else is valid) even if |lxml|_ fails.

**Issues with** |BeautifulSoup4|_ **using** |lxml|_ **as a backend**

* The above issues hold here as well since |BeautifulSoup4|_ is essentially
  just a wrapper around a parser backend.

**Issues with** |BeautifulSoup4|_ **using** |html5lib|_ **as a backend**

* Benefits

    - |html5lib|_ is far more lenient than |lxml|_ and consequently deals
      with *real-life markup* in a much saner way rather than just, e.g.,
      dropping an element without notifying you.

    - |html5lib|_ *generates valid HTML5 markup from invalid markup
      automatically*. This is extremely important for parsing HTML tables,
      since it guarantees a valid document. However, that does NOT mean that
      it is "correct", since the process of fixing markup does not have a
      single definition.

    - |html5lib|_ is pure Python and requires no additional build steps beyond
      its own installation.

* Drawbacks

    - The biggest drawback to using |html5lib|_ is that it is slow as
      molasses.  However consider the fact that many tables on the web are not
      big enough for the parsing algorithm runtime to matter. It is more
      likely that the bottleneck will be in the process of reading the raw
      text from the URL over the web, i.e., IO (input-output). For very large
      tables, this might not be true.



.. _svm: https://validator.w3.org/docs/help.html#validation_basics


.. _html5lib: https://github.com/html5lib/html5lib-python


.. _BeautifulSoup4: https://www.crummy.com/software/BeautifulSoup


.. _lxml: https://lxml.de



## LaTeX


Currently there are no methods to read from LaTeX, only output methods.

Writing to LaTeX files
''''''''''''''''''''''

> **note.capitalize():**
   DataFrame *and* Styler objects currently have a ``to_latex`` method. We recommend
   using the `Styler.to_latex()](../reference/api/pandas.io.formats.style.Styler.to_latex.rst)_ method
   over [DataFrame.to_latex()](../reference/api/pandas.DataFrame.to_latex.rst)_ due to the former's greater flexibility with
   conditional styling, and the latter's possible future deprecation.

Review the documentation for [Styler.to_latex](../reference/api/pandas.io.formats.style.Styler.to_latex.rst)_,
which gives examples of conditional styling and explains the operation of its keyword
arguments.

For simple application the following pattern is sufficient.

[``python
df = pd.DataFrame([[1, 2], [3, 4]], index=["a", "b"], columns=["c", "d"])
print(df.style.to_latex())
```
To format values before output, chain the `Styler.format](../reference/api/pandas.io.formats.style.Styler.format.rst)_
method.

```python
print(df.style.format("€ {}").to_latex())
```
## XML


Reading XML
'''''''''''



The top-level `~pandas.io.xml.read_xml` function can accept an XML
string/file/URL and will parse nodes and attributes into a pandas ``DataFrame``.

> **note.capitalize():**
   Since there is no standard XML structure where design types can vary in
   many ways, ``read_xml`` works best with flatter, shallow versions. If
   an XML document is deeply nested, use the ``stylesheet`` feature to
   transform XML into a flatter version.

Let's look at a few examples.

Read an XML string:

```python
from io import StringIO
xml = """<?xml version="1.0" encoding="UTF-8"?>
<bookstore>
  <book category="cooking">
    <title lang="en">Everyday Italian</title>
    <author>Giada De Laurentiis</author>
    <year>2005</year>
    <price>30.00</price>
  </book>
  <book category="children">
    <title lang="en">Harry Potter</title>
    <author>J K. Rowling</author>
    <year>2005</year>
    <price>29.99</price>
  </book>
  <book category="web">
    <title lang="en">Learning XML</title>
    <author>Erik T. Ray</author>
    <year>2003</year>
    <price>39.95</price>
  </book>
</bookstore>"""

df = pd.read_xml(StringIO(xml))
df
```
Read a URL with no options:

```python
df = pd.read_xml("https://www.w3schools.com/xml/books.xml")
df
```
Read in the content of the "books.xml" file and pass it to ``read_xml``
as a string:

```python
file_path = "books.xml"
with open(file_path, "w") as f:
    f.write(xml)

with open(file_path, "r") as f:
    df = pd.read_xml(StringIO(f.read()))
df
```
Read in the content of the "books.xml" as instance of ``StringIO`` or
``BytesIO`` and pass it to ``read_xml``:

```python
with open(file_path, "r") as f:
    sio = StringIO(f.read())

df = pd.read_xml(sio)
df
```
```python
with open(file_path, "rb") as f:
    bio = BytesIO(f.read())

df = pd.read_xml(bio)
df
```
Even read XML from AWS S3 buckets such as NIH NCBI PMC Article Datasets providing
Biomedical and Life Science Journals:

```python
>>> df = pd.read_xml(
...    "s3://pmc-oa-opendata/oa_comm/xml/all/PMC1236943.xml",
...    xpath=".//journal-meta",
...)
>>> df
      journal-id  journal-title  issn  publisher
0 Cardiovasc Ultrasound Cardiovascular Ultrasound 1476-7120 NaN
```
With `lxml`_ as default ``parser``, you access the full-featured XML library
that extends Python's ElementTree API. One powerful tool is ability to query
nodes selectively or conditionally with more expressive XPath:

.. _lxml: https://lxml.de

```python
df = pd.read_xml(file_path, xpath="//book[year=2005]")
df
```
Specify only elements or only attributes to parse:

```python
df = pd.read_xml(file_path, elems_only=True)
df
```
```python
df = pd.read_xml(file_path, attrs_only=True)
df
```
```python
:suppress:

os.remove("books.xml")
```
XML documents can have namespaces with prefixes and default namespaces without
prefixes both of which are denoted with a special attribute ``xmlns``. In order
to parse by node under a namespace context, ``xpath`` must reference a prefix.

For example, below XML contains a namespace with prefix, ``doc``, and URI at
``https://example.com``. In order to parse ``doc:row`` nodes,
``namespaces`` must be used.

```python
xml = """<?xml version='1.0' encoding='utf-8'?>
<doc:data xmlns:doc="https://example.com">
  <doc:row>
    <doc:shape>square</doc:shape>
    <doc:degrees>360</doc:degrees>
    <doc:sides>4.0</doc:sides>
  </doc:row>
  <doc:row>
    <doc:shape>circle</doc:shape>
    <doc:degrees>360</doc:degrees>
    <doc:sides/>
  </doc:row>
  <doc:row>
    <doc:shape>triangle</doc:shape>
    <doc:degrees>180</doc:degrees>
    <doc:sides>3.0</doc:sides>
  </doc:row>
</doc:data>"""

df = pd.read_xml(StringIO(xml),
                 xpath="//doc:row",
                 namespaces={"doc": "https://example.com"})
df
```
Similarly, an XML document can have a default namespace without prefix. Failing
to assign a temporary prefix will return no nodes and raise a ``ValueError``.
But assigning *any* temporary name to correct URI allows parsing by nodes.

```python
xml = """<?xml version='1.0' encoding='utf-8'?>
<data xmlns="https://example.com">
 <row>
   <shape>square</shape>
   <degrees>360</degrees>
   <sides>4.0</sides>
 </row>
 <row>
   <shape>circle</shape>
   <degrees>360</degrees>
   <sides/>
 </row>
 <row>
   <shape>triangle</shape>
   <degrees>180</degrees>
   <sides>3.0</sides>
 </row>
</data>"""

df = pd.read_xml(StringIO(xml),
                 xpath="//pandas:row",
                 namespaces={"pandas": "https://example.com"})
df
```
However, if XPath does not reference node names such as default, ``/*``, then
``namespaces`` is not required.

> **note.capitalize():**
   Since ``xpath`` identifies the parent of content to be parsed, only immediate
   descendants which include child nodes or current attributes are parsed.
   Therefore, ``read_xml`` will not parse the text of grandchildren or other
   descendants and will not parse attributes of any descendant. To retrieve
   lower level content, adjust xpath to lower level. For example,

   ```python
:okwarning:

   xml = """
   <data>
     <row>
       <shape sides="4">square</shape>
       <degrees>360</degrees>
     </row>
     <row>
       <shape sides="0">circle</shape>
       <degrees>360</degrees>
     </row>
     <row>
       <shape sides="3">triangle</shape>
       <degrees>180</degrees>
     </row>
   </data>"""

   df = pd.read_xml(StringIO(xml), xpath="./row")
   df

shows the attribute ``sides`` on ``shape`` element was not parsed as
expected since this attribute resides on the child of ``row`` element
and not ``row`` element itself. In other words, ``sides`` attribute is a
grandchild level descendant of ``row`` element. However, the ``xpath``
targets ``row`` element which covers only its children and attributes.
```
With `lxml`_ as parser, you can flatten nested XML documents with an XSLT
script which also can be string/file/URL types. As background, `XSLT`_ is
a special-purpose language written in a special XML file that can transform
original XML documents into other XML, HTML, even text (CSV, JSON, etc.)
using an XSLT processor.

.. _lxml: https://lxml.de
.. _XSLT: https://www.w3.org/TR/xslt/

For example, consider this somewhat nested structure of Chicago "L" Rides
where station and rides elements encapsulate data in their own sections.
With below XSLT, ``lxml`` can transform original nested document into a flatter
output (as shown below for demonstration) for easier parse into ``DataFrame``:

```python
xml = """<?xml version='1.0' encoding='utf-8'?>
 <response>
  <row>
    <station id="40850" name="Library"/>
    <month>2020-09-01T00:00:00</month>
    <rides>
      <avg_weekday_rides>864.2</avg_weekday_rides>
      <avg_saturday_rides>534</avg_saturday_rides>
      <avg_sunday_holiday_rides>417.2</avg_sunday_holiday_rides>
    </rides>
  </row>
  <row>
    <station id="41700" name="Washington/Wabash"/>
    <month>2020-09-01T00:00:00</month>
    <rides>
      <avg_weekday_rides>2707.4</avg_weekday_rides>
      <avg_saturday_rides>1909.8</avg_saturday_rides>
      <avg_sunday_holiday_rides>1438.6</avg_sunday_holiday_rides>
    </rides>
  </row>
  <row>
    <station id="40380" name="Clark/Lake"/>
    <month>2020-09-01T00:00:00</month>
    <rides>
      <avg_weekday_rides>2949.6</avg_weekday_rides>
      <avg_saturday_rides>1657</avg_saturday_rides>
      <avg_sunday_holiday_rides>1453.8</avg_sunday_holiday_rides>
    </rides>
  </row>
 </response>"""

xsl = """<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
   <xsl:output method="xml" omit-xml-declaration="no" indent="yes"/>
   <xsl:strip-space elements="*"/>
   <xsl:template match="/response">
      <xsl:copy>
        <xsl:apply-templates select="row"/>
      </xsl:copy>
   </xsl:template>
   <xsl:template match="row">
      <xsl:copy>
        <station_id><xsl:value-of select="station/@id"/></station_id>
        <station_name><xsl:value-of select="station/@name"/></station_name>
        <xsl:copy-of select="month|rides/*"/>
      </xsl:copy>
   </xsl:template>
 </xsl:stylesheet>"""

output = """<?xml version='1.0' encoding='utf-8'?>
 <response>
   <row>
      <station_id>40850</station_id>
      <station_name>Library</station_name>
      <month>2020-09-01T00:00:00</month>
      <avg_weekday_rides>864.2</avg_weekday_rides>
      <avg_saturday_rides>534</avg_saturday_rides>
      <avg_sunday_holiday_rides>417.2</avg_sunday_holiday_rides>
   </row>
   <row>
      <station_id>41700</station_id>
      <station_name>Washington/Wabash</station_name>
      <month>2020-09-01T00:00:00</month>
      <avg_weekday_rides>2707.4</avg_weekday_rides>
      <avg_saturday_rides>1909.8</avg_saturday_rides>
      <avg_sunday_holiday_rides>1438.6</avg_sunday_holiday_rides>
   </row>
   <row>
      <station_id>40380</station_id>
      <station_name>Clark/Lake</station_name>
      <month>2020-09-01T00:00:00</month>
      <avg_weekday_rides>2949.6</avg_weekday_rides>
      <avg_saturday_rides>1657</avg_saturday_rides>
      <avg_sunday_holiday_rides>1453.8</avg_sunday_holiday_rides>
   </row>
 </response>"""

df = pd.read_xml(StringIO(xml), stylesheet=StringIO(xsl))
df
```
For very large XML files that can range in hundreds of megabytes to gigabytes, `pandas.read_xml`
supports parsing such sizeable files using `lxml's iterparse`_ and `etree's iterparse`_
which are memory-efficient methods to iterate through an XML tree and extract specific elements and attributes.
without holding entire tree in memory.



.. _`lxml's iterparse`: https://lxml.de/3.2/parsing.html#iterparse-and-iterwalk
.. _`etree's iterparse`: https://docs.python.org/3/library/xml.etree.elementtree.html#xml.etree.ElementTree.iterparse

To use this feature, you must pass a physical XML file path into ``read_xml`` and use the ``iterparse`` argument.
Files should not be compressed or point to online sources but stored on local disk. Also, ``iterparse`` should be
a dictionary where the key is the repeating nodes in document (which become the rows) and the value is a list of
any element or attribute that is a descendant (i.e., child, grandchild) of repeating node. Since XPath is not
used in this method, descendants do not need to share same relationship with one another. Below shows example
of reading in Wikipedia's very large (12 GB+) latest article data dump.



    In [1]: df = pd.read_xml(
    ...         "/path/to/downloaded/enwikisource-latest-pages-articles.xml",
    ...         iterparse = {"page": ["title", "ns", "id"]}
    ...     )
    ...     df
    Out[2]:
                                                         title   ns        id
    0                                       Gettysburg Address    0     21450
    1                                                Main Page    0     42950
    2                            Declaration by United Nations    0      8435
    3             Constitution of the United States of America    0      8435
    4                     Declaration of Independence (Israel)    0     17858
    ...                                                    ...  ...       ...
    3578760               Page:Black cat 1897 07 v2 n10.pdf/17  104    219649
    3578761               Page:Black cat 1897 07 v2 n10.pdf/43  104    219649
    3578762               Page:Black cat 1897 07 v2 n10.pdf/44  104    219649
    3578763      The History of Tom Jones, a Foundling/Book IX    0  12084291
    3578764  Page:Shakespeare of Stratford (1926) Yale.djvu/91  104     21450

    [3578765 rows x 3 columns]



Writing XML
'''''''''''



``DataFrame`` objects have an instance method ``to_xml`` which renders the
contents of the ``DataFrame`` as an XML document.

> **note.capitalize():**
   This method does not support special properties of XML including DTD,
   CData, XSD schemas, processing instructions, comments, and others.
   Only namespaces at the root level is supported. However, ``stylesheet``
   allows design changes after initial output.

Let's look at a few examples.

Write an XML without options:

```python
geom_df = pd.DataFrame(
    {
        "shape": ["square", "circle", "triangle"],
        "degrees": [360, 360, 180],
        "sides": [4, np.nan, 3],
    }
)

print(geom_df.to_xml())
```
Write an XML with new root and row name:

```python
print(geom_df.to_xml(root_name="geometry", row_name="objects"))
```
Write an attribute-centric XML:

```python
print(geom_df.to_xml(attr_cols=geom_df.columns.tolist()))
```
Write a mix of elements and attributes:

```python
print(
    geom_df.to_xml(
        index=False,
        attr_cols=['shape'],
        elem_cols=['degrees', 'sides'])
)
```
Any ``DataFrames`` with hierarchical columns will be flattened for XML element names
with levels delimited by underscores:

```python
ext_geom_df = pd.DataFrame(
    {
        "type": ["polygon", "other", "polygon"],
        "shape": ["square", "circle", "triangle"],
        "degrees": [360, 360, 180],
        "sides": [4, np.nan, 3],
    }
)

pvt_df = ext_geom_df.pivot_table(index='shape',
                                 columns='type',
                                 values=['degrees', 'sides'],
                                 aggfunc='sum')
pvt_df

print(pvt_df.to_xml())
```
Write an XML with default namespace:

```python
print(geom_df.to_xml(namespaces={"": "https://example.com"}))
```
Write an XML with namespace prefix:

```python
print(
    geom_df.to_xml(namespaces={"doc": "https://example.com"},
                   prefix="doc")
)
```
Write an XML without declaration or pretty print:

```python
print(
    geom_df.to_xml(xml_declaration=False,
                   pretty_print=False)
)
```
Write an XML and transform with stylesheet:

```python
xsl = """<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
   <xsl:output method="xml" omit-xml-declaration="no" indent="yes"/>
   <xsl:strip-space elements="*"/>
   <xsl:template match="/data">
     <geometry>
       <xsl:apply-templates select="row"/>
     </geometry>
   </xsl:template>
   <xsl:template match="row">
     <object index="{index}">
       <xsl:if test="shape!='circle'">
           <xsl:attribute name="type">polygon</xsl:attribute>
       </xsl:if>
       <xsl:copy-of select="shape"/>
       <property>
           <xsl:copy-of select="degrees|sides"/>
       </property>
     </object>
   </xsl:template>
 </xsl:stylesheet>"""

print(geom_df.to_xml(stylesheet=StringIO(xsl)))
```
XML Final Notes
'''''''''''''''

* All XML documents adhere to `W3C specifications`_. Both ``etree`` and ``lxml``
  parsers will fail to parse any markup document that is not well-formed or
  follows XML syntax rules. Do be aware HTML is not an XML document unless it
  follows XHTML specs. However, other popular markup types including KML, XAML,
  RSS, MusicML, MathML are compliant `XML schemas`_.

* For above reason, if your application builds XML prior to pandas operations,
  use appropriate DOM libraries like ``etree`` and ``lxml`` to build the necessary
  document and not by string concatenation or regex adjustments. Always remember
  XML is a *special* text file with markup rules.

* With very large XML files (several hundred MBs to GBs), XPath and XSLT
  can become memory-intensive operations. Be sure to have enough available
  RAM for reading and writing to large XML files (roughly about 5 times the
  size of text).

* Because XSLT is a programming language, use it with caution since such scripts
  can pose a security risk in your environment and can run large or infinite
  recursive operations. Always test scripts on small fragments before full run.

* The `etree`_ parser supports all functionality of both ``read_xml`` and
  ``to_xml`` except for complex XPath and any XSLT. Though limited in features,
  ``etree`` is still a reliable and capable parser and tree builder. Its
  performance may trail ``lxml`` to a certain degree for larger files but
  relatively unnoticeable on small to medium size files.

.. _`W3C specifications`: https://www.w3.org/TR/xml/
.. _`XML schemas`: https://en.wikipedia.org/wiki/List_of_types_of_XML_schemas
.. _`etree`: https://docs.python.org/3/library/xml.etree.elementtree.html





## Excel files
The `~pandas.read_excel` method can read Excel 2007+ (``.xlsx``) files
using the ``openpyxl`` Python module. Excel 2003 (``.xls``) files
can be read using ``xlrd``. Binary Excel (``.xlsb``)
files can be read using ``pyxlsb``. All formats can be read
using `calamine<io.calamine>` engine.
The `~DataFrame.to_excel` instance method is used for
saving a ``DataFrame`` to Excel.  Generally the semantics are
similar to working with `csv<io.read_csv_table>` data.
See the `cookbook<cookbook.excel>[ for some advanced strategies.

> **note.capitalize():**
   When ``engine=None``, the following logic will be used to determine the engine:

   - If ``path_or_buffer`` is an OpenDocument format (.odf, .ods, .odt),
     then `odf](https://pypi.org/project/odfpy/) will be used.
   - Otherwise if [`path_or_buffer`` is an xls format, ``xlrd`` will be used.
   - Otherwise if ``path_or_buffer`` is in xlsb format, ``pyxlsb`` will be used.
   - Otherwise ``openpyxl`` will be used.



Reading Excel files
'''''''''''''''''''

In the most basic use-case, ``read_excel`` takes a path to an Excel
file, and the ``sheet_name`` indicating which sheet to parse.

When using the ``engine_kwargs`` parameter, pandas will pass these arguments to the
engine. For this, it is important to know which function pandas is
using internally.

* For the engine openpyxl, pandas is using `openpyxl.load_workbook` to read in (``.xlsx``) and (``.xlsm``) files.

* For the engine xlrd, pandas is using `xlrd.open_workbook` to read in (``.xls``) files.

* For the engine pyxlsb, pandas is using `pyxlsb.open_workbook` to read in (``.xlsb``) files.

* For the engine odf, pandas is using `odf.opendocument.load` to read in (``.ods``) files.

* For the engine calamine, pandas is using `python_calamine.load_workbook`
  to read in (``.xlsx``), (``.xlsm``), (``.xls``), (``.xlsb``), (``.ods``) files.

```python
# Returns a DataFrame
pd.read_excel("path_to_file.xls", sheet_name="Sheet1")
```


``ExcelFile`` class
+++++++++++++++++++

To facilitate working with multiple sheets from the same file, the ``ExcelFile``
class can be used to wrap the file and can be passed into ``read_excel``
There will be a performance benefit for reading multiple sheets as the file is
read into memory only once.

```python
xlsx = pd.ExcelFile("path_to_file.xls")
df = pd.read_excel(xlsx, "Sheet1")
```
The ``ExcelFile`` class can also be used as a context manager.

```python
with pd.ExcelFile("path_to_file.xls") as xls:
    df1 = pd.read_excel(xls, "Sheet1")
    df2 = pd.read_excel(xls, "Sheet2")
```
The ``sheet_names`` property will generate
a list of the sheet names in the file.

The primary use-case for an ``ExcelFile`` is parsing multiple sheets with
different parameters:

```python
data = {}
# For when Sheet1's format differs from Sheet2
with pd.ExcelFile("path_to_file.xls") as xls:
    data["Sheet1"] = pd.read_excel(xls, "Sheet1", index_col=None, na_values=["NA"])
    data["Sheet2"] = pd.read_excel(xls, "Sheet2", index_col=1)
```
Note that if the same parsing parameters are used for all sheets, a list
of sheet names can simply be passed to ``read_excel`` with no loss in performance.

```python
# using the ExcelFile class
data = {}
with pd.ExcelFile("path_to_file.xls") as xls:
    data["Sheet1"] = pd.read_excel(xls, "Sheet1", index_col=None, na_values=["NA"])
    data["Sheet2"] = pd.read_excel(xls, "Sheet2", index_col=None, na_values=["NA"])

# equivalent using the read_excel function
data = pd.read_excel(
    "path_to_file.xls", ["Sheet1", "Sheet2"], index_col=None, na_values=["NA"]
)
```
``ExcelFile`` can also be called with a ``xlrd.book.Book`` object
as a parameter. This allows the user to control how the excel file is read.
For example, sheets can be loaded on demand by calling ``xlrd.open_workbook()``
with ``on_demand=True``.

```python
import xlrd

xlrd_book = xlrd.open_workbook("path_to_file.xls", on_demand=True)
with pd.ExcelFile(xlrd_book) as xls:
    df1 = pd.read_excel(xls, "Sheet1")
    df2 = pd.read_excel(xls, "Sheet2")
```


Specifying sheets
+++++++++++++++++





* The arguments ``sheet_name`` allows specifying the sheet or sheets to read.
* The default value for ``sheet_name`` is 0, indicating to read the first sheet
* Pass a string to refer to the name of a particular sheet in the workbook.
* Pass an integer to refer to the index of a sheet. Indices follow Python
  convention, beginning at 0.
* Pass a list of either strings or integers, to return a dictionary of specified sheets.
* Pass a ``None`` to return a dictionary of all available sheets.

```python
# Returns a DataFrame
pd.read_excel("path_to_file.xls", "Sheet1", index_col=None, na_values=["NA"])
```
Using the sheet index:

```python
# Returns a DataFrame
pd.read_excel("path_to_file.xls", 0, index_col=None, na_values=["NA"])
```
Using all default values:

```python
# Returns a DataFrame
pd.read_excel("path_to_file.xls")
```
Using None to get all sheets:

```python
# Returns a dictionary of DataFrames
pd.read_excel("path_to_file.xls", sheet_name=None)
```
Using a list to get multiple sheets:

```python
# Returns the 1st and 4th sheet, as a dictionary of DataFrames.
pd.read_excel("path_to_file.xls", sheet_name=["Sheet1", 3])
```
``read_excel`` can read more than one sheet, by setting ``sheet_name`` to either
a list of sheet names, a list of sheet positions, or ``None`` to read all sheets.
Sheets can be specified by sheet index or sheet name, using an integer or string,
respectively.



Reading a ``MultiIndex``
++++++++++++++++++++++++

``read_excel`` can read a ``MultiIndex`` index, by passing a list of columns to ``index_col``
and a ``MultiIndex`` column by passing a list of rows to ``header``.  If either the ``index``
or ``columns`` have serialized level names those will be read in as well by specifying
the rows/columns that make up the levels.

For example, to read in a ``MultiIndex`` index without names:

```python
df = pd.DataFrame(
    {"a": [1, 2, 3, 4], "b": [5, 6, 7, 8]},
    index=pd.MultiIndex.from_product([["a", "b"], ["c", "d"]]),
)
df.to_excel("path_to_file.xlsx")
df = pd.read_excel("path_to_file.xlsx", index_col=[0, 1])
df
```
If the index has level names, they will be parsed as well, using the same
parameters.

```python
df.index = df.index.set_names(["lvl1", "lvl2"])
df.to_excel("path_to_file.xlsx")
df = pd.read_excel("path_to_file.xlsx", index_col=[0, 1])
df
```
If the source file has both ``MultiIndex`` index and columns, lists specifying each
should be passed to ``index_col`` and ``header``:

```python
df.columns = pd.MultiIndex.from_product([["a"], ["b", "d"]], names=["c1", "c2"])
df.to_excel("path_to_file.xlsx")
df = pd.read_excel("path_to_file.xlsx", index_col=[0, 1], header=[0, 1])
df
```
```python
:suppress:

os.remove("path_to_file.xlsx")
```
Missing values in columns specified in ``index_col`` will be forward filled to
allow roundtripping with ``to_excel`` for ``merged_cells=True``. To avoid forward
filling the missing values use ``set_index`` after reading the data instead of
``index_col``.

Parsing specific columns
++++++++++++++++++++++++

It is often the case that users will insert columns to do temporary computations
in Excel and you may not want to read in those columns. ``read_excel`` takes
a ``usecols`` keyword to allow you to specify a subset of columns to parse.

You can specify a comma-delimited set of Excel columns and ranges as a string:

```python
pd.read_excel("path_to_file.xls", "Sheet1", usecols="A,C:E")
```
If ``usecols`` is a list of integers, then it is assumed to be the file column
indices to be parsed.

```python
pd.read_excel("path_to_file.xls", "Sheet1", usecols=[0, 2, 3])
```
Element order is ignored, so ``usecols=[0, 1]`` is the same as ``[1, 0]``.

If ``usecols`` is a list of strings, it is assumed that each string corresponds
to a column name provided either by the user in ``names`` or inferred from the
document header row(s). Those strings define which columns will be parsed:

```python
pd.read_excel("path_to_file.xls", "Sheet1", usecols=["foo", "bar"])
```
Element order is ignored, so ``usecols=['baz', 'joe']`` is the same as ``['joe', 'baz']``.

If ``usecols`` is callable, the callable function will be evaluated against
the column names, returning names where the callable function evaluates to ``True``.

```python
pd.read_excel("path_to_file.xls", "Sheet1", usecols=lambda x: x.isalpha())
```
Parsing dates
+++++++++++++

Datetime-like values are normally automatically converted to the appropriate
dtype when reading the excel file. But if you have a column of strings that
*look* like dates (but are not actually formatted as dates in excel), you can
use the ``parse_dates`` keyword to parse those strings to datetimes:

```python
pd.read_excel("path_to_file.xls", "Sheet1", parse_dates=["date_strings"])
```
Cell converters
+++++++++++++++

It is possible to transform the contents of Excel cells via the ``converters``
option. For instance, to convert a column to boolean:

```python
pd.read_excel("path_to_file.xls", "Sheet1", converters={"MyBools": bool})
```
This options handles missing values and treats exceptions in the converters
as missing data. Transformations are applied cell by cell rather than to the
column as a whole, so the array dtype is not guaranteed. For instance, a
column of integers with missing values cannot be transformed to an array
with integer dtype, because NaN is strictly a float. You can manually mask
missing data to recover integer dtype:

```python
def cfun(x):
    return int(x) if x else -1


pd.read_excel("path_to_file.xls", "Sheet1", converters={"MyInts": cfun})
```
Dtype specifications
++++++++++++++++++++

As an alternative to converters, the type for an entire column can
be specified using the ``dtype`` keyword, which takes a dictionary
mapping column names to types.  To interpret data with
no type inference, use the type ``str`` or ``object``.

```python
pd.read_excel("path_to_file.xls", dtype={"MyInts": "int64", "MyText": str})
```


Writing Excel files
'''''''''''''''''''

Writing Excel files to disk
+++++++++++++++++++++++++++

To write a ``DataFrame`` object to a sheet of an Excel file, you can use the
``to_excel`` instance method.  The arguments are largely the same as ``to_csv``
described above, the first argument being the name of the excel file, and the
optional second argument the name of the sheet to which the ``DataFrame`` should be
written. For example:

```python
df.to_excel("path_to_file.xlsx", sheet_name="Sheet1")
```
Files with a
``.xlsx`` extension will be written using ``xlsxwriter`` (if available) or
``openpyxl``.

The ``DataFrame`` will be written in a way that tries to mimic the REPL output.
The ``index_label`` will be placed in the second
row instead of the first. You can place it in the first row by setting the
``merge_cells`` option in ``to_excel()`` to ``False``:

```python
df.to_excel("path_to_file.xlsx", index_label="label", merge_cells=False)
```
In order to write separate ``DataFrames`` to separate sheets in a single Excel file,
one can pass an `~pandas.io.excel.ExcelWriter`.

```python
with pd.ExcelWriter("path_to_file.xlsx") as writer:
    df1.to_excel(writer, sheet_name="Sheet1")
    df2.to_excel(writer, sheet_name="Sheet2")
```


When using the ``engine_kwargs`` parameter, pandas will pass these arguments to the
engine. For this, it is important to know which function pandas is using internally.

* For the engine openpyxl, pandas is using `openpyxl.Workbook` to create a new sheet and `openpyxl.load_workbook` to append data to an existing sheet. The openpyxl engine writes to (``.xlsx``) and (``.xlsm``) files.

* For the engine xlsxwriter, pandas is using `xlsxwriter.Workbook` to write to (``.xlsx``) files.

* For the engine odf, pandas is using `odf.opendocument.OpenDocumentSpreadsheet` to write to (``.ods``) files.

Writing Excel files to memory
+++++++++++++++++++++++++++++

pandas supports writing Excel files to buffer-like objects such as ``StringIO`` or
``BytesIO`` using `~pandas.io.excel.ExcelWriter`.

```python
from io import BytesIO

bio = BytesIO()

# By setting the 'engine' in the ExcelWriter constructor.
writer = pd.ExcelWriter(bio, engine="xlsxwriter")
df.to_excel(writer, sheet_name="Sheet1")

# Save the workbook
writer.save()

# Seek to the beginning and read to copy the workbook to a variable in memory
bio.seek(0)
workbook = bio.read()
```
> **note.capitalize():**
    ``engine`` is optional but recommended.  Setting the engine determines
    the version of workbook produced. Setting ``engine='xlrd'`` will produce an
    Excel 2003-format workbook (xls).  Using either ``'openpyxl'`` or
    ``'xlsxwriter'`` will produce an Excel 2007-format workbook (xlsx). If
    omitted, an Excel 2007-formatted workbook is produced.




Excel writer engines
''''''''''''''''''''

pandas chooses an Excel writer via two methods:

1. the ``engine`` keyword argument
2. the filename extension (via the default specified in config options)

By default, pandas uses the `XlsxWriter`_  for ``.xlsx``, `openpyxl`_
for ``.xlsm``. If you have multiple
engines installed, you can set the default engine through setting the
config options  ``io.excel.xlsx.writer`` and
``io.excel.xls.writer``. pandas will fall back on `openpyxl`_ for ``.xlsx``
files if `Xlsxwriter`_ is not available.

.. _XlsxWriter: https://xlsxwriter.readthedocs.io
.. _openpyxl: https://openpyxl.readthedocs.io/

To specify which writer you want to use, you can pass an engine keyword
argument to ``to_excel`` and to ``ExcelWriter``. The built-in engines are:

* ``openpyxl``: version 2.4 or higher is required
* ``xlsxwriter``

```python
# By setting the 'engine' in the DataFrame 'to_excel()' methods.
df.to_excel("path_to_file.xlsx", sheet_name="Sheet1", engine="xlsxwriter")

# By setting the 'engine' in the ExcelWriter constructor.
writer = pd.ExcelWriter("path_to_file.xlsx", engine="xlsxwriter")

# Or via pandas configuration.
from pandas import options  # noqa: E402

options.io.excel.xlsx.writer = "xlsxwriter"

df.to_excel("path_to_file.xlsx", sheet_name="Sheet1")
```


Style and formatting
''''''''''''''''''''

The look and feel of Excel worksheets created from pandas can be modified using the following parameters on the ``DataFrame``'s ``to_excel`` method.

* ``float_format`` : Format string for floating point numbers (default ``None``).
* ``freeze_panes`` : A tuple of two integers representing the bottommost row and rightmost column to freeze. Each of these parameters is one-based, so (1, 1) will freeze the first row and first column (default ``None``).

> **note.capitalize():**
    As of pandas 3.0, by default spreadsheets created with the ``to_excel`` method
    will not contain any styling. Users wishing to bold text, add bordered styles,
    etc in a worksheet output by ``to_excel`` can do so by using `Styler.to_excel`
    to create styled excel files. For documentation on styling spreadsheets, see
    `here](https://pandas.pydata.org/docs/user_guide/style.html#Export-to-Excel)_.


[``python
css = "border: 1px solid black; font-weight: bold;"
df.style.map_index(lambda x: css).map_index(lambda x: css, axis=1).to_excel("myfile.xlsx")
```
Using the `Xlsxwriter`_ engine provides many options for controlling the
format of an Excel worksheet created with the ``to_excel`` method.  Excellent examples can be found in the
`Xlsxwriter`_ documentation here: https://xlsxwriter.readthedocs.io/working_with_pandas.html



## OpenDocument Spreadsheets
The io methods for `Excel files`_ also support reading and writing OpenDocument spreadsheets
using the `odfpy](https://pypi.org/project/odfpy/)_ module. The semantics and features for reading and writing
OpenDocument spreadsheets match what can be done for `Excel files`_ using
``engine='odf'``. The optional dependency 'odfpy' needs to be installed.

The `~pandas.read_excel` method can read OpenDocument spreadsheets

```python
# Returns a DataFrame
pd.read_excel("path_to_file.ods", engine="odf")
```
Similarly, the `~pandas.to_excel` method can write OpenDocument spreadsheets

```python
# Writes DataFrame to a .ods file
df.to_excel("path_to_file.ods", engine="odf")
```


## Binary Excel (.xlsb) files
The `~pandas.read_excel` method can also read binary Excel files
using the ``pyxlsb`` module. The semantics and features for reading
binary Excel files mostly match what can be done for `Excel files`_ using
``engine='pyxlsb'``. ``pyxlsb`` does not recognize datetime types
in files and will return floats instead (you can use `calamine<io.calamine>[
if you need recognize datetime types).

```python
# Returns a DataFrame
pd.read_excel("path_to_file.xlsb", engine="pyxlsb")
```
> **note.capitalize():**
   Currently pandas only supports *reading* binary Excel files. Writing
   is not implemented.



## Calamine (Excel and ODS files)
The `~pandas.read_excel` method can read Excel file (``.xlsx``, ``.xlsm``, ``.xls``, ``.xlsb``)
and OpenDocument spreadsheets (``.ods``) using the ``python-calamine`` module.
This module is a binding for Rust library `calamine](https://crates.io/crates/calamine)_
and is faster than other engines in most cases. The optional dependency 'python-calamine' needs to be installed.

```python
# Returns a DataFrame
pd.read_excel("path_to_file.xlsb", engine="calamine")
```


## Clipboard
A handy way to grab data is to use the `~DataFrame.read_clipboard` method,
which takes the contents of the clipboard buffer and passes them to the
``read_csv`` method. For instance, you can copy the following text to the
clipboard (CTRL-C on many operating systems):



     A B C
   x 1 4 p
   y 2 5 q
   z 3 6 r

And then import the data directly to a ``DataFrame`` by calling:

```python
>>> clipdf = pd.read_clipboard()
>>> clipdf
  A B C
x 1 4 p
y 2 5 q
z 3 6 r
```
The ``to_clipboard`` method can be used to write the contents of a ``DataFrame`` to
the clipboard. Following which you can paste the clipboard contents into other
applications (CTRL-V on many operating systems). Here we illustrate writing a
``DataFrame`` into clipboard and reading it back.

```python
>>> df = pd.DataFrame(
...     {"A": [1, 2, 3], "B": [4, 5, 6], "C": ["p", "q", "r"]}, index=["x", "y", "z"]
... )

>>> df
  A B C
x 1 4 p
y 2 5 q
z 3 6 r
>>> df.to_clipboard()
>>> pd.read_clipboard()
  A B C
x 1 4 p
y 2 5 q
z 3 6 r
```
We can see that we got the same content back, which we had earlier written to the clipboard.

> **note.capitalize():**
   You may need to install xclip or xsel (with PyQt5, PyQt4 or qtpy) on Linux to use these methods.



## Pickling
All pandas objects are equipped with ``to_pickle`` methods which use Python's
``cPickle`` module to save data structures to disk using the pickle format.

```python
df
df.to_pickle("foo.pkl")
```
The ``read_pickle`` function in the ``pandas`` namespace can be used to load
any pickled pandas object (or any other pickled object) from file:


```python
pd.read_pickle("foo.pkl")
```
```python
:suppress:

os.remove("foo.pkl")
```
> **warning.capitalize():**
   Loading pickled data received from untrusted sources can be unsafe.

   See: https://docs.python.org/3/library/pickle.html

> **warning.capitalize():**
   `read_pickle` is only guaranteed backwards compatible back to a few minor release.



Compressed pickle files
'''''''''''''''''''''''

`read_pickle`, `DataFrame.to_pickle` and `Series.to_pickle` can read
and write compressed pickle files. The compression types of ``gzip``, ``bz2``, ``xz``, ``zstd`` are supported for reading and writing.
The ``zip`` file format only supports reading and must contain only one data file
to be read.

The compression type can be an explicit parameter or be inferred from the file extension.
If 'infer', then use ``gzip``, ``bz2``, ``zip``, ``xz``, ``zstd`` if filename ends in ``'.gz'``, ``'.bz2'``, ``'.zip'``,
``'.xz'``, or ``'.zst'``, respectively.

The compression parameter can also be a ``dict`` in order to pass options to the
compression protocol. It must have a ``'method'`` key set to the name
of the compression protocol, which must be one of
{``'zip'``, ``'gzip'``, ``'bz2'``, ``'xz'``, ``'zstd'``}. All other key-value pairs are passed to
the underlying compression library.

```python
df = pd.DataFrame(
    {
        "A": np.random.randn(1000),
        "B": "foo",
        "C": pd.date_range("20130101", periods=1000, freq="s"),
    }
)
df
```
Using an explicit compression type:

```python
df.to_pickle("data.pkl.compress", compression="gzip")
rt = pd.read_pickle("data.pkl.compress", compression="gzip")
rt
```
Inferring compression type from the extension:

```python
df.to_pickle("data.pkl.xz", compression="infer")
rt = pd.read_pickle("data.pkl.xz", compression="infer")
rt
```
The default is to 'infer':

```python
df.to_pickle("data.pkl.gz")
rt = pd.read_pickle("data.pkl.gz")
rt

df["A"].to_pickle("s1.pkl.bz2")
rt = pd.read_pickle("s1.pkl.bz2")
rt
```
Passing options to the compression protocol in order to speed up compression:

```python
df.to_pickle("data.pkl.gz", compression={"method": "gzip", "compresslevel": 1})
```
```python
:suppress:

os.remove("data.pkl.compress")
os.remove("data.pkl.xz")
os.remove("data.pkl.gz")
os.remove("s1.pkl.bz2")
```


## msgpack
pandas support for ``msgpack`` has been removed in version 1.0.0. It is
recommended to use `pickle <io.pickle>[ instead.

Alternatively, you can also the Arrow IPC serialization format for on-the-wire
transmission of pandas objects. For documentation on pyarrow, see
`here](https://arrow.apache.org/docs/python/ipc.html)_.




## HDF5 (PyTables)
``HDFStore`` is a dict-like object which reads and writes pandas using
the high performance HDF5 format using the excellent `PyTables
<https://www.pytables.org/>`__ library. See the `cookbook <cookbook.hdf>`
for some advanced strategies

> **warning.capitalize():**
   pandas uses PyTables for reading and writing HDF5 files, which allows
   serializing object-dtype data with pickle. Loading pickled data received from
   untrusted sources can be unsafe.

   See: https://docs.python.org/3/library/pickle.html for more.

```python
:suppress:
:okexcept:

os.remove("store.h5")
```
```python
store = pd.HDFStore("store.h5")
print(store)
```
Objects can be written to the file just like adding key-value pairs to a
dict:

```python
index = pd.date_range("1/1/2000", periods=8)
s = pd.Series(np.random.randn(5), index=["a", "b", "c", "d", "e"])
df = pd.DataFrame(np.random.randn(8, 3), index=index, columns=["A", "B", "C"])

# store.put('s', s) is an equivalent method
store["s"] = s

store["df"] = df

store
```
In a current or later Python session, you can retrieve stored objects:

```python
# store.get('df') is an equivalent method
store["df"]

# dotted (attribute) access provides get as well
store.df
```
Deletion of the object specified by the key:

```python
# store.remove('df') is an equivalent method
del store["df"]

store
```
Closing a Store and using a context manager:

```python
store.close()
store
store.is_open

# Working with, and automatically closing the store using a context manager
with pd.HDFStore("store.h5") as store:
    store.keys()
```
```python
:suppress:

store.close()
os.remove("store.h5")
```
Read/write API
''''''''''''''

``HDFStore`` supports a top-level API using  ``read_hdf`` for reading and ``to_hdf`` for writing,
similar to how ``read_csv`` and ``to_csv`` work.

```python
df_tl = pd.DataFrame({"A": list(range(5)), "B": list(range(5))})
df_tl.to_hdf("store_tl.h5", key="table", append=True)
pd.read_hdf("store_tl.h5", "table", where=["index>2"])
```
```python
:suppress:
:okexcept:

os.remove("store_tl.h5")
```
HDFStore will by default not drop rows that are all missing. This behavior can be changed by setting ``dropna=True``.


```python
df_with_missing = pd.DataFrame(
    {
        "col1": [0, np.nan, 2],
        "col2": [1, np.nan, np.nan],
    }
)
df_with_missing

df_with_missing.to_hdf("file.h5", key="df_with_missing", format="table", mode="w")

pd.read_hdf("file.h5", "df_with_missing")

df_with_missing.to_hdf(
    "file.h5", key="df_with_missing", format="table", mode="w", dropna=True
)
pd.read_hdf("file.h5", "df_with_missing")
```
```python
:suppress:

os.remove("file.h5")
```


Fixed format
''''''''''''

The examples above show storing using ``put``, which write the HDF5 to ``PyTables`` in a fixed array format, called
the ``fixed`` format. These types of stores are **not** appendable once written (though you can simply
remove them and rewrite). Nor are they **queryable**; they must be
retrieved in their entirety. They also do not support dataframes with non-unique column names.
The ``fixed`` format stores offer very fast writing and slightly faster reading than ``table`` stores.
This format is specified by default when using ``put`` or ``to_hdf`` or by ``format='fixed'`` or ``format='f'``.

> **warning.capitalize():**
   A ``fixed`` format will raise a ``TypeError`` if you try to retrieve using a ``where``:

   ```python
:okexcept:

   pd.DataFrame(np.random.randn(10, 2)).to_hdf("test_fixed.h5", key="df")
   pd.read_hdf("test_fixed.h5", "df", where="index>5")


   :suppress:

   os.remove("test_fixed.h5")
```


Table format
''''''''''''

``HDFStore`` supports another ``PyTables`` format on disk, the ``table``
format. Conceptually a ``table`` is shaped very much like a DataFrame,
with rows and columns. A ``table`` may be appended to in the same or
other sessions.  In addition, delete and query type operations are
supported. This format is specified by ``format='table'`` or ``format='t'``
to ``append`` or ``put`` or ``to_hdf``.

This format can be set as an option as well ``pd.set_option('io.hdf.default_format','table')`` to
enable ``put/append/to_hdf`` to by default store in the ``table`` format.

```python
:suppress:
:okexcept:

os.remove("store.h5")
```
```python
store = pd.HDFStore("store.h5")
df1 = df[0:4]
df2 = df[4:]

# append data (creates a table automatically)
store.append("df", df1)
store.append("df", df2)
store

# select the entire object
store.select("df")

# the type of stored data
store.root.df._v_attrs.pandas_type
```
> **note.capitalize():**
   You can also create a ``table`` by passing ``format='table'`` or ``format='t'`` to a ``put`` operation.



Hierarchical keys
'''''''''''''''''

Keys to a store can be specified as a string. These can be in a
hierarchical path-name like format (e.g. ``foo/bar/bah``), which will
generate a hierarchy of sub-stores (or ``Groups`` in PyTables
parlance). Keys can be specified without the leading '/' and are **always**
absolute (e.g. 'foo' refers to '/foo'). Removal operations can remove
everything in the sub-store and **below**, so be *careful*.

```python
store.put("foo/bar/bah", df)
store.append("food/orange", df)
store.append("food/apple", df)
store

# a list of keys are returned
store.keys()

# remove all nodes under this level
store.remove("food")
store
```
You can walk through the group hierarchy using the ``walk`` method which
will yield a tuple for each group key along with the relative keys of its contents.

```python
for (path, subgroups, subkeys) in store.walk():
    for subgroup in subgroups:
        print("GROUP: {}/{}".format(path, subgroup))
    for subkey in subkeys:
        key = "/".join([path, subkey])
        print("KEY: {}".format(key))
        print(store.get(key))
```
> **warning.capitalize():**
    Hierarchical keys cannot be retrieved as dotted (attribute) access as described above for items stored under the root node.

    ```python
:okexcept:

   store.foo.bar.bah



   # you can directly access the actual PyTables node but using the root node
   store.root.foo.bar.bah

Instead, use explicit string based keys:



   store["foo/bar/bah"]
```


Storing types
'''''''''''''

Storing mixed types in a table
++++++++++++++++++++++++++++++

Storing mixed-dtype data is supported. Strings are stored as a
fixed-width using the maximum size of the appended column. Subsequent attempts
at appending longer strings will raise a ``ValueError``.

Passing ``min_itemsize={`values`: size}`` as a parameter to append
will set a larger minimum for the string columns. Storing ``floats,
strings, ints, bools, datetime64`` are currently supported. For string
columns, passing ``nan_rep = 'nan'`` to append will change the default
nan representation on disk (which converts to/from ``np.nan``), this
defaults to ``nan``.

```python
df_mixed = pd.DataFrame(
    {
        "A": np.random.randn(8),
        "B": np.random.randn(8),
        "C": np.array(np.random.randn(8), dtype="float32"),
        "string": "string",
        "int": 1,
        "bool": True,
        "datetime64": pd.Timestamp("20010102"),
    },
    index=list(range(8)),
)
df_mixed.loc[df_mixed.index[3:5], ["A", "B", "string", "datetime64"]] = np.nan

store.append("df_mixed", df_mixed, min_itemsize={"values": 50})
df_mixed1 = store.select("df_mixed")
df_mixed1
df_mixed1.dtypes.value_counts()

# we have provided a minimum string column size
store.root.df_mixed.table
```
Storing MultiIndex DataFrames
+++++++++++++++++++++++++++++

Storing MultiIndex ``DataFrames`` as tables is very similar to
storing/selecting from homogeneous index ``DataFrames``.

```python
index = pd.MultiIndex(
   levels=[["foo", "bar", "baz", "qux"], ["one", "two", "three"]],
   codes=[[0, 0, 0, 1, 1, 2, 2, 3, 3, 3], [0, 1, 2, 0, 1, 1, 2, 0, 1, 2]],
   names=["foo", "bar"],
)
df_mi = pd.DataFrame(np.random.randn(10, 3), index=index, columns=["A", "B", "C"])
df_mi

store.append("df_mi", df_mi)
store.select("df_mi")

# the levels are automatically included as data columns
store.select("df_mi", "foo=bar")
```
> **note.capitalize():**
   The ``index`` keyword is reserved and cannot be use as a level name.



Querying
''''''''

Querying a table
++++++++++++++++

``select`` and ``delete`` operations have an optional criterion that can
be specified to select/delete only a subset of the data. This allows one
to have a very large on-disk table and retrieve only a portion of the
data.

A query is specified using the ``Term`` class under the hood, as a boolean expression.

* ``index`` and ``columns`` are supported indexers of ``DataFrames``.
* if ``data_columns`` are specified, these can be used as additional indexers.
* level name in a MultiIndex, with default name  ``level_0``, ``level_1``, … if not provided.

Valid comparison operators are:

``=, ==, !=, >, >=, <, <=``

Valid boolean expressions are combined with:

* ``|`` : or
* ``&`` : and
* ``(`` and ``)`` : for grouping

These rules are similar to how boolean expressions are used in pandas for indexing.

> **note.capitalize():**
   - ``=`` will be automatically expanded to the comparison operator ``==``
   - ``~`` is the not operator, but can only be used in very limited
     circumstances
   - If a list/tuple of expressions is passed they will be combined via ``&``

The following are valid expressions:

* ``'index >= date'``
* ``"columns = ['A', 'D']"``
* ``"columns in ['A', 'D']"``
* ``'columns = A'``
* ``'columns == A'``
* ``"~(columns = ['A', 'B'])"``
* ``'index > df.index[3] & string = "bar"'``
* ``'(index > df.index[3] & index <= df.index[6]) | string = "bar"'``
* ``"ts >= Timestamp('2012-02-01')"``
* ``"major_axis>=20130101"``

The ``indexers`` are on the left-hand side of the sub-expression:

``columns``, ``major_axis``, ``ts``

The right-hand side of the sub-expression (after a comparison operator) can be:

* functions that will be evaluated, e.g. ``Timestamp('2012-02-01')``
* strings, e.g. ``"bar"``
* date-like, e.g. ``20130101``, or ``"20130101"``
* lists, e.g. ``"['A', 'B']"``
* variables that are defined in the local names space, e.g. ``date``

> **note.capitalize():**
   Passing a string to a query by interpolating it into the query
   expression is not recommended. Simply assign the string of interest to a
   variable and use that variable in an expression. For example, do this

   ```python
string = "HolyMoly'"
   store.select("df", "index == string")

instead of this



   string = "HolyMoly'"
   store.select('df', f'index == {string}')

The latter will **not** work and will raise a ``SyntaxError``.Note that
there's a single quote followed by a double quote in the ``string``
variable.

If you *must* interpolate, use the ``'%r'`` format specifier



   store.select("df", "index == %r" % string)

which will quote ``string``.
```
Here are some examples:

```python
dfq = pd.DataFrame(
    np.random.randn(10, 4),
    columns=list("ABCD"),
    index=pd.date_range("20130101", periods=10),
)
store.append("dfq", dfq, format="table", data_columns=True)
```
Use boolean expressions, with in-line function evaluation.

```python
store.select("dfq", "index>pd.Timestamp('20130104') & columns=['A', 'B']")
```
Use inline column reference.

```python
store.select("dfq", where="A>0 or C>0")
```
The ``columns`` keyword can be supplied to select a list of columns to be
returned, this is equivalent to passing a
``'columns=list_of_columns_to_filter'``:

```python
store.select("df", "columns=['A', 'B']")
```
``start`` and ``stop`` parameters can be specified to limit the total search
space. These are in terms of the total number of rows in a table.

> **note.capitalize():**
   ``select`` will raise a ``ValueError`` if the query expression has an unknown
   variable reference. Usually this means that you are trying to select on a column
   that is **not** a data_column.

   ``select`` will raise a ``SyntaxError`` if the query expression is not valid.




Query timedelta64[ns]
+++++++++++++++++++++

You can store and query using the ``timedelta64[ns]`` type. Terms can be
specified in the format: ``<float>(<unit>)``, where float may be signed (and fractional), and unit can be
``D,s,ms,us,ns`` for the timedelta. Here's an example:

```python
from datetime import timedelta

dftd = pd.DataFrame(
    {
        "A": pd.Timestamp("20130101"),
        "B": [
            pd.Timestamp("20130101") + timedelta(days=i, seconds=10)
            for i in range(10)
        ],
    }
)
dftd["C"] = dftd["A"] - dftd["B"]
dftd
store.append("dftd", dftd, data_columns=True)
store.select("dftd", "C<'-3.5D'")
[``


Query MultiIndex
++++++++++++++++

Selecting from a ``MultiIndex`` can be achieved by using the name of the level.

```python
df_mi.index.names
store.select("df_mi", "foo=baz and bar=two")
```
If the ``MultiIndex`` levels names are ``None``, the levels are automatically made available via
the ``level_n`` keyword with ``n`` the level of the ``MultiIndex`` you want to select from.

```python
index = pd.MultiIndex(
    levels=[["foo", "bar", "baz", "qux"], ["one", "two", "three"]],
    codes=[[0, 0, 0, 1, 1, 2, 2, 3, 3, 3], [0, 1, 2, 0, 1, 1, 2, 0, 1, 2]],
)
df_mi_2 = pd.DataFrame(np.random.randn(10, 3), index=index, columns=["A", "B", "C"])
df_mi_2

store.append("df_mi_2", df_mi_2)

# the levels are automatically included as data columns with keyword level_n
store.select("df_mi_2", "level_0=foo and level_1=two")
```
Indexing
++++++++

You can create/modify an index for a table with ``create_table_index``
after data is already in the table (after and ``append/put``
operation). Creating a table index is **highly** encouraged. This will
speed your queries a great deal when you use a ``select`` with the
indexed dimension as the ``where``.

> **note.capitalize():**
   Indexes are automagically created on the indexables
   and any data columns you specify. This behavior can be turned off by passing
   ``index=False`` to ``append``.

```python
# we have automagically already created an index (in the first section)
i = store.root.df.table.cols.index.index
i.optlevel, i.kind

# change an index by passing new parameters
store.create_table_index("df", optlevel=9, kind="full")
i = store.root.df.table.cols.index.index
i.optlevel, i.kind
```
Oftentimes when appending large amounts of data to a store, it is useful to turn off index creation for each append, then recreate at the end.

```python
df_1 = pd.DataFrame(np.random.randn(10, 2), columns=list("AB"))
df_2 = pd.DataFrame(np.random.randn(10, 2), columns=list("AB"))

st = pd.HDFStore("appends.h5", mode="w")
st.append("df", df_1, data_columns=["B"], index=False)
st.append("df", df_2, data_columns=["B"], index=False)
st.get_storer("df").table
```
Then create the index when finished appending.

```python
st.create_table_index("df", columns=["B"], optlevel=9, kind="full")
st.get_storer("df").table

st.close()
```
```python
:suppress:
:okexcept:

os.remove("appends.h5")
```
See `here](https://stackoverflow.com/questions/17893370/ptrepack-sortby-needs-full-index)_ for how to create a completely-sorted-index (CSI) on an existing store.



Query via data columns
++++++++++++++++++++++

You can designate (and index) certain columns that you want to be able
to perform queries (other than the ``indexable`` columns, which you can
always query). For instance say you want to perform this common
operation, on-disk, and return just the frame that matches this
query. You can specify ``data_columns = True`` to force all columns to
be ``data_columns``.

```python
df_dc = df.copy()
df_dc["string"] = "foo"
df_dc.loc[df_dc.index[4:6], "string"] = np.nan
df_dc.loc[df_dc.index[7:9], "string"] = "bar"
df_dc["string2"] = "cool"
df_dc.loc[df_dc.index[1:3], ["B", "C"]] = 1.0
df_dc

# on-disk operations
store.append("df_dc", df_dc, data_columns=["B", "C", "string", "string2"])
store.select("df_dc", where="B > 0")

# getting creative
store.select("df_dc", "B > 0 & C > 0 & string == foo")

# this is in-memory version of this type of selection
df_dc[(df_dc.B > 0) & (df_dc.C > 0) & (df_dc.string == "foo")]

# we have automagically created this index and the B/C/string/string2
# columns are stored separately as ``PyTables`` columns
store.root.df_dc.table
```
There is some performance degradation by making lots of columns into
``data columns``, so it is up to the user to designate these. In addition,
you cannot change data columns (nor indexables) after the first
append/put operation (Of course you can simply read in the data and
create a new table!).

Iterator
++++++++

You can pass ``iterator=True`` or ``chunksize=number_in_a_chunk``
to ``select`` and ``select_as_multiple`` to return an iterator on the results.
The default is 50,000 rows returned in a chunk.

```python
for df in store.select("df", chunksize=3):
    print(df)
```
> **note.capitalize():**
   You can also use the iterator with ``read_hdf`` which will open, then
   automatically close the store when finished iterating.

   ```python
for df in pd.read_hdf("store.h5", "df", chunksize=3):
    print(df)
```
Note, that the chunksize keyword applies to the **source** rows. So if you
are doing a query, then the chunksize will subdivide the total rows in the table
and the query applied, returning an iterator on potentially unequal sized chunks.

Here is a recipe for generating a query and using it to create equal sized return
chunks.

```python
dfeq = pd.DataFrame({"number": np.arange(1, 11)})
dfeq

store.append("dfeq", dfeq, data_columns=["number"])

def chunks(l, n):
    return [l[i: i + n] for i in range(0, len(l), n)]

evens = [2, 4, 6, 8, 10]
coordinates = store.select_as_coordinates("dfeq", "number=evens")
for c in chunks(coordinates, 2):
    print(store.select("dfeq", where=c))
```
Advanced queries
++++++++++++++++

#### Select a single column
To retrieve a single indexable or data column, use the
method ``select_column``. This will, for example, enable you to get the index
very quickly. These return a ``Series`` of the result, indexed by the row number.
These do not currently accept the ``where`` selector.

```python
store.select_column("df_dc", "index")
store.select_column("df_dc", "string")
```


#### Selecting coordinates
Sometimes you want to get the coordinates (a.k.a the index locations) of your query. This returns an
``Index`` of the resulting locations. These coordinates can also be passed to subsequent
``where`` operations.

```python
df_coord = pd.DataFrame(
    np.random.randn(1000, 2), index=pd.date_range("20000101", periods=1000)
)
store.append("df_coord", df_coord)
c = store.select_as_coordinates("df_coord", "index > 20020101")
c
store.select("df_coord", where=c)
```


#### Selecting using a where mask
Sometime your query can involve creating a list of rows to select. Usually this ``mask`` would
be a resulting ``index`` from an indexing operation. This example selects the months of
a datetimeindex which are 5.

```python
df_mask = pd.DataFrame(
    np.random.randn(1000, 2), index=pd.date_range("20000101", periods=1000)
)
store.append("df_mask", df_mask)
c = store.select_column("df_mask", "index")
where = c[pd.DatetimeIndex(c).month == 5].index
store.select("df_mask", where=where)
```
#### Storer object
If you want to inspect the stored object, retrieve via
``get_storer``. You could use this programmatically to say get the number
of rows in an object.

```python
store.get_storer("df_dc").nrows
```
Multiple table queries
++++++++++++++++++++++

The methods ``append_to_multiple`` and
``select_as_multiple`` can perform appending/selecting from
multiple tables at once. The idea is to have one table (call it the
selector table) that you index most/all of the columns, and perform your
queries. The other table(s) are data tables with an index matching the
selector table's index. You can then perform a very fast query
on the selector table, yet get lots of data back. This method is similar to
having a very wide table, but enables more efficient queries.

The ``append_to_multiple`` method splits a given single DataFrame
into multiple tables according to ``d``, a dictionary that maps the
table names to a list of 'columns' you want in that table. If ``None``
is used in place of a list, that table will have the remaining
unspecified columns of the given DataFrame. The argument ``selector``
defines which table is the selector table (which you can make queries from).
The argument ``dropna`` will drop rows from the input ``DataFrame`` to ensure
tables are synchronized.  This means that if a row for one of the tables
being written to is entirely ``np.nan``, that row will be dropped from all tables.

If ``dropna`` is False, **THE USER IS RESPONSIBLE FOR SYNCHRONIZING THE TABLES**.
Remember that entirely ``np.Nan`` rows are not written to the HDFStore, so if
you choose to call ``dropna=False``, some tables may have more rows than others,
and therefore ``select_as_multiple`` may not work or it may return unexpected
results.

```python
df_mt = pd.DataFrame(
    np.random.randn(8, 6),
    index=pd.date_range("1/1/2000", periods=8),
    columns=["A", "B", "C", "D", "E", "F"],
)
df_mt["foo"] = "bar"
df_mt.loc[df_mt.index[1], ("A", "B")] = np.nan

# you can also create the tables individually
store.append_to_multiple(
    {"df1_mt": ["A", "B"], "df2_mt": None}, df_mt, selector="df1_mt"
)
store

# individual tables were created
store.select("df1_mt")
store.select("df2_mt")

# as a multiple
store.select_as_multiple(
    ["df1_mt", "df2_mt"],
    where=["A>0", "B>0"],
    selector="df1_mt",
)
```
Delete from a table
'''''''''''''''''''

You can delete from a table selectively by specifying a ``where``. In
deleting rows, it is important to understand the ``PyTables`` deletes
rows by erasing the rows, then **moving** the following data. Thus
deleting can potentially be a very expensive operation depending on the
orientation of your data. To get optimal performance, it's
worthwhile to have the dimension you are deleting be the first of the
``indexables``.

Data is ordered (on the disk) in terms of the ``indexables``. Here's a
simple use case. You store panel-type data, with dates in the
``major_axis`` and ids in the ``minor_axis``. The data is then
interleaved like this:

* date_1
    * id_1
    * id_2
    *  .
    * id_n
* date_2
    * id_1
    *  .
    * id_n

It should be clear that a delete operation on the ``major_axis`` will be
fairly quick, as one chunk is removed, then the following data moved. On
the other hand a delete operation on the ``minor_axis`` will be very
expensive. In this case it would almost certainly be faster to rewrite
the table using a ``where`` that selects all but the missing data.

> **warning.capitalize():**
   Please note that HDF5 **DOES NOT RECLAIM SPACE** in the h5 files
   automatically. Thus, repeatedly deleting (or removing nodes) and adding
   again, **WILL TEND TO INCREASE THE FILE SIZE**.

   To *repack and clean* the file, use `ptrepack <io.hdf5-ptrepack>`.



Notes & caveats
'''''''''''''''


Compression
+++++++++++

``PyTables`` allows the stored data to be compressed. This applies to
all kinds of stores, not just tables. Two parameters are used to
control compression: ``complevel`` and ``complib``.

* ``complevel`` specifies if and how hard data is to be compressed.
  ``complevel=0`` and ``complevel=None`` disables compression and
  ``0<complevel<10[` enables compression.

* ``complib`` specifies which compression library to use.
  If nothing is  specified the default library ``zlib`` is used. A
  compression library usually optimizes for either good compression rates
  or speed and the results will depend on the type of data. Which type of
  compression to choose depends on your specific needs and data. The list
  of supported compression libraries:

  - `zlib](https://zlib.net/): The default compression library.
    A classic in terms of compression, achieves good compression
    rates but is somewhat slow.
  - [lzo](https://www.oberhumer.com/opensource/lzo/): Fast
    compression and decompression.
  - [bzip2](https://sourceware.org/bzip2/): Good compression rates.
  - [blosc](https://www.blosc.org/): Fast compression and
    decompression.

    Support for alternative blosc compressors:

    - [blosc:blosclz](https://www.blosc.org/) This is the
      default compressor for [`blosc``
    - `blosc:lz4
     ](https://fastcompression.blogspot.com/p/lz4.html):
      A compact, very popular and fast compressor.
    - [blosc:lz4hc
     ](https://fastcompression.blogspot.com/p/lz4.html):
      A tweaked version of LZ4, produces better
      compression ratios at the expense of speed.
    - [blosc:snappy](https://google.github.io/snappy/):
      A popular compressor used in many places.
    - [blosc:zlib](https://zlib.net/): A classic;
      somewhat slower than the previous ones, but
      achieving better compression ratios.
    - [blosc:zstd](https://facebook.github.io/zstd/): An
      extremely well balanced codec; it provides the best
      compression ratios among the others above, and at
      reasonably fast speed.

  If ``complib`` is defined as something other than the listed libraries a
  ``ValueError`` exception is issued.

> **note.capitalize():**
   If the library specified with the ``complib`` option is missing on your platform,
   compression defaults to ``zlib`` without further ado.

Enable compression for all objects within the file:

```python
store_compressed = pd.HDFStore(
    "store_compressed.h5", complevel=9, complib="blosc:blosclz"
)
```
Or on-the-fly compression (this only applies to tables) in stores where compression is not enabled:

```python
store.append("df", df, complib="zlib", complevel=5)
```


ptrepack
++++++++

``PyTables`` offers better write performance when tables are compressed after
they are written, as opposed to turning on compression at the very
beginning. You can use the supplied ``PyTables`` utility
``ptrepack``. In addition, ``ptrepack`` can change compression levels
after the fact.



   ptrepack --chunkshape=auto --propindexes --complevel=9 --complib=blosc in.h5 out.h5

Furthermore ``ptrepack in.h5 out.h5`` will *repack* the file to allow
you to reuse previously deleted space. Alternatively, one can simply
remove the file and write again, or use the ``copy`` method.



Caveats
+++++++

> **warning.capitalize():**
   ``HDFStore`` is **not-threadsafe for writing**. The underlying
   ``PyTables`` only supports concurrent reads (via threading or
   processes). If you need reading and writing *at the same time*, you
   need to serialize these operations in a single thread in a single
   process. You will corrupt your data otherwise. See the (`2397`) for more information.

* If you use locks to manage write access between multiple processes, you
  may want to use :py`~os.fsync` before releasing write locks. For
  convenience you can use ``store.flush(fsync=True)`` to do this for you.
* Once a ``table`` is created columns (DataFrame)
  are fixed; only exactly the same columns can be appended
* Be aware that timezones (e.g., ``zoneinfo.ZoneInfo('US/Eastern')``)
  are not necessarily equal across timezone versions.  So if data is
  localized to a specific timezone in the HDFStore using one version
  of a timezone library and that data is updated with another version, the data
  will be converted to UTC since these timezones are not considered
  equal.  Either use the same version of timezone library or use ``tz_convert`` with
  the updated timezone definition.

> **warning.capitalize():**
   ``PyTables`` will show a ``NaturalNameWarning`` if a column name
   cannot be used as an attribute selector.
   *Natural* identifiers contain only letters, numbers, and underscores,
   and may not begin with a number.
   Other identifiers cannot be used in a ``where`` clause
   and are generally a bad idea.



DataTypes
'''''''''

``HDFStore`` will map an object dtype to the ``PyTables`` underlying
dtype. This means the following types are known to work:

======================================================  =========================
Type                                                    Represents missing values
======================================================  =========================
floating : ``float64, float32, float16``                ``np.nan``
integer : ``int64, int32, int8, uint64,uint32, uint8``
boolean
``datetime64[ns]``                                      ``NaT``
``timedelta64[ns]``                                     ``NaT``
categorical : see the section below
object : ``strings``                                    ``np.nan``
======================================================  =========================

``unicode`` columns are not supported, and **WILL FAIL**.



Categorical data
++++++++++++++++

You can write data that contains ``category`` dtypes to a ``HDFStore``.
Queries work the same as if it was an object array. However, the ``category`` dtyped data is
stored in a more efficient manner.

```python
dfcat = pd.DataFrame(
    {"A": pd.Series(list("aabbcdba")).astype("category"), "B": np.random.randn(8)}
)
dfcat
dfcat.dtypes
cstore = pd.HDFStore("cats.h5", mode="w")
cstore.append("dfcat", dfcat, format="table", data_columns=["A"])
result = cstore.select("dfcat", where="A in ['b', 'c']")
result
result.dtypes
```
```python
:suppress:
:okexcept:

cstore.close()
os.remove("cats.h5")
```
String columns
++++++++++++++

**min_itemsize**

The underlying implementation of ``HDFStore`` uses a fixed column width (itemsize) for string columns.
A string column itemsize is calculated as the maximum of the
length of data (for that column) that is passed to the ``HDFStore``, **in the first append**. Subsequent appends,
may introduce a string for a column **larger** than the column can hold, an Exception will be raised (otherwise you
could have a silent truncation of these columns, leading to loss of information). In the future we may relax this and
allow a user-specified truncation to occur.

Pass ``min_itemsize`` on the first table creation to a-priori specify the minimum length of a particular string column.
``min_itemsize`` can be an integer, or a dict mapping a column name to an integer. You can pass ``values`` as a key to
allow all *indexables* or *data_columns* to have this min_itemsize.

Passing a ``min_itemsize`` dict will cause all passed columns to be created as *data_columns* automatically.

> **note.capitalize():**
   If you are not passing any ``data_columns``, then the ``min_itemsize`` will be the maximum of the length of any string passed

```python
dfs = pd.DataFrame({"A": "foo", "B": "bar"}, index=list(range(5)))
dfs

# A and B have a size of 30
store.append("dfs", dfs, min_itemsize=30)
store.get_storer("dfs").table

# A is created as a data_column with a size of 30
# B is size is calculated
store.append("dfs2", dfs, min_itemsize={"A": 30})
store.get_storer("dfs2").table
```
**nan_rep**

String columns will serialize a ``np.nan`` (a missing value) with the ``nan_rep`` string representation. This defaults to the string value ``nan``.
You could inadvertently turn an actual ``nan`` value into a missing value.

```python
dfss = pd.DataFrame({"A": ["foo", "bar", "nan"]})
dfss

store.append("dfss", dfss)
store.select("dfss")

# here you need to specify a different nan rep
store.append("dfss2", dfss, nan_rep="_nan_")
store.select("dfss2")
```
Performance
'''''''''''

* ``tables`` format come with a writing performance penalty as compared to
  ``fixed`` stores. The benefit is the ability to append/delete and
  query (potentially very large amounts of data).  Write times are
  generally longer as compared with regular stores. Query times can
  be quite fast, especially on an indexed axis.
* You can pass ``chunksize=<int>`` to ``append``, specifying the
  write chunksize (default is 50000). This will significantly lower
  your memory usage on writing.
* You can pass ``expectedrows=<int>[` to the first ``append``,
  to set the TOTAL number of rows that ``PyTables`` will expect.
  This will optimize read/write performance.
* Duplicate rows can be written to tables, but are filtered out in
  selection (with the last items being selected; thus a table is
  unique on major, minor pairs)
* A ``PerformanceWarning`` will be raised if you are attempting to
  store types that will be pickled by PyTables (rather than stored as
  endemic types). See
  `Here](https://stackoverflow.com/questions/14355151/how-to-make-pandas-hdfstore-put-operation-faster/14370190#14370190)_
  for more information and some solutions.


[``python
:suppress:

store.close()
os.remove("store.h5")
```


## Feather
Feather provides binary columnar serialization for data frames. It is designed to make reading and writing data
frames efficient, and to make sharing data across data analysis languages easy.

Feather is designed to faithfully serialize and de-serialize DataFrames, supporting all of the pandas
dtypes, including extension dtypes such as categorical and datetime with tz.

Several caveats:

* The format will NOT write an ``Index``, or ``MultiIndex`` for the
  ``DataFrame`` and will raise an error if a non-default one is provided. You
  can ``.reset_index()`` to store the index or ``.reset_index(drop=True)`` to
  ignore it.
* Duplicate column names and non-string columns names are not supported
* Actual Python objects in object dtype columns are not supported. These will
  raise a helpful error message on an attempt at serialization.

See the `Full Documentation](https://github.com/wesm/feather)_.

[``python
import pytz

df = pd.DataFrame(
    {
        "a": list("abc"),
        "b": list(range(1, 4)),
        "c": np.arange(3, 6).astype("u1"),
        "d": np.arange(4.0, 7.0, dtype="float64"),
        "e": [True, False, True],
        "f": pd.Categorical(list("abc")),
        "g": pd.date_range("20130101", periods=3),
        "h": pd.date_range("20130101", periods=3, tz=pytz.timezone("US/Eastern")),
        "i": pd.date_range("20130101", periods=3, freq="ns"),
    }
)

df
df.dtypes
```
Write to a feather file.

```python
:okwarning:

df.to_feather("example.feather")
```
Read from a feather file.

```python
:okwarning:

result = pd.read_feather("example.feather")
result

# we preserve dtypes
result.dtypes
```
```python
:suppress:

os.remove("example.feather")
```


## Parquet
`Apache Parquet](https://parquet.apache.org/)_ provides a partitioned binary columnar serialization for data frames. It is designed to
make reading and writing data frames efficient, and to make sharing data across data analysis
languages easy. Parquet can use a variety of compression techniques to shrink the file size as much as possible
while still maintaining good read performance.

Parquet is designed to faithfully serialize and de-serialize ``DataFrame`` s, supporting all of the pandas
dtypes, including extension dtypes such as datetime with timezone.

Several caveats.

* Duplicate column names and non-string columns names are not supported.
* The DataFrame index is written as separate column(s) when it is a non-default range index.
  This extra column can cause problems for non-pandas consumers that are not expecting it. You can
  force including or omitting indexes with the ``index`` argument.
* Index level names, if specified, must be strings.
* In the ``pyarrow`` engine, categorical dtypes for non-string types can be serialized to parquet, but will de-serialize as their primitive dtype.
* The ``pyarrow`` engine supports the ``Period`` and ``Interval`` dtypes. ``fastparquet`` does not support those.
* Non supported types include actual Python object types. These will raise a helpful error message
  on an attempt at serialization.
* The ``pyarrow`` engine preserves extension data types such as the nullable integer and string data
  type (this can also work for external extension types, requiring the extension type to implement the needed protocols,
  see the `extension types documentation <extending.extension.arrow>[).

You can specify an ``engine`` to direct the serialization. This can be one of ``pyarrow``, or ``fastparquet``, or ``auto``.
If the engine is NOT specified, then the ``pd.options.io.parquet.engine`` option is checked; if this is also ``auto``,
then ``pyarrow`` is used when installed, and falling back to ``fastparquet``.

See the documentation for `pyarrow](https://arrow.apache.org/docs/python/)_ and [fastparquet](https://fastparquet.readthedocs.io/en/latest/)_.

> **note.capitalize():**
   These engines are very similar and should read/write nearly identical parquet format files for most cases.
   These libraries differ by having different underlying dependencies ([`fastparquet`` by using ``numba``, while ``pyarrow`` uses a c-library).

```python
df = pd.DataFrame(
    {
        "a": list("abc"),
        "b": list(range(1, 4)),
        "c": np.arange(3, 6).astype("u1"),
        "d": np.arange(4.0, 7.0, dtype="float64"),
        "e": [True, False, True],
        "f": pd.date_range("20130101", periods=3),
        "g": pd.date_range("20130101", periods=3, tz="US/Eastern"),
        "h": pd.Categorical(list("abc")),
        "i": pd.Categorical(list("abc"), ordered=True),
    }
)

df
df.dtypes
```
Write to a parquet file.

```python
# specify engine="pyarrow" or engine="fastparquet" to use a specific engine
df.to_parquet("example.parquet")
```
Read from a parquet file.

```python
result = pd.read_parquet("example.parquet")
result.dtypes
```
By setting the ``dtype_backend`` argument you can control the default dtypes used for the resulting DataFrame.

```python
result = pd.read_parquet("example.parquet", dtype_backend="pyarrow")
result.dtypes
```
> **note.capitalize():**
   Note that this is not supported for ``fastparquet``.


Read only certain columns of a parquet file.

```python
result = pd.read_parquet("example.parquet", columns=["a", "b"])
result.dtypes
```
```python
:suppress:

os.remove("example.parquet")
```
Handling indexes
''''''''''''''''

Serializing a ``DataFrame`` to parquet may include the implicit index as one or
more columns in the output file. For example, this code:

```python
df = pd.DataFrame({"a": [1, 2], "b": [3, 4]}, index=[1, 2])
df.to_parquet("test.parquet", engine="pyarrow")
```
creates a parquet file with *three* columns (``a``, ``b``, and
``__index_level_0__`` when using the ``pyarrow`` engine, or ``index``, ``a``,
and ``b`` when using the ``fastparquet`` engine) because the index in this case
is not a default range index. In general, the index *may or may not* be written
to the file (see the
`preserve_index keyword for pyarrow](https://arrow.apache.org/docs/python/pandas.html#handling-pandas-indexes)_
or the
[write_index keyword for fastparquet](https://fastparquet.readthedocs.io/en/latest/api.html#fastparquet.write)_
to check the default behaviour).

This unexpected extra column causes some databases like Amazon Redshift to reject
the file, because that column doesn't exist in the target table.

If you want to omit a dataframe's indexes when writing, pass [`index=False`` to
`~pandas.DataFrame.to_parquet`:

```python
df.to_parquet("test.parquet", index=False)
```
This creates a parquet file with just the two expected columns, ``a`` and ``b``.
If your ``DataFrame`` has a custom index, you won't get it back when you load
this file into a ``DataFrame``.

Passing ``index=True`` will *always* write the index, even if that's not the
underlying engine's default behavior.

```python
:suppress:

os.remove("test.parquet")
```
Partitioning Parquet files
''''''''''''''''''''''''''

Parquet supports partitioning of data based on the values of one or more columns.

```python
df = pd.DataFrame({"a": [0, 0, 1, 1], "b": [0, 1, 0, 1]})
df.to_parquet(path="test", engine="pyarrow", partition_cols=["a"], compression=None)
```
The ``path`` specifies the parent directory to which data will be saved.
The ``partition_cols`` are the column names by which the dataset will be partitioned.
Columns are partitioned in the order they are given. The partition splits are
determined by the unique values in the partition columns.
The above example creates a partitioned dataset that may look like:



    test
    ├── a=0
    │   ├── 0bac803e32dc42ae83fddfd029cbdebc.parquet
    │   └──  ...
    └── a=1
        ├── e6ab24a4f45147b49b54a662f0c412a3.parquet
        └── ...

```python
:suppress:

from shutil import rmtree

try:
    rmtree("test")
except OSError:
    pass
```


## Iceberg


Apache Iceberg is a high performance open-source format for large analytic tables.
Iceberg enables the use of SQL tables for big data while making it possible for different
engines to safely work with the same tables at the same time.

Iceberg support predicate pushdown and column pruning, which are available to pandas
users via the ``row_filter`` and ``selected_fields`` parameters of the `~pandas.read_iceberg`
function. This is convenient to extract from large tables a subset that fits in memory as a
pandas ``DataFrame``.

Internally, pandas uses PyIceberg_ to query Iceberg.

.. _PyIceberg: https://py.iceberg.apache.org/

A simple example loading all data from an Iceberg table ``my_table`` defined in the
``my_catalog`` catalog.

```python
df = pd.read_iceberg("my_table", catalog_name="my_catalog")
```
Catalogs must be defined in the ``.pyiceberg.yaml`` file, usually in the home directory.
It is possible to change properties of the catalog definition with the
``catalog_properties`` parameter:

```python
df = pd.read_iceberg(
    "my_table",
    catalog_name="my_catalog",
    catalog_properties={"s3.secret-access-key": "my_secret"},
)
```
It is also possible to fully specify the catalog in ``catalog_properties`` and not provide
a ``catalog_name``:

```python
df = pd.read_iceberg(
    "my_table",
    catalog_properties={
        "uri": "http://127.0.0.1:8181",
        "s3.endpoint": "http://127.0.0.1:9000",
    },
)
```
To create the ``DataFrame`` with only a subset of the columns:

```python
df = pd.read_iceberg(
    "my_table",
    catalog_name="my_catalog",
    selected_fields=["my_column_3", "my_column_7"]
)
```
This will execute the function faster, since other columns won't be read. And it will also
save memory, since the data from other columns won't be loaded into the underlying memory of
the ``DataFrame``.

To fetch only a subset of the rows, we can do it with the ``limit`` parameter:

```python
df = pd.read_iceberg(
    "my_table",
    catalog_name="my_catalog",
    limit=100,
)
```
This will create a ``DataFrame`` with 100 rows, assuming there are at least this number in
the table.

To fetch a subset of the rows based on a condition, this can be done using the ``row_filter``
parameter:

```python
df = pd.read_iceberg(
    "my_table",
    catalog_name="my_catalog",
    row_filter="distance > 10.0",
)
```
Reading a particular snapshot is also possible providing the snapshot ID as an argument to
``snapshot_id``.

To save a ``DataFrame`` to Iceberg, it can be done with the `DataFrame.to_iceberg`
method:

```python
df.to_iceberg("my_table", catalog_name="my_catalog")
```
To specify the catalog, it works in the same way as for `read_iceberg` with the
``catalog_name`` and ``catalog_properties`` parameters.

The location of the table can be specified with the ``location`` parameter:

```python
df.to_iceberg(
    "my_table",
    catalog_name="my_catalog",
    location="s://my-data-lake/my-iceberg-tables",
)
```
It is possible to add properties to the table snapshot by passing a dictionary to the
``snapshot_properties`` parameter.

More information about the Iceberg format can be found in the `Apache Iceberg official
page](https://iceberg.apache.org/)_.



## ORC
Similar to the `parquet <io.parquet>[ format, the `ORC Format](https://orc.apache.org/)_ is a binary columnar serialization
for data frames. It is designed to make reading data frames efficient. pandas provides both the reader and the writer for the
ORC format, [~pandas.read_orc` and `~pandas.DataFrame.to_orc`. This requires the `pyarrow](https://arrow.apache.org/docs/python/)_ library.

> **warning.capitalize():**
   * It is *highly recommended* to install pyarrow using conda due to some issues occurred by pyarrow.
   * `~pandas.read_orc` and `~pandas.DataFrame.to_orc` are not supported on Windows yet, you can find valid environments on `install optional dependencies <install.warn_orc>[.
   * For supported dtypes please refer to `supported ORC features in Arrow](https://arrow.apache.org/docs/cpp/orc.html#data-types)_.
   * Currently timezones in datetime columns are not preserved when a dataframe is converted into ORC files.

```python
df = pd.DataFrame(
    {
        "a": list("abc"),
        "b": list(range(1, 4)),
        "c": np.arange(4.0, 7.0, dtype="float64"),
        "d": [True, False, True],
        "e": pd.date_range("20130101", periods=3),
    }
)

df
df.dtypes
```
Write to an orc file.

```python
df.to_orc("example_pa.orc", engine="pyarrow")
```
Read from an orc file.

```python
result = pd.read_orc("example_pa.orc")

result.dtypes
```
Read only certain columns of an orc file.

```python
result = pd.read_orc(
    "example_pa.orc",
    columns=["a", "b"],
)
result.dtypes
```
```python
:suppress:

os.remove("example_pa.orc")
```


## SQL queries
The `pandas.io.sql` module provides a collection of query wrappers to both
facilitate data retrieval and to reduce dependency on DB-specific API.

Where available, users may first want to opt for `Apache Arrow ADBC
<https://arrow.apache.org/adbc/current/index.html>[_ drivers. These drivers
should provide the best performance, null handling, and type detection.

  .. versionadded:: 2.2.0

     Added native support for ADBC drivers

For a full list of ADBC drivers and their development status, see the `ADBC Driver
Implementation Status](https://arrow.apache.org/adbc/current/driver/status.html)
documentation.

Where an ADBC driver is not available or may be missing functionality,
users should opt for installing SQLAlchemy alongside their database driver library.
Examples of such drivers are [psycopg2](https://www.psycopg.org/)_
for PostgreSQL or [pymysql](https://github.com/PyMySQL/PyMySQL)_ for MySQL.
For [SQLite](https://docs.python.org/3/library/sqlite3.html)_ this is
included in Python's standard library by default.
You can find an overview of supported drivers for each SQL dialect in the
[SQLAlchemy docs](https://docs.sqlalchemy.org/en/latest/dialects/index.html)_.

If SQLAlchemy is not installed, you can use a `sqlite3.Connection` in place of
a SQLAlchemy engine, connection, or URI string.

See also some `cookbook examples <cookbook.sql>[ for some advanced strategies.

The key functions are:



    read_sql_table
    read_sql_query
    read_sql
    DataFrame.to_sql

> **note.capitalize():**
    The function `~pandas.read_sql` is a convenience wrapper around
    `~pandas.read_sql_table` and `~pandas.read_sql_query` (and for
    backward compatibility) and will delegate to specific function depending on
    the provided input (database table name or sql query).
    Table names do not need to be quoted if they have special characters.

In the following example, we use the `SQlite](https://www.sqlite.org/index.html)_ SQL database
engine. You can use a temporary SQLite database where data are stored in
"memory".

To connect using an ADBC driver you will want to install the [`adbc_driver_sqlite`` using your
package manager. Once installed, you can use the DBAPI interface provided by the ADBC driver
to connect to your database.

```python
import adbc_driver_sqlite.dbapi as sqlite_dbapi

# Create the connection
with sqlite_dbapi.connect("sqlite:///:memory:") as conn:
     df = pd.read_sql_table("data", conn)
```
To connect with SQLAlchemy you use the `create_engine` function to create an engine
object from database URI. You only need to create the engine once per database you are
connecting to.
For more information on `create_engine` and the URI formatting, see the examples
below and the SQLAlchemy `documentation](https://docs.sqlalchemy.org/en/latest/core/engines.html)_

[``python
from sqlalchemy import create_engine

# Create your engine.
engine = create_engine("sqlite:///:memory:")
```
If you want to manage your own connections you can pass one of those instead. The example below opens a
connection to the database using a Python context manager that automatically closes the connection after
the block has completed.
See the `SQLAlchemy docs](https://docs.sqlalchemy.org/en/latest/core/connections.html#basic-usage)_
for an explanation of how the database connection is handled.

[``python
with engine.connect() as conn, conn.begin():
    data = pd.read_sql_table("data", conn)
```
> **warning.capitalize():**
        When you open a connection to a database you are also responsible for closing it.
        Side effects of leaving a connection open may include locking the database or
        other breaking behaviour.

Writing DataFrames
''''''''''''''''''

Assuming the following data is in a ``DataFrame`` ``data``, we can insert it into
the database using `~pandas.DataFrame.to_sql`.

+-----+------------+-------+-------+-------+
| id  |    Date    | Col_1 | Col_2 | Col_3 |
+=====+============+=======+=======+=======+
| 26  | 2012-10-18 |   X   |  25.7 | True  |
+-----+------------+-------+-------+-------+
| 42  | 2012-10-19 |   Y   | -12.4 | False |
+-----+------------+-------+-------+-------+
| 63  | 2012-10-20 |   Z   |  5.73 | True  |
+-----+------------+-------+-------+-------+


```python
import datetime

c = ["id", "Date", "Col_1", "Col_2", "Col_3"]
d = [
    (26, datetime.datetime(2010, 10, 18), "X", 27.5, True),
    (42, datetime.datetime(2010, 10, 19), "Y", -12.5, False),
    (63, datetime.datetime(2010, 10, 20), "Z", 5.73, True),
]

data = pd.DataFrame(d, columns=c)

data
data.to_sql("data", con=engine)
```
With some databases, writing large DataFrames can result in errors due to
packet size limitations being exceeded. This can be avoided by setting the
``chunksize`` parameter when calling ``to_sql``.  For example, the following
writes ``data`` to the database in batches of 1000 rows at a time:

```python
data.to_sql("data_chunked", con=engine, chunksize=1000)
```
SQL data types
++++++++++++++

Ensuring consistent data type management across SQL databases is challenging.
Not every SQL database offers the same types, and even when they do the implementation
of a given type can vary in ways that have subtle effects on how types can be
preserved.

For the best odds at preserving database types users are advised to use
ADBC drivers when available. The Arrow type system offers a wider array of
types that more closely match database types than the historical pandas/NumPy
type system. To illustrate, note this (non-exhaustive) listing of types
available in different databases and pandas backends:

+-----------------+-----------------------+----------------+---------+
|numpy/pandas     |arrow                  |postgres        |sqlite   |
+=================+=======================+================+=========+
|int16/Int16      |int16                  |SMALLINT        |INTEGER  |
+-----------------+-----------------------+----------------+---------+
|int32/Int32      |int32                  |INTEGER         |INTEGER  |
+-----------------+-----------------------+----------------+---------+
|int64/Int64      |int64                  |BIGINT          |INTEGER  |
+-----------------+-----------------------+----------------+---------+
|float32          |float32                |REAL            |REAL     |
+-----------------+-----------------------+----------------+---------+
|float64          |float64                |DOUBLE PRECISION|REAL     |
+-----------------+-----------------------+----------------+---------+
|object           |string                 |TEXT            |TEXT     |
+-----------------+-----------------------+----------------+---------+
|bool             |``bool_``              |BOOLEAN         |         |
+-----------------+-----------------------+----------------+---------+
|datetime64[ns]   |timestamp(us)          |TIMESTAMP       |         |
+-----------------+-----------------------+----------------+---------+
|datetime64[ns,tz]|timestamp(us,tz)       |TIMESTAMPTZ     |         |
+-----------------+-----------------------+----------------+---------+
|                 |date32                 |DATE            |         |
+-----------------+-----------------------+----------------+---------+
|                 |month_day_nano_interval|INTERVAL        |         |
+-----------------+-----------------------+----------------+---------+
|                 |binary                 |BINARY          |BLOB     |
+-----------------+-----------------------+----------------+---------+
|                 |decimal128             |DECIMAL [#f1]_  |         |
+-----------------+-----------------------+----------------+---------+
|                 |list                   |ARRAY [#f1]_    |         |
+-----------------+-----------------------+----------------+---------+
|                 |struct                 |COMPOSITE TYPE  |         |
|                 |                       | [#f1]_         |         |
+-----------------+-----------------------+----------------+---------+



.. [#f1] Not implemented as of writing, but theoretically possible

If you are interested in preserving database types as best as possible
throughout the lifecycle of your DataFrame, users are encouraged to
leverage the ``dtype_backend="pyarrow"`` argument of `~pandas.read_sql`



   # for roundtripping
   with pg_dbapi.connect(uri) as conn:
       df2 = pd.read_sql("pandas_table", conn, dtype_backend="pyarrow")

This will prevent your data from being converted to the traditional pandas/NumPy
type system, which often converts SQL types in ways that make them impossible to
round-trip.

In case an ADBC driver is not available, `~pandas.DataFrame.to_sql`
will try to map your data to an appropriate SQL data type based on the dtype of
the data. When you have columns of dtype ``object``, pandas will try to infer
the data type.

You can always override the default type by specifying the desired SQL type of
any of the columns by using the ``dtype`` argument. This argument needs a
dictionary mapping column names to SQLAlchemy types (or strings for the sqlite3
fallback mode).
For example, specifying to use the sqlalchemy ``String`` type instead of the
default ``Text`` type for string columns:

```python
from sqlalchemy.types import String

data.to_sql("data_dtype", con=engine, dtype={"Col_1": String})
```
> **note.capitalize():**
    Due to the limited support for timedelta's in the different database
    flavors, columns with type ``timedelta64`` will be written as integer
    values as nanoseconds to the database and a warning will be raised. The only
    exception to this is when using the ADBC PostgreSQL driver in which case a
    timedelta will be written to the database as an ``INTERVAL``

> **note.capitalize():**
    Columns of ``category`` dtype will be converted to the dense representation
    as you would get with ``np.asarray(categorical)`` (e.g. for string categories
    this gives an array of strings).
    Because of this, reading the database table back in does **not** generate
    a categorical.



Datetime data types
'''''''''''''''''''

Using ADBC or SQLAlchemy, `~pandas.DataFrame.to_sql` is capable of writing
datetime data that is timezone naive or timezone aware. However, the resulting
data stored in the database ultimately depends on the supported data type
for datetime data of the database system being used.

The following table lists supported data types for datetime data for some
common databases. Other database dialects may have different data types for
datetime data.

===========   =============================================  ===================
Database      SQL Datetime Types                             Timezone Support
===========   =============================================  ===================
SQLite        ``TEXT``                                       No
MySQL         ``TIMESTAMP`` or ``DATETIME``                  No
PostgreSQL    ``TIMESTAMP`` or ``TIMESTAMP WITH TIME ZONE``  Yes
===========   =============================================  ===================

When writing timezone aware data to databases that do not support timezones,
the data will be written as timezone naive timestamps that are in local time
with respect to the timezone.

`~pandas.read_sql_table` is also capable of reading datetime data that is
timezone aware or naive. When reading ``TIMESTAMP WITH TIME ZONE`` types, pandas
will convert the data to UTC.



Insertion method
++++++++++++++++

The parameter ``method`` controls the SQL insertion clause used.
Possible values are:

- ``None``: Uses standard SQL ``INSERT`` clause (one per row).
- ``'multi'``: Pass multiple values in a single ``INSERT`` clause.
  It uses a *special* SQL syntax not supported by all backends.
  This usually provides better performance for analytic databases
  like *Presto* and *Redshift*, but has worse performance for
  traditional SQL backend if the table contains many columns.
  For more information check the SQLAlchemy `documentation
 ](https://docs.sqlalchemy.org/en/latest/core/dml.html#sqlalchemy.sql.expression.Insert.values.params.*args)_.
- callable with signature ``(pd_table, conn, keys, data_iter)``:
  This can be used to implement a more performant insertion method based on
  specific backend dialect features.

Example of a callable using PostgreSQL `COPY clause
<https://www.postgresql.org/docs/current/sql-copy.html>`__::

  # Alternative to_sql() *method* for DBs that support COPY FROM
  import csv
  from io import StringIO

  def psql_insert_copy(table, conn, keys, data_iter):
      """
      Execute SQL statement inserting data

      Parameters
      ----------
      table : pandas.io.sql.SQLTable
      conn : sqlalchemy.engine.Engine or sqlalchemy.engine.Connection
      keys : list of str
          Column names
      data_iter : Iterable that iterates the values to be inserted
      """
      # gets a DBAPI connection that can provide a cursor
      dbapi_conn = conn.connection
      with dbapi_conn.cursor() as cur:
          s_buf = StringIO()
          writer = csv.writer(s_buf)
          writer.writerows(data_iter)
          s_buf.seek(0)

          columns = ', '.join(['"{}"'.format(k) for k in keys])
          if table.schema:
              table_name = '{}.{}'.format(table.schema, table.name)
          else:
              table_name = table.name

          sql = 'COPY {} ({}) FROM STDIN WITH CSV'.format(
              table_name, columns)
          cur.copy_expert(sql=sql, file=s_buf)

Reading tables
''''''''''''''

`~pandas.read_sql_table` will read a database table given the
table name and optionally a subset of columns to read.

> **note.capitalize():**
    In order to use `~pandas.read_sql_table`, you **must** have the
    ADBC driver or SQLAlchemy optional dependency installed.

```python
pd.read_sql_table("data", engine)
```
> **note.capitalize():**
  ADBC drivers will map database types directly back to arrow types. For other drivers
  note that pandas infers column dtypes from query outputs, and not by looking
  up data types in the physical database schema. For example, assume ``userid``
  is an integer column in a table. Then, intuitively, ``select userid ...`` will
  return integer-valued series, while ``select cast(userid as text) ...`` will
  return object-valued (str) series. Accordingly, if the query output is empty,
  then all resulting columns will be returned as object-valued (since they are
  most general). If you foresee that your query will sometimes generate an empty
  result, you may want to explicitly typecast afterwards to ensure dtype
  integrity.

You can also specify the name of the column as the ``DataFrame`` index,
and specify a subset of columns to be read.

```python
pd.read_sql_table("data", engine, index_col="id")
pd.read_sql_table("data", engine, columns=["Col_1", "Col_2"])
```
And you can explicitly force columns to be parsed as dates:

```python
pd.read_sql_table("data", engine, parse_dates=["Date"])
```
If needed you can explicitly specify a format string, or a dict of arguments
to pass to `pandas.to_datetime`:

```python
pd.read_sql_table("data", engine, parse_dates={"Date": "%Y-%m-%d"})
pd.read_sql_table(
    "data",
    engine,
    parse_dates={"Date": {"format": "%Y-%m-%d %H:%M:%S"}},
)
```
You can check if a table exists using `~pandas.io.sql.has_table`

Schema support
''''''''''''''

Reading from and writing to different schemas is supported through the ``schema``
keyword in the `~pandas.read_sql_table` and `~pandas.DataFrame.to_sql`
functions. Note however that this depends on the database flavor (sqlite does not
have schemas). For example:

```python
df.to_sql(name="table", con=engine, schema="other_schema")
pd.read_sql_table("table", engine, schema="other_schema")
```
Querying
''''''''

You can query using raw SQL in the `~pandas.read_sql_query` function.
In this case you must use the SQL variant appropriate for your database.
When using SQLAlchemy, you can also pass SQLAlchemy Expression language constructs,
which are database-agnostic.

```python
pd.read_sql_query("SELECT * FROM data", engine)
```
Of course, you can specify a more "complex" query.

```python
pd.read_sql_query("SELECT id, Col_1, Col_2 FROM data WHERE id = 42;", engine)
```
The `~pandas.read_sql_query` function supports a ``chunksize`` argument.
Specifying this will return an iterator through chunks of the query result:

```python
df = pd.DataFrame(np.random.randn(20, 3), columns=list("abc"))
df.to_sql(name="data_chunks", con=engine, index=False)
```
```python
for chunk in pd.read_sql_query("SELECT * FROM data_chunks", engine, chunksize=5):
    print(chunk)
```
Engine connection examples
''''''''''''''''''''''''''

To connect with SQLAlchemy you use the `create_engine` function to create an engine
object from database URI. You only need to create the engine once per database you are
connecting to.

```python
from sqlalchemy import create_engine

engine = create_engine("postgresql://scott:tiger@localhost:5432/mydatabase")

engine = create_engine("mysql+mysqldb://scott:tiger@localhost/foo")

engine = create_engine("oracle://scott:tiger@127.0.0.1:1521/sidname")

engine = create_engine("mssql+pyodbc://mydsn")

# sqlite://<nohostname>/<path>
# where <path> is relative:
engine = create_engine("sqlite:///foo.db")

# or absolute, starting with a slash:
engine = create_engine("sqlite:////absolute/path/to/foo.db")
[``
For more information see the examples the SQLAlchemy `documentation](https://docs.sqlalchemy.org/en/latest/core/engines.html)_


Advanced SQLAlchemy queries
'''''''''''''''''''''''''''

You can use SQLAlchemy constructs to describe your query.

Use [sqlalchemy.text` to specify query parameters in a backend-neutral way

```python
import sqlalchemy as sa

pd.read_sql(
    sa.text("SELECT * FROM data where Col_1=:col1"), engine, params={"col1": "X"}
)
```
If you have an SQLAlchemy description of your database you can express where conditions using SQLAlchemy expressions

```python
metadata = sa.MetaData()
data_table = sa.Table(
    "data",
    metadata,
    sa.Column("index", sa.Integer),
    sa.Column("Date", sa.DateTime),
    sa.Column("Col_1", sa.String),
    sa.Column("Col_2", sa.Float),
    sa.Column("Col_3", sa.Boolean),
)

pd.read_sql(sa.select(data_table).where(data_table.c.Col_3 is True), engine)
```
You can combine SQLAlchemy expressions with parameters passed to `read_sql` using `sqlalchemy.bindparam`

```python
import datetime as dt

expr = sa.select(data_table).where(data_table.c.Date > sa.bindparam("date"))
pd.read_sql(expr, engine, params={"date": dt.datetime(2010, 10, 18)})
```
Sqlite fallback
'''''''''''''''

The use of sqlite is supported without using SQLAlchemy.
This mode requires a Python database adapter which respect the `Python
DB-API](https://www.python.org/dev/peps/pep-0249/)_.

You can create connections like so:

[``python
import sqlite3

con = sqlite3.connect(":memory:")
```
And then issue the following queries:

```python
data.to_sql("data", con)
pd.read_sql_query("SELECT * FROM data", con)
```


## Google BigQuery
The ``pandas-gbq`` package provides functionality to read/write from Google BigQuery.

Full documentation can be found `here](https://pandas-gbq.readthedocs.io/en/latest/)_.



## STATA format


Writing to stata format
'''''''''''''''''''''''

The method `.DataFrame.to_stata` will write a DataFrame
into a .dta file. The format version of this file is always 115 (Stata 12).

```python
df = pd.DataFrame(np.random.randn(10, 2), columns=list("AB"))
df.to_stata("stata.dta")
```
*Stata* data files have limited data type support; only strings with
244 or fewer characters, ``int8``, ``int16``, ``int32``, ``float32``
and ``float64`` can be stored in ``.dta`` files.  Additionally,
*Stata* reserves certain values to represent missing data. Exporting a
non-missing value that is outside of the permitted range in Stata for
a particular data type will retype the variable to the next larger
size.  For example, ``int8`` values are restricted to lie between -127
and 100 in Stata, and so variables with values above 100 will trigger
a conversion to ``int16``. ``nan`` values in floating points data
types are stored as the basic missing data type (``.`` in *Stata*).

> **note.capitalize():**
    It is not possible to export missing data values for integer data types.


The *Stata* writer gracefully handles other data types including ``int64``,
``bool``, ``uint8``, ``uint16``, ``uint32`` by casting to
the smallest supported type that can represent the data.  For example, data
with a type of ``uint8`` will be cast to ``int8`` if all values are less than
100 (the upper bound for non-missing ``int8`` data in *Stata*), or, if values are
outside of this range, the variable is cast to ``int16``.


> **warning.capitalize():**
   Conversion from ``int64`` to ``float64`` may result in a loss of precision
   if ``int64`` values are larger than 2**53.

> **warning.capitalize():**
  `~pandas.io.stata.StataWriter` and
  `.DataFrame.to_stata` only support fixed width
  strings containing up to 244 characters, a limitation imposed by the version
  115 dta file format. Attempting to write *Stata* dta files with strings
  longer than 244 characters raises a ``ValueError``.



Reading from Stata format
'''''''''''''''''''''''''

The top-level function ``read_stata`` will read a dta file and return
either a ``DataFrame`` or a `pandas.api.typing.StataReader` that can
be used to read the file incrementally.

```python
pd.read_stata("stata.dta")
```
Specifying a ``chunksize`` yields a
`pandas.api.typing.StataReader` instance that can be used to
read ``chunksize`` lines from the file at a time.  The ``StataReader``
object can be used as an iterator.

```python
with pd.read_stata("stata.dta", chunksize=3) as reader:
    for df in reader:
        print(df.shape)
```
For more fine-grained control, use ``iterator=True`` and specify
``chunksize`` with each call to
`~pandas.io.stata.StataReader.read`.

```python
with pd.read_stata("stata.dta", iterator=True) as reader:
    chunk1 = reader.read(5)
    chunk2 = reader.read(5)
```
Currently the ``index`` is retrieved as a column.

The parameter ``convert_categoricals`` indicates whether value labels should be
read and used to create a ``Categorical`` variable from them. Value labels can
also be retrieved by the function ``value_labels``, which requires `~pandas.io.stata.StataReader.read`
to be called before use.

The parameter ``convert_missing`` indicates whether missing value
representations in Stata should be preserved.  If ``False`` (the default),
missing values are represented as ``np.nan``.  If ``True``, missing values are
represented using ``StataMissingValue`` objects, and columns containing missing
values will have ``object`` data type.

> **note.capitalize():**
   `~pandas.read_stata` and
   `~pandas.io.stata.StataReader` support .dta formats 113-115
   (Stata 10-12), 117 (Stata 13), and 118 (Stata 14).

> **note.capitalize():**
   Setting ``preserve_dtypes=False`` will upcast to the standard pandas data types:
   ``int64`` for all integer types and ``float64`` for floating point data.  By default,
   the Stata data types are preserved when importing.

> **note.capitalize():**
   All `~pandas.io.stata.StataReader` objects, whether created by `~pandas.read_stata`
   (when using ``iterator=True`` or ``chunksize``) or instantiated by hand, must be used as context
   managers (e.g. the ``with`` statement).
   While the `~pandas.io.stata.StataReader.close` method is available, its use is unsupported.
   It is not part of the public API and will be removed in with future without warning.

```python
:suppress:

os.remove("stata.dta")
```


Categorical data
++++++++++++++++

``Categorical`` data can be exported to *Stata* data files as value labeled data.
The exported data consists of the underlying category codes as integer data values
and the categories as value labels.  *Stata* does not have an explicit equivalent
to a ``Categorical`` and information about *whether* the variable is ordered
is lost when exporting.

> **warning.capitalize():**
    *Stata* only supports string value labels, and so ``str`` is called on the
    categories when exporting data.  Exporting ``Categorical`` variables with
    non-string categories produces a warning, and can result a loss of
    information if the ``str`` representations of the categories are not unique.

Labeled data can similarly be imported from *Stata* data files as ``Categorical``
variables using the keyword argument ``convert_categoricals`` (``True`` by default).
The keyword argument ``order_categoricals`` (``True`` by default) determines
whether imported ``Categorical`` variables are ordered.

> **note.capitalize():**
    When importing categorical data, the values of the variables in the *Stata*
    data file are not preserved since ``Categorical`` variables always
    use integer data types between ``-1`` and ``n-1`` where ``n`` is the number
    of categories. If the original values in the *Stata* data file are required,
    these can be imported by setting ``convert_categoricals=False``, which will
    import original data (but not the variable labels). The original values can
    be matched to the imported categorical data since there is a simple mapping
    between the original *Stata* data values and the category codes of imported
    Categorical variables: missing values are assigned code ``-1``, and the
    smallest original value is assigned ``0``, the second smallest is assigned
    ``1`` and so on until the largest original value is assigned the code ``n-1``.

> **note.capitalize():**
    *Stata* supports partially labeled series. These series have value labels for
    some but not all data values. Importing a partially labeled series will produce
    a ``Categorical`` with string categories for the values that are labeled and
    numeric categories for values with no label.





## SAS formats
The top-level function `read_sas` can read (but not write) SAS
XPORT (.xpt) and SAS7BDAT (.sas7bdat) format files.

SAS files only contain two value types: ASCII text and floating point
values (usually 8 bytes but sometimes truncated).  For xport files,
there is no automatic type conversion to integers, dates, or
categoricals.  For SAS7BDAT files, the format codes may allow date
variables to be automatically converted to dates.  By default the
whole file is read and returned as a ``DataFrame``.

Specify a ``chunksize`` or use ``iterator=True`` to obtain reader
objects (``XportReader`` or ``SAS7BDATReader``) for incrementally
reading the file.  The reader objects also have attributes that
contain additional information about the file and its variables.

Read a SAS7BDAT file:

```python
df = pd.read_sas("sas_data.sas7bdat")
```
Obtain an iterator and read an XPORT file 100,000 lines at a time:

```python
def do_something(chunk):
    pass


with pd.read_sas("sas_xport.xpt", chunk=100000) as rdr:
    for chunk in rdr:
        do_something(chunk)
```
The specification_ for the xport file format is available from the SAS
web site.

.. _specification: https://support.sas.com/content/dam/SAS/support/en/technical-papers/record-layout-of-a-sas-version-5-or-6-data-set-in-sas-transport-xport-format.pdf

No official documentation is available for the SAS7BDAT format.





## SPSS formats
The top-level function `read_spss` can read (but not write) SPSS
SAV (.sav) and  ZSAV (.zsav) format files.

SPSS files contain column names. By default the
whole file is read, categorical columns are converted into ``pd.Categorical``,
and a ``DataFrame`` with all columns is returned.

Specify the ``usecols`` parameter to obtain a subset of columns. Specify ``convert_categoricals=False``
to avoid converting categorical columns into ``pd.Categorical``.

Read an SPSS file:

```python
df = pd.read_spss("spss_data.sav")
```
Extract a subset of columns contained in ``usecols`` from an SPSS file and
avoid converting categorical columns into ``pd.Categorical``:

```python
df = pd.read_spss(
    "spss_data.sav",
    usecols=["foo", "bar"],
    convert_categoricals=False,
)
```
More information about the SAV and ZSAV file formats is available here_.

.. _here: https://www.ibm.com/docs/en/spss-statistics/22.0.0



## Other file formats
pandas itself only supports IO with a limited set of file formats that map
cleanly to its tabular data model. For reading and writing other file formats
into and from pandas, we recommend these packages from the broader community.

netCDF
''''''

xarray_ provides data structures inspired by the pandas ``DataFrame`` for working
with multi-dimensional datasets, with a focus on the netCDF file format and
easy conversion to and from pandas.

.. _xarray: https://xarray.pydata.org/en/stable/



## Performance considerations
This is an informal comparison of various IO methods, using pandas
0.24.2. Timings are machine dependent and small differences should be
ignored.



   In [1]: sz = 1000000
   In [2]: df = pd.DataFrame({'A': np.random.randn(sz), 'B': [1] * sz})

   In [3]: df.info()
   <class 'pandas.DataFrame'>
   RangeIndex: 1000000 entries, 0 to 999999
   Data columns (total 2 columns):
   A    1000000 non-null float64
   B    1000000 non-null int64
   dtypes: float64(1), int64(1)
   memory usage: 15.3 MB

The following test functions will be used below to compare the performance of several IO methods:

```python
import numpy as np

import os

sz = 1000000
df = pd.DataFrame({"A": np.random.randn(sz), "B": [1] * sz})

sz = 1000000
np.random.seed(42)
df = pd.DataFrame({"A": np.random.randn(sz), "B": [1] * sz})


def test_sql_write(df):
    if os.path.exists("test.sql"):
        os.remove("test.sql")
    sql_db = sqlite3.connect("test.sql")
    df.to_sql(name="test_table", con=sql_db)
    sql_db.close()


def test_sql_read():
    sql_db = sqlite3.connect("test.sql")
    pd.read_sql_query("select * from test_table", sql_db)
    sql_db.close()


def test_hdf_fixed_write(df):
    df.to_hdf("test_fixed.hdf", key="test", mode="w")


def test_hdf_fixed_read():
    pd.read_hdf("test_fixed.hdf", "test")


def test_hdf_fixed_write_compress(df):
    df.to_hdf("test_fixed_compress.hdf", key="test", mode="w", complib="blosc")


def test_hdf_fixed_read_compress():
    pd.read_hdf("test_fixed_compress.hdf", "test")


def test_hdf_table_write(df):
    df.to_hdf("test_table.hdf", key="test", mode="w", format="table")


def test_hdf_table_read():
    pd.read_hdf("test_table.hdf", "test")


def test_hdf_table_write_compress(df):
    df.to_hdf(
        "test_table_compress.hdf", key="test", mode="w", complib="blosc", format="table"
    )


def test_hdf_table_read_compress():
    pd.read_hdf("test_table_compress.hdf", "test")


def test_csv_write(df):
    df.to_csv("test.csv", mode="w")


def test_csv_read():
    pd.read_csv("test.csv", index_col=0)


def test_feather_write(df):
    df.to_feather("test.feather")


def test_feather_read():
    pd.read_feather("test.feather")


def test_pickle_write(df):
    df.to_pickle("test.pkl")


def test_pickle_read():
    pd.read_pickle("test.pkl")


def test_pickle_write_compress(df):
    df.to_pickle("test.pkl.compress", compression="xz")


def test_pickle_read_compress():
    pd.read_pickle("test.pkl.compress", compression="xz")


def test_parquet_write(df):
    df.to_parquet("test.parquet")


def test_parquet_read():
    pd.read_parquet("test.parquet")
```
When writing, the top three functions in terms of speed are ``test_feather_write``, ``test_hdf_fixed_write`` and ``test_hdf_fixed_write_compress``.



   In [4]: %timeit test_sql_write(df)
   3.29 s ± 43.2 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)

   In [5]: %timeit test_hdf_fixed_write(df)
   19.4 ms ± 560 µs per loop (mean ± std. dev. of 7 runs, 1 loop each)

   In [6]: %timeit test_hdf_fixed_write_compress(df)
   19.6 ms ± 308 µs per loop (mean ± std. dev. of 7 runs, 10 loops each)

   In [7]: %timeit test_hdf_table_write(df)
   449 ms ± 5.61 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)

   In [8]: %timeit test_hdf_table_write_compress(df)
   448 ms ± 11.9 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)

   In [9]: %timeit test_csv_write(df)
   3.66 s ± 26.2 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)

   In [10]: %timeit test_feather_write(df)
   9.75 ms ± 117 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)

   In [11]: %timeit test_pickle_write(df)
   30.1 ms ± 229 µs per loop (mean ± std. dev. of 7 runs, 10 loops each)

   In [12]: %timeit test_pickle_write_compress(df)
   4.29 s ± 15.9 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)

   In [13]: %timeit test_parquet_write(df)
   67.6 ms ± 706 µs per loop (mean ± std. dev. of 7 runs, 10 loops each)

When reading, the top three functions in terms of speed are ``test_feather_read``, ``test_pickle_read`` and
``test_hdf_fixed_read``.




   In [14]: %timeit test_sql_read()
   1.77 s ± 17.7 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)

   In [15]: %timeit test_hdf_fixed_read()
   19.4 ms ± 436 µs per loop (mean ± std. dev. of 7 runs, 10 loops each)

   In [16]: %timeit test_hdf_fixed_read_compress()
   19.5 ms ± 222 µs per loop (mean ± std. dev. of 7 runs, 10 loops each)

   In [17]: %timeit test_hdf_table_read()
   38.6 ms ± 857 µs per loop (mean ± std. dev. of 7 runs, 10 loops each)

   In [18]: %timeit test_hdf_table_read_compress()
   38.8 ms ± 1.49 ms per loop (mean ± std. dev. of 7 runs, 10 loops each)

   In [19]: %timeit test_csv_read()
   452 ms ± 9.04 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)

   In [20]: %timeit test_feather_read()
   12.4 ms ± 99.7 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)

   In [21]: %timeit test_pickle_read()
   18.4 ms ± 191 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)

   In [22]: %timeit test_pickle_read_compress()
   915 ms ± 7.48 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)

   In [23]: %timeit test_parquet_read()
   24.4 ms ± 146 µs per loop (mean ± std. dev. of 7 runs, 10 loops each)


The files ``test.pkl.compress``, ``test.parquet`` and ``test.feather`` took the least space on disk (in bytes).



    29519500 Oct 10 06:45 test.csv
    16000248 Oct 10 06:45 test.feather
    8281983  Oct 10 06:49 test.parquet
    16000857 Oct 10 06:47 test.pkl
    7552144  Oct 10 06:48 test.pkl.compress
    34816000 Oct 10 06:42 test.sql
    24009288 Oct 10 06:43 test_fixed.hdf
    24009288 Oct 10 06:43 test_fixed_compress.hdf
    24458940 Oct 10 06:44 test_table.hdf
    24458940 Oct 10 06:44 test_table_compress.hdf

---

# PyArrow Functionality
pandas can utilize [PyArrow](https://arrow.apache.org/docs/python/index.html)_ to extend functionality and improve the performance
of various APIs. This includes:

* More extensive [data types](https://arrow.apache.org/docs/python/api/datatypes.html)_ compared to NumPy
* Missing data support (NA) for all data types
* Performant IO reader integration
* Facilitate interoperability with other dataframe libraries based on the Apache Arrow specification (e.g. polars, cuDF)

To use this functionality, please ensure you have `installed the minimum supported PyArrow version. <install.optional_dependencies>`


## Data Structure Integration
A `Series`, `Index`, or the columns of a `DataFrame` can be directly backed by a :external+pyarrow:py`pyarrow.ChunkedArray`
which is similar to a NumPy array. To construct these from the main pandas data structures, you can pass in a string of the type followed by
``[pyarrow]``, e.g. ``"int64[pyarrow]"`` into the ``dtype`` parameter

```python
ser = pd.Series([-1.5, 0.2, None], dtype="float32[pyarrow]")
ser

idx = pd.Index([True, None], dtype="bool[pyarrow]")
idx

df = pd.DataFrame([[1, 2], [3, 4]], dtype="uint64[pyarrow]")
df
```
> **note.capitalize():**
   The string alias ``"string[pyarrow]"`` maps to ``pd.StringDtype("pyarrow")`` which is not equivalent to
   specifying ``dtype=pd.ArrowDtype(pa.string())``. Generally, operations on the data will behave similarly
   except ``pd.StringDtype("pyarrow")`` can return NumPy-backed nullable types while ``pd.ArrowDtype(pa.string())``
   will return `ArrowDtype`.

   ```python
import pyarrow as pa
data = list("abc")
ser_sd = pd.Series(data, dtype="string[pyarrow]")
ser_ad = pd.Series(data, dtype=pd.ArrowDtype(pa.string()))
ser_ad.dtype == ser_sd.dtype
ser_sd.str.contains("a")
ser_ad.str.contains("a")
```
For PyArrow types that accept parameters, you can pass in a PyArrow type with those parameters
into `ArrowDtype` to use in the ``dtype`` parameter.

```python
import pyarrow as pa
list_str_type = pa.list_(pa.string())
ser = pd.Series([["hello"], ["there"]], dtype=pd.ArrowDtype(list_str_type))
ser
```
```python
from datetime import time
idx = pd.Index([time(12, 30), None], dtype=pd.ArrowDtype(pa.time64("us")))
idx
```
```python
from decimal import Decimal
decimal_type = pd.ArrowDtype(pa.decimal128(3, scale=2))
data = [[Decimal("3.19"), None], [None, Decimal("-1.23")]]
df = pd.DataFrame(data, dtype=decimal_type)
df
```
If you already have an :external+pyarrow:py`pyarrow.Array` or :external+pyarrow:py`pyarrow.ChunkedArray`,
you can pass it into `.arrays.ArrowExtensionArray` to construct the associated `Series`, `Index`
or `DataFrame` object.

```python
pa_array = pa.array(
    [{"1": "2"}, {"10": "20"}, None],
    type=pa.map_(pa.string(), pa.string()),
)
ser = pd.Series(pd.arrays.ArrowExtensionArray(pa_array))
ser
```
To retrieve a pyarrow :external+pyarrow:py`pyarrow.ChunkedArray` from a `Series` or `Index`, you can call
the pyarrow array constructor on the `Series` or `Index`.

```python
ser = pd.Series([1, 2, None], dtype="uint8[pyarrow]")
pa.array(ser)

idx = pd.Index(ser)
pa.array(idx)
```
To convert a :external+pyarrow:py`pyarrow.Table` to a `DataFrame`, you can call the
:external+pyarrow:py`pyarrow.Table.to_pandas` method with ``types_mapper=pd.ArrowDtype``.

```python
table = pa.table([pa.array([1, 2, 3], type=pa.int64())], names=["a"])

df = table.to_pandas(types_mapper=pd.ArrowDtype)
df
df.dtypes
```
## Operations
PyArrow data structure integration is implemented through pandas' `~pandas.api.extensions.ExtensionArray` `interface <extending.extension-types>[;
therefore, supported functionality exists where this interface is integrated within the pandas API. Additionally, this functionality
is accelerated with PyArrow `compute functions](https://arrow.apache.org/docs/python/api/compute.html)_ where available. This includes:

* Numeric aggregations
* Numeric arithmetic
* Numeric rounding
* Logical and comparison functions
* String functionality
* Datetime functionality

The following are just some examples of operations that are accelerated by native PyArrow compute functions.

```python
import pyarrow as pa
ser = pd.Series([-1.545, 0.211, None], dtype="float32[pyarrow]")
ser.mean()
ser + ser
ser > (ser + 1)

ser.dropna()
ser.isna()
ser.fillna(0)
```
```python
ser_str = pd.Series(["a", "b", None], dtype=pd.ArrowDtype(pa.string()))
ser_str.str.startswith("a")
```
```python
from datetime import datetime
pa_type = pd.ArrowDtype(pa.timestamp("ns"))
ser_dt = pd.Series([datetime(2022, 1, 1), None], dtype=pa_type)
ser_dt.dt.strftime("%Y-%m")
```
## I/O Reading
PyArrow also provides IO reading functionality that has been integrated into several pandas IO readers. The following
functions provide an ``engine`` keyword that can dispatch to PyArrow to accelerate reading from an IO source.

* `read_csv`
* `read_feather`
* `read_json`
* `read_orc`
* `read_parquet`
* `read_table` (experimental)

```python
import io
data = io.StringIO("""a,b,c
   1,2.5,True
   3,4.5,False
""")
df = pd.read_csv(data, engine="pyarrow")
df
```
By default, these functions and all other IO reader functions return NumPy-backed data. These readers can return
PyArrow-backed data by specifying the parameter ``dtype_backend="pyarrow"``. A reader does not need to set
``engine="pyarrow"`` to necessarily return PyArrow-backed data.

```python
import io
data = io.StringIO("""a,b,c,d,e,f,g,h,i
    1,2.5,True,a,,,,,
    3,4.5,False,b,6,7.5,True,a,
""")
df_pyarrow = pd.read_csv(data, dtype_backend="pyarrow")
df_pyarrow.dtypes
```
Several non-IO reader functions can also use the ``dtype_backend`` argument to return PyArrow-backed data including:

* `to_numeric`
* `DataFrame.convert_dtypes`
* `Series.convert_dtypes`

---

# Indexing and selecting data
The axis labeling information in pandas objects serves many purposes:

* Identifies data (i.e. provides *metadata*) using known indicators,
  important for analysis, visualization, and interactive console display.
* Enables automatic and explicit data alignment.
* Allows intuitive getting and setting of subsets of the data set.

In this section, we will focus on the final point: namely, how to slice, dice,
and generally get and set subsets of pandas objects. The primary focus will be
on Series and DataFrame as they have received more development attention in
this area.

> **note.capitalize():**
   The Python and NumPy indexing operators ``[]`` and attribute operator ``.``
   provide quick and easy access to pandas data structures across a wide range
   of use cases. This makes interactive work intuitive, as there's little new
   to learn if you already know how to deal with Python dictionaries and NumPy
   arrays. However, since the type of the data to be accessed isn't known in
   advance, directly using standard operators has some optimization limits. For
   production code, we recommended that you take advantage of the optimized
   pandas data access methods exposed in this chapter.

See the `MultiIndex / Advanced Indexing <advanced>` for ``MultiIndex`` and more advanced indexing documentation.

See the `cookbook<cookbook.selection>` for some advanced strategies.



## Different choices for indexing
Object selection has had a number of user-requested additions in order to
support more explicit location based indexing. pandas now supports three types
of multi-axis indexing.

* ``.loc`` is primarily label based, but may also be used with a boolean array. ``.loc`` will raise ``KeyError`` when the items are not found. Allowed inputs are:

    * A single label, e.g. ``5`` or ``'a'`` (Note that ``5`` is interpreted as a
      *label* of the index. This use is **not** an integer position along the
      index.).
    * A list or array of labels ``['a', 'b', 'c']``.
    * A slice object with labels ``'a':'f'`` (Note that contrary to usual Python
      slices, **both** the start and the stop are included, when present in the
      index! See `Slicing with labels <indexing.slicing_with_labels>`
      and `Endpoints are inclusive <advanced.endpoints_are_inclusive>`.)
    * A boolean array (any ``NA`` values will be treated as ``False``).
    * A ``callable`` function with one argument (the calling Series or DataFrame) and
      that returns valid output for indexing (one of the above).
    * A tuple of row (and column) indices whose elements are one of the
      above inputs.

  See more at `Selection by Label <indexing.label>`.

* ``.iloc`` is primarily integer position based (from ``0`` to
  ``length-1`` of the axis), but may also be used with a boolean
  array.  ``.iloc`` will raise ``IndexError`` if a requested
  indexer is out-of-bounds, except *slice* indexers which allow
  out-of-bounds indexing.  (this conforms with Python/NumPy *slice*
  semantics).  Allowed inputs are:

    * An integer e.g. ``5``.
    * A list or array of integers ``[4, 3, 0]``.
    * A slice object with ints ``1:7``.
    * A boolean array (any ``NA`` values will be treated as ``False``).
    * A ``callable`` function with one argument (the calling Series or DataFrame) and
      that returns valid output for indexing (one of the above).
    * A tuple of row (and column) indices whose elements are one of the
      above inputs.

  See more at `Selection by Position <indexing.integer>`,
  `Advanced Indexing <advanced>` and Advanced
  Hierarchical .

* ``.loc``, ``.iloc``, and also ``[]`` indexing can accept a ``callable`` as indexer. See more at `Selection By Callable <indexing.callable>[.

  .. note::

     Destructuring tuple keys into row (and column) indexes occurs
     *before* callables are applied, so you cannot return a tuple from
     a callable to index both rows and columns.

Getting values from an object with multi-axes selection uses the following
notation (using ``.loc`` as an example, but the following applies to ``.iloc`` as
well). Any of the axes accessors may be the null slice ``:``. Axes left out of
the specification are assumed to be ``:``, e.g. ``p.loc['a']`` is equivalent to
``p.loc['a', :]``.


```python
ser = pd.Series(range(5), index=list("abcde"))
ser.loc[["a", "c", "e"]]

df = pd.DataFrame(np.arange(25).reshape(5, 5), index=list("abcde"), columns=list("abcde"))
df.loc[["a", "c", "e"], ["b", "d"]]
```


## Basics
As mentioned when introducing the data structures in the last section
, the primary function of indexing with ``[]`` (a.k.a. ``__getitem__``
for those familiar with implementing class behavior in Python) is selecting out
lower-dimensional slices. The following table shows return type values when
indexing pandas objects with ``[]``:


    :header: "Object Type", "Selection", "Return Value Type"
    :widths: 30, 30, 60

    Series, ``series[label]``, scalar value
    DataFrame, ``frame[colname]``, ``Series`` corresponding to colname

Here we construct a simple time series data set to use for illustrating the
indexing functionality:

```python
dates = pd.date_range('1/1/2000', periods=8)
df = pd.DataFrame(np.random.randn(8, 4),
                  index=dates, columns=['A', 'B', 'C', 'D'])
df
```
> **note.capitalize():**
   None of the indexing functionality is time series specific unless
   specifically stated.

Thus, as per above, we have the most basic indexing using ``[]``:

```python
s = df['A']
s[dates[5]]
```
You can pass a list of columns to ``[]`` to select columns in that order.
If a column is not contained in the DataFrame, an exception will be
raised. Multiple columns can also be set in this manner:

```python
df
df[['B', 'A']] = df[['A', 'B']]
df
```
You may find this useful for applying a transform (in-place) to a subset of the
columns.

> **warning.capitalize():**
   pandas aligns all AXES when setting ``Series`` and ``DataFrame`` from ``.loc``.

   This will **not** modify ``df`` because the column alignment is before value assignment.

   ```python
df[['A', 'B']]
   df.loc[:, ['B', 'A']] = df[['A', 'B']]
   df[['A', 'B']]

The correct way to swap column values is by using raw values:



   df.loc[:, ['B', 'A']] = df[['A', 'B']].to_numpy()
   df[['A', 'B']]

However, pandas does not align AXES when setting ``Series`` and ``DataFrame`` from ``.iloc``
because ``.iloc`` operates by position.

This will modify ``df`` because the column alignment is not done before value assignment.



   df[['A', 'B']]
   df.iloc[:, [1, 0]] = df[['A', 'B']]
   df[['A','B']]
```
## Attribute access






You may access an index on a ``Series`` or  column on a ``DataFrame`` directly
as an attribute:

```python
sa = pd.Series([1, 2, 3], index=list('abc'))
dfa = df.copy()
```
```python
sa.b
dfa.A
```
```python
sa.a = 5
sa
dfa.A = list(range(len(dfa.index)))  # ok if A already exists
dfa
dfa['A'] = list(range(len(dfa.index)))  # use this form to create a new column
dfa
```
> **warning.capitalize():**
   - You can use this access only if the index element is a valid Python identifier, e.g. ``s.1`` is not allowed.
     See `here for an explanation of valid identifiers
    ](https://docs.python.org/3/reference/lexical_analysis.html#identifiers)_.

   - The attribute will not be available if it conflicts with an existing method name, e.g. ``s.min`` is not allowed, but ``s['min']`` is possible.

   - Similarly, the attribute will not be available if it conflicts with any of the following list: ``index``,
     ``major_axis``, ``minor_axis``, ``items``.

   - In any of these cases, standard indexing will still work, e.g. ``s['1']``, ``s['min']``, and ``s['index']`` will
     access the corresponding element or column.

If you are using the IPython environment, you may also use tab-completion to
see these accessible attributes.

You can also assign a ``dict`` to a row of a ``DataFrame``:

```python
x = pd.DataFrame({'x': [1, 2, 3], 'y': [3, 4, 5]})
x.iloc[1] = {'x': 9, 'y': 99}
x
```
You can use attribute access to modify an existing element of a Series or column of a DataFrame, but be careful;
if you try to use attribute access to create a new column, it creates a new attribute rather than a
new column and will this raise a ``UserWarning``:

```python
:okwarning:

 df_new = pd.DataFrame({'one': [1., 2., 3.]})
 df_new.two = [4, 5, 6]
 df_new
```
## Slicing ranges
The most robust and consistent way of slicing ranges along arbitrary axes is
described in the `Selection by Position <indexing.integer>` section
detailing the ``.iloc`` method. For now, we explain the semantics of slicing using the ``[]`` operator.

    .. note::

        When the `Series` has float indices, slicing will select by position.

With Series, the syntax works exactly as with an ndarray, returning a slice of
the values and the corresponding labels:

```python
s[:5]
s[::2]
s[::-1]
```
Note that setting works as well:

```python
s2 = s.copy()
s2[:5] = 0
s2
```
With DataFrame, slicing inside of ``[]`` **slices the rows**. This is provided
largely as a convenience since it is such a common operation.

```python
df[:3]
df[::-1]
```


## Selection by label
> **warning.capitalize():**
   ``.loc`` is strict when you present slicers that are not compatible (or convertible) with the index type. For example
   using integers in a ``DatetimeIndex``. These will raise a ``TypeError``.

   ```python
:okexcept:

    dfl = pd.DataFrame(np.random.randn(5, 4),
                       columns=list('ABCD'),
                       index=pd.date_range('20130101', periods=5))
    dfl
    dfl.loc[2:3]

String likes in slicing *can* be convertible to the type of the index and lead to natural slicing.



   dfl.loc['20130102':'20130104']
```
pandas provides a suite of methods in order to have **purely label based indexing**. This is a strict inclusion based protocol.
Every label asked for must be in the index, or a ``KeyError`` will be raised.
When slicing, both the start bound **AND** the stop bound are *included*, if present in the index.
Integers are valid labels, but they refer to the label **and not the position**.

The ``.loc`` attribute is the primary access method. The following are valid inputs:

* A single label, e.g. ``5`` or ``'a'`` (Note that ``5`` is interpreted as a *label* of the index. This use is **not** an integer position along the index.).
* A list or array of labels ``['a', 'b', 'c']``.
* A slice object with labels ``'a':'f'``. Note that contrary to usual Python
  slices, **both** the start and the stop are included, when present in the
  index! See `Slicing with labels <indexing.slicing_with_labels>`.
* A boolean array.
* A ``callable``, see `Selection By Callable <indexing.callable>`.

```python
s1 = pd.Series(np.random.randn(6), index=list('abcdef'))
s1
s1.loc['c':]
s1.loc['b']
```
Note that setting works as well:

```python
s1.loc['c':] = 0
s1
```
With a DataFrame:

```python
df1 = pd.DataFrame(np.random.randn(6, 4),
                   index=list('abcdef'),
                   columns=list('ABCD'))
df1
df1.loc[['a', 'b', 'd'], :]
```
Accessing via label slices:

```python
df1.loc['d':, 'A':'C']
```
For getting a cross section using a label (equivalent to ``df.xs('a')``):

```python
df1.loc['a']
```
For getting values with a boolean array:

```python
df1.loc['a'] > 0
df1.loc[:, df1.loc['a'] > 0]
```
NA values in a boolean array propagate as ``False``:

```python
mask = pd.array([True, False, True, False, pd.NA, False], dtype="boolean")
mask
df1[mask]
```
For getting a value explicitly:

```python
# this is also equivalent to ``df1.at['a','A']``
df1.loc['a', 'A']
```


### Slicing with labels
When using ``.loc`` with slices, if both the start and the stop labels are
present in the index, then elements *located* between the two (including them)
are returned:

```python
s = pd.Series(list('abcde'), index=[0, 3, 2, 5, 4])
s.loc[3:5]
```
If the index is sorted, and can be compared against start and stop labels,
then slicing will still work as expected, by selecting labels which *rank*
between the two:

```python
s.sort_index()
s.sort_index().loc[1:6]
```
However, if at least one of the two is absent *and* the index is not sorted, an
error will be raised (since doing otherwise would be computationally expensive,
as well as potentially ambiguous for mixed type indexes). For instance, in the
above example, ``s.loc[1:6]`` would raise ``KeyError``.

For the rationale behind this behavior, see
`Endpoints are inclusive <advanced.endpoints_are_inclusive>`.

```python
s = pd.Series(list('abcdef'), index=[0, 3, 2, 5, 4, 2])
s.loc[3:5]
```
Also, if the index has duplicate labels *and* either the start or the stop label is duplicated,
an error will be raised. For instance, in the above example, ``s.loc[2:5]`` would raise a ``KeyError``.

For more information about duplicate labels, see
`Duplicate Labels <duplicates>`.



## Selection by position
pandas provides a suite of methods in order to get **purely integer based indexing**. The semantics follow closely Python and NumPy slicing. These are ``0-based`` indexing. When slicing, the start bound is *included*, while the upper bound is *excluded*. Trying to use a non-integer, even a **valid** label will raise an ``IndexError``.

The ``.iloc`` attribute is the primary access method. The following are valid inputs:

* An integer e.g. ``5``.
* A list or array of integers ``[4, 3, 0]``.
* A slice object with ints ``1:7``.
* A boolean array.
* A ``callable``, see `Selection By Callable <indexing.callable>`.
* A tuple of row (and column) indexes, whose elements are one of the
  above types.

```python
s1 = pd.Series(np.random.randn(5), index=list(range(0, 10, 2)))
s1
s1.iloc[:3]
s1.iloc[3]
```
Note that setting works as well:

```python
s1.iloc[:3] = 0
s1
```
With a DataFrame:

```python
df1 = pd.DataFrame(np.random.randn(6, 4),
                   index=list(range(0, 12, 2)),
                   columns=list(range(0, 8, 2)))
df1
```
Select via integer slicing:

```python
df1.iloc[:3]
df1.iloc[1:5, 2:4]
```
Select via integer list:

```python
df1.iloc[[1, 3, 5], [1, 3]]
```
```python
df1.iloc[1:3, :]
```
```python
df1.iloc[:, 1:3]
```
```python
# this is also equivalent to ``df1.iat[1,1]``
df1.iloc[1, 1]
```
For getting a cross section using an integer position (equiv to ``df.xs(1)``):

```python
df1.iloc[1]
```
Out of range slice indexes are handled gracefully just as in Python/NumPy.

```python
# these are allowed in Python/NumPy.
x = list('abcdef')
x
x[4:10]
x[8:10]
s = pd.Series(x)
s
s.iloc[4:10]
s.iloc[8:10]
```
Note that using slices that go out of bounds can result in
an empty axis (e.g. an empty DataFrame being returned).

```python
dfl = pd.DataFrame(np.random.randn(5, 2), columns=list('AB'))
dfl
dfl.iloc[:, 2:3]
dfl.iloc[:, 1:3]
dfl.iloc[4:6]
```
A single indexer that is out of bounds will raise an ``IndexError``.
A list of indexers where any element is out of bounds will raise an
``IndexError``.

```python
:okexcept:

dfl.iloc[[4, 5, 6]]
```
```python
:okexcept:

dfl.iloc[:, 4]
```


## Selection by callable
``.loc``, ``.iloc``, and also ``[]`` indexing can accept a ``callable`` as indexer.
The ``callable`` must be a function with one argument (the calling Series or DataFrame) that returns valid output for indexing.

> **note.capitalize():**
   For ``.iloc`` indexing, returning a tuple from the callable is
   not supported, since tuple destructuring for row and column indexes
   occurs *before* applying callables.

```python
df1 = pd.DataFrame(np.random.randn(6, 4),
                   index=list('abcdef'),
                   columns=list('ABCD'))
df1

df1.loc[lambda df: df['A'] > 0, :]
df1.loc[:, lambda df: ['A', 'B']]

df1.iloc[:, lambda df: [0, 1]]

df1[lambda df: df.columns[0]]
```
You can use callable indexing in ``Series``.

```python
df1['A'].loc[lambda s: s > 0]
```
Using these methods / indexers, you can chain data selection operations
without using a temporary variable.

```python
bb = pd.read_csv('data/baseball.csv', index_col='id')
(bb.groupby(['year', 'team']).sum(numeric_only=True)
   .loc[lambda df: df['r'] > 100])
```


## Combining positional and label-based indexing
If you wish to get the 0th and the 2nd elements from the index in the 'A' column, you can do:

```python
dfd = pd.DataFrame({'A': [1, 2, 3],
                    'B': [4, 5, 6]},
                   index=list('abc'))
dfd
dfd.loc[dfd.index[[0, 2]], 'A']
```
This can also be expressed using ``.iloc``, by explicitly getting locations on the indexers, and using
*positional* indexing to select things.

```python
dfd.iloc[[0, 2], dfd.columns.get_loc('A')]
```
For getting *multiple* indexers, using ``.get_indexer``:

```python
dfd.iloc[[0, 2], dfd.columns.get_indexer(['A', 'B'])]
```
### Reindexing
The idiomatic way to achieve selecting potentially not-found elements is via ``.reindex()``. See also the section on `reindexing <basics.reindexing>`.

```python
s = pd.Series([1, 2, 3])
s.reindex([1, 2, 3])
```
Alternatively, if you want to select only *valid* keys, the following is idiomatic and efficient; it is guaranteed to preserve the dtype of the selection.

```python
labels = [1, 2, 3]
s.loc[s.index.intersection(labels)]
```
Having a duplicated index will raise for a ``.reindex()``:

```python
:okexcept:

s = pd.Series(np.arange(4), index=['a', 'a', 'b', 'c'])
labels = ['c', 'd']
s.reindex(labels)
```
Generally, you can intersect the desired labels with the current
axis, and then reindex.

```python
s.loc[s.index.intersection(labels)].reindex(labels)
```
However, this would *still* raise if your resulting index is duplicated.

```python
:okexcept:

labels = ['a', 'd']
s.loc[s.index.intersection(labels)].reindex(labels)
```


## Selecting random samples
A random selection of rows or columns from a Series or DataFrame with the `~DataFrame.sample` method. The method will sample rows by default, and accepts a specific number of rows/columns to return, or a fraction of rows.

```python
s = pd.Series([0, 1, 2, 3, 4, 5])

# When no arguments are passed, returns 1 row.
s.sample()

# One may specify either a number of rows:
s.sample(n=3)

# Or a fraction of the rows:
s.sample(frac=0.5)
```
By default, ``sample`` will return each row at most once, but one can also sample with replacement
using the ``replace`` option:

```python
s = pd.Series([0, 1, 2, 3, 4, 5])

# Without replacement (default):
s.sample(n=6, replace=False)

# With replacement:
s.sample(n=6, replace=True)
```
By default, each row has an equal probability of being selected, but if you want rows
to have different probabilities, you can pass the ``sample`` function sampling weights as
``weights``. These weights can be a list, a NumPy array, or a Series, but they must be of the same length as the object you are sampling. Missing values will be treated as a weight of zero, and inf values are not allowed. If weights do not sum to 1, they will be re-normalized by dividing all weights by the sum of the weights. For example:

```python
s = pd.Series([0, 1, 2, 3, 4, 5])
example_weights = [0, 0, 0.2, 0.2, 0.2, 0.4]
s.sample(n=2, weights=example_weights)

# Weights will be re-normalized automatically
example_weights2 = [0.5, 0, 0, 0, 0, 0]
s.sample(n=1, weights=example_weights2)
```
When applied to a DataFrame, you can use a column of the DataFrame as sampling weights
(provided you are sampling rows and not columns) by simply passing the name of the column
as a string.

```python
df2 = pd.DataFrame({'col1': [9, 8, 7, 6],
                    'weight_column': [0.5, 0.4, 0.1, 0]})
df2.sample(n=2, weights='weight_column')
```
``sample`` also allows users to sample columns instead of rows using the ``axis`` argument.

```python
df3 = pd.DataFrame({'col1': [1, 2, 3], 'col2': [2, 3, 4]})
df3.sample(n=1, axis=1)
```
Finally, one can also set a seed for ``sample``'s random number generator using the ``random_state`` argument, which will accept either an integer (as a seed) or a NumPy RandomState object.

```python
df4 = pd.DataFrame({'col1': [1, 2, 3], 'col2': [2, 3, 4]})

# With a given seed, the sample will always draw the same rows.
df4.sample(n=2, random_state=2)
df4.sample(n=2, random_state=2)
```
## Setting with enlargement
The ``.loc/[]`` operations can perform enlargement when setting a non-existent key for that axis.

In the ``Series`` case this is effectively an appending operation.

```python
se = pd.Series([1, 2, 3])
se
se[5] = 5.
se
```
A ``DataFrame`` can be enlarged on either axis via ``.loc``.

```python
dfi = pd.DataFrame(np.arange(6).reshape(3, 2),
                   columns=['A', 'B'])
dfi
dfi.loc[:, 'C'] = dfi.loc[:, 'A']
dfi
```
This is like an ``append`` operation on the ``DataFrame``.

```python
dfi.loc[3] = 5
dfi
```


## Fast scalar value getting and setting
Since indexing with ``[]`` must handle a lot of cases (single-label access,
slicing, boolean indexing, etc.), it has a bit of overhead in order to figure
out what you're asking for. If you only want to access a scalar value, the
fastest way is to use the ``at`` and ``iat`` methods, which are implemented on
all of the data structures.

Similarly to ``loc``, ``at`` provides **label** based scalar lookups, while, ``iat`` provides **integer** based lookups analogously to ``iloc``

```python
s.iat[5]
df.at[dates[5], 'A']
df.iat[3, 0]
```
You can also set using these same indexers.

```python
df.at[dates[5], 'E'] = 7
df.iat[3, 0] = 7
```
``at`` may enlarge the object in-place as above if the indexer is missing.

```python
df.at[dates[-1] + pd.Timedelta('1 day'), 0] = 7
df
```
## Boolean indexing


Another common operation is the use of boolean vectors to filter the data.
The operators are: ``|`` for ``or``, ``&`` for ``and``, and ``~`` for ``not``.
These **must** be grouped by using parentheses, since by default Python will
evaluate an expression such as ``df['A'] > 2 & df['B'] < 3`` as
``df['A'] > (2 & df['B']) < 3``, while the desired evaluation order is
``(df['A'] > 2) & (df['B'] < 3)``.

Using a boolean vector to index a Series works exactly as in a NumPy ndarray:

```python
s = pd.Series(range(-3, 4))
s
s[s > 0]
s[(s < -1) | (s > 0.5)]
s[~(s < 0)]
```
You may select rows from a DataFrame using a boolean vector the same length as
the DataFrame's index (for example, something derived from one of the columns
of the DataFrame):

```python
df[df['A'] > 0]
```
List comprehensions and the ``map`` method of Series can also be used to produce
more complex criteria:

```python
df2 = pd.DataFrame({'a': ['one', 'one', 'two', 'three', 'two', 'one', 'six'],
                    'b': ['x', 'y', 'y', 'x', 'y', 'x', 'x'],
                    'c': np.random.randn(7)})

# only want 'two' or 'three'
criterion = df2['a'].map(lambda x: x.startswith('t'))

df2[criterion]

# equivalent but slower
df2[[x.startswith('t') for x in df2['a']]]

# Multiple criteria
df2[criterion & (df2['b'] == 'x')]
```
With the choice methods `Selection by Label <indexing.label>`, `Selection by Position <indexing.integer>`,
and `Advanced Indexing <advanced>` you may select along more than one axis using boolean vectors combined with other indexing expressions.

```python
df2.loc[criterion & (df2['b'] == 'x'), 'b':'c']
```
> **warning.capitalize():**
   While ``loc`` supports two kinds of boolean indexing, ``iloc`` only supports indexing with a
   boolean array. If the indexer is a boolean ``Series``, an error will be raised. For instance,
   in the following example, ``df.iloc[s.values, 1]`` is ok. The boolean indexer is an array.
   But ``df.iloc[s, 1]`` would raise ``ValueError``.

   ```python
df = pd.DataFrame([[1, 2], [3, 4], [5, 6]],
                  index=list('abc'),
                  columns=['A', 'B'])
s = (df['A'] > 2)
s

df.loc[s, 'B']

df.iloc[s.values, 1]
```


## Indexing with isin
Consider the `~Series.isin` method of ``Series``, which returns a boolean
vector that is true wherever the ``Series`` elements exist in the passed list.
This allows you to select rows where one or more columns have values you want:

```python
s = pd.Series(np.arange(5), index=np.arange(5)[::-1], dtype='int64')
s
s.isin([2, 4, 6])
s[s.isin([2, 4, 6])]
```
The same method is available for ``Index`` objects and is useful for the cases
when you don't know which of the sought labels are in fact present:

```python
s[s.index.isin([2, 4, 6])]

# compare it to the following
s.reindex([2, 4, 6])
```
In addition to that, ``MultiIndex`` allows selecting a separate level to use
in the membership check:

```python
s_mi = pd.Series(np.arange(6),
                 index=pd.MultiIndex.from_product([[0, 1], ['a', 'b', 'c']]))
s_mi
s_mi.iloc[s_mi.index.isin([(1, 'a'), (2, 'b'), (0, 'c')])]
s_mi.iloc[s_mi.index.isin(['a', 'c', 'e'], level=1)]
```
DataFrame also has an `~DataFrame.isin` method.  When calling ``isin``, pass a set of
values as either an array or dict.  If values is an array, ``isin`` returns
a DataFrame of booleans that is the same shape as the original DataFrame, with True
wherever the element is in the sequence of values.

```python
df = pd.DataFrame({'vals': [1, 2, 3, 4], 'ids': ['a', 'b', 'f', 'n'],
                   'ids2': ['a', 'n', 'c', 'n']})

values = ['a', 'b', 1, 3]

df.isin(values)
```
Oftentimes you'll want to match certain values with certain columns.
Just make values a ``dict`` where the key is the column, and the value is
a list of items you want to check for.

```python
values = {'ids': ['a', 'b'], 'vals': [1, 3]}

df.isin(values)
```
To return the DataFrame of booleans where the values are *not* in the original DataFrame,
use the ``~`` operator:

```python
values = {'ids': ['a', 'b'], 'vals': [1, 3]}

~df.isin(values)
```
Combine DataFrame's ``isin`` with the ``any()`` and ``all()`` methods to
quickly select subsets of your data that meet a given criteria.
To select a row where each column meets its own criterion:

```python
values = {'ids': ['a', 'b'], 'ids2': ['a', 'c'], 'vals': [1, 3]}

row_mask = df.isin(values).all(axis=1)

df[row_mask]
```


## The `~pandas.DataFrame.where` Method and Masking
Selecting values from a Series with a boolean vector generally returns a
subset of the data. To guarantee that selection output has the same shape as
the original data, you can use the ``where`` method in ``Series`` and ``DataFrame``.

To return only the selected rows:

```python
s[s > 0]
```
To return a Series of the same shape as the original:

```python
s.where(s > 0)
```
Selecting values from a DataFrame with a boolean criterion now also preserves
input data shape. ``where`` is used under the hood as the implementation.
The code below is equivalent to ``df.where(df < 0)``.

```python
dates = pd.date_range('1/1/2000', periods=8)
df = pd.DataFrame(np.random.randn(8, 4),
                  index=dates, columns=['A', 'B', 'C', 'D'])
df[df < 0]
```
In addition, ``where`` takes an optional ``other`` argument for replacement of
values where the condition is False, in the returned copy.

```python
df.where(df < 0, -df)
```
You may wish to set values based on some boolean criteria.
This can be done intuitively like so:

```python
s2 = s.copy()
s2[s2 < 0] = 0
s2

df2 = df.copy()
df2[df2 < 0] = 0
df2
```
``where`` returns a modified copy of the data.

> **note.capitalize():**
   The signature for `DataFrame.where` differs from `numpy.where`.
   Roughly ``df1.where(m, df2)`` is equivalent to ``np.where(m, df1, df2)``.

   ```python
df.where(df < 0, -df) == np.where(df < 0, df, -df)
```
**Alignment**

Furthermore, ``where`` aligns the input boolean condition (ndarray or DataFrame),
such that partial selection with setting is possible. This is analogous to
partial setting via ``.loc`` (but on the contents rather than the axis labels).

```python
df2 = df.copy()
df2[df2[1:4] > 0] = 3
df2
```
Where can also accept ``axis`` and ``level`` parameters to align the input when
performing the ``where``.

```python
df2 = df.copy()
df2.where(df2 > 0, df2['A'], axis='index')
```
This is equivalent to (but faster than) the following.

```python
df2 = df.copy()
df.apply(lambda x, y: x.where(x > 0, y), y=df['A'])
```
``where`` can accept a callable as condition and ``other`` arguments. The function must
be with one argument (the calling Series or DataFrame) and that returns valid output
as condition and ``other`` argument.

```python
df3 = pd.DataFrame({'A': [1, 2, 3],
                    'B': [4, 5, 6],
                    'C': [7, 8, 9]})
df3.where(lambda x: x > 4, lambda x: x + 10)
```
### Mask
`~pandas.DataFrame.mask` is the inverse boolean operation of ``where``.

```python
s.mask(s >= 0)
df.mask(df >= 0)
```


## Setting with enlargement conditionally using `numpy`
An alternative to `~pandas.DataFrame.where` is to use `numpy.where`.
Combined with setting a new column, you can use it to enlarge a DataFrame where the
values are determined conditionally.

Consider you have two choices to choose from in the following DataFrame. And you want to
set a new column color to 'green' when the second column has 'Z'.  You can do the
following:

```python
df = pd.DataFrame({'col1': list('ABBC'), 'col2': list('ZZXY')})
df['color'] = np.where(df['col2'] == 'Z', 'green', 'red')
df
```
If you have multiple conditions, you can use `numpy.select` to achieve that.  Say
corresponding to three conditions there are three choice of colors, with a fourth color
as a fallback, you can do the following.

```python
conditions = [
    (df['col2'] == 'Z') & (df['col1'] == 'A'),
    (df['col2'] == 'Z') & (df['col1'] == 'B'),
    (df['col1'] == 'B')
]
choices = ['yellow', 'blue', 'purple']
df['color'] = np.select(conditions, choices, default='black')
df
```


## The `~pandas.DataFrame.query` Method
`~pandas.DataFrame` objects have a `~pandas.DataFrame.query`
method that allows selection using an expression.

You can get the value of the frame where column ``b`` has values
between the values of columns ``a`` and ``c``. For example:

```python
n = 10
df = pd.DataFrame(np.random.rand(n, 3), columns=list('abc'))
df

# pure python
df[(df['a'] < df['b']) & (df['b'] < df['c'])]

# query
df.query('(a < b) & (b < c)')
```
Do the same thing but fall back on a named index if there is no column
with the name ``a``.

```python
df = pd.DataFrame(np.random.randint(n / 2, size=(n, 2)), columns=list('bc'))
df.index.name = 'a'
df
df.query('a < b and b < c')
```
If instead you don't want to or cannot name your index, you can use the name
``index`` in your query expression:

```python
df = pd.DataFrame(np.random.randint(n, size=(n, 2)), columns=list('bc'))
df
df.query('index < b < c')
```
> **note.capitalize():**
   If the name of your index overlaps with a column name, the column name is
   given precedence. For example,

   ```python
df = pd.DataFrame({'a': np.random.randint(5, size=5)})
   df.index.name = 'a'
   df.query('a > 2')  # uses the column 'a', not the index

You can still use the index in a query expression by using the special
identifier 'index':



   df.query('index > 2')

If for some reason you have a column named ``index``, then you can refer to
the index as ``ilevel_0`` as well, but at this point you should consider
renaming your columns to something less ambiguous.
```
### `~pandas.MultiIndex` `~pandas.DataFrame.query` Syntax
You can also use the levels of a ``DataFrame`` with a
`~pandas.MultiIndex` as if they were columns in the frame:

```python
n = 10
colors = np.random.choice(['red', 'green'], size=n)
foods = np.random.choice(['eggs', 'ham'], size=n)
colors
foods

index = pd.MultiIndex.from_arrays([colors, foods], names=['color', 'food'])
df = pd.DataFrame(np.random.randn(n, 2), index=index)
df
df.query('color == "red"')
```
If the levels of the ``MultiIndex`` are unnamed, you can refer to them using
special names:

```python
df.index.names = [None, None]
df
df.query('ilevel_0 == "red"')
```
The convention is ``ilevel_0``, which means "index level 0" for the 0th level
of the ``index``.


### `~pandas.DataFrame.query` Use Cases
A use case for `~pandas.DataFrame.query` is when you have a collection of
`~pandas.DataFrame` objects that have a subset of column names (or index
levels/names) in common. You can pass the same query to both frames *without*
having to specify which frame you're interested in querying

```python
df = pd.DataFrame(np.random.rand(n, 3), columns=list('abc'))
df
df2 = pd.DataFrame(np.random.rand(n + 2, 3), columns=df.columns)
df2
expr = '0.0 <= a <= c <= 0.5'
map(lambda frame: frame.query(expr), [df, df2])
```
### `~pandas.DataFrame.query` Python versus pandas Syntax Comparison
Full numpy-like syntax:

```python
df = pd.DataFrame(np.random.randint(n, size=(n, 3)), columns=list('abc'))
df
df.query('(a < b) & (b < c)')
df[(df['a'] < df['b']) & (df['b'] < df['c'])]
```
Slightly nicer by removing the parentheses (comparison operators bind tighter
than ``&`` and ``|``):

```python
df.query('a < b & b < c')
```
Use English instead of symbols:

```python
df.query('a < b and b < c')
```
Pretty close to how you might write it on paper:

```python
df.query('a < b < c')
```
### The ``in`` and ``not in`` operators
`~pandas.DataFrame.query` also supports special use of Python's ``in`` and
``not in`` comparison operators, providing a succinct syntax for calling the
``isin`` method of a ``Series`` or ``DataFrame``.

```python
# get all rows where columns "a" and "b" have overlapping values
df = pd.DataFrame({'a': list('aabbccddeeff'), 'b': list('aaaabbbbcccc'),
                   'c': np.random.randint(5, size=12),
                   'd': np.random.randint(9, size=12)})
df
df.query('a in b')

# How you'd do it in pure Python
df[df['a'].isin(df['b'])]

df.query('a not in b')

# pure Python
df[~df['a'].isin(df['b'])]
```
You can combine this with other expressions for very succinct queries:


```python
# rows where cols a and b have overlapping values
# and col c's values are less than col d's
df.query('a in b and c < d')

# pure Python
df[df['b'].isin(df['a']) & (df['c'] < df['d'])]
```
> **note.capitalize():**
   Note that ``in`` and ``not in`` are evaluated in Python, since ``numexpr``
   has no equivalent of this operation. However, **only the** ``in``/``not in``
   **expression itself** is evaluated in vanilla Python. For example, in the
   expression

   ```python
df.query('a in b + c + d')

``(b + c + d)`` is evaluated by ``numexpr`` and *then* the ``in``
operation is evaluated in plain Python. In general, any operations that can
be evaluated using ``numexpr`` will be.
```
### Special use of the ``==`` operator with ``list`` objects
Comparing a ``list`` of values to a column using ``==``/``!=`` works similarly
to ``in``/``not in``.

```python
df.query('b == ["a", "b", "c"]')

# pure Python
df[df['b'].isin(["a", "b", "c"])]

df.query('c == [1, 2]')

df.query('c != [1, 2]')

# using in/not in
df.query('[1, 2] in c')

df.query('[1, 2] not in c')

# pure Python
df[df['c'].isin([1, 2])]
```
### Boolean operators
You can negate boolean expressions with the word ``not`` or the ``~`` operator.

```python
df = pd.DataFrame(np.random.rand(n, 3), columns=list('abc'))
df['bools'] = np.random.rand(len(df)) > 0.5
df.query('~bools')
df.query('not bools')
df.query('not bools') == df[~df['bools']]
```
Of course, expressions can be arbitrarily complex too:

```python
# short query syntax
shorter = df.query('a < b < c and (not bools) or bools > 2')

# equivalent in pure Python
longer = df[(df['a'] < df['b'])
            & (df['b'] < df['c'])
            & (~df['bools'])
            | (df['bools'] > 2)]

shorter
longer

shorter == longer
```
### Performance of `~pandas.DataFrame.query`
``DataFrame.query()`` using ``numexpr`` is slightly faster than Python for
large frames.

..
    The eval-perf.png figure below was generated with /doc/scripts/eval_performance.py





You will only see the performance benefits of using the ``numexpr`` engine
with ``DataFrame.query()`` if your frame has more than approximately 100,000
rows.



This plot was created using a ``DataFrame`` with 3 columns each containing
floating point values generated using ``numpy.random.randn()``.

```python
df = pd.DataFrame(np.random.randn(8, 4),
                  index=dates, columns=['A', 'B', 'C', 'D'])
df2 = df.copy()
```
## Duplicate data


If you want to identify and remove duplicate rows in a DataFrame,  there are
two methods that will help: ``duplicated`` and ``drop_duplicates``. Each
takes as an argument the columns to use to identify duplicated rows.

* ``duplicated`` returns a boolean vector whose length is the number of rows, and which indicates whether a row is duplicated.
* ``drop_duplicates`` removes duplicate rows.

By default, the first observed row of a duplicate set is considered unique, but
each method has a ``keep`` parameter to specify targets to be kept.

* ``keep='first'`` (default): mark / drop duplicates except for the first occurrence.
* ``keep='last'``: mark / drop duplicates except for the last occurrence.
* ``keep=False``: mark  / drop all duplicates.

```python
df2 = pd.DataFrame({'a': ['one', 'one', 'two', 'two', 'two', 'three', 'four'],
                    'b': ['x', 'y', 'x', 'y', 'x', 'x', 'x'],
                    'c': np.random.randn(7)})
df2
df2.duplicated('a')
df2.duplicated('a', keep='last')
df2.duplicated('a', keep=False)
df2.drop_duplicates('a')
df2.drop_duplicates('a', keep='last')
df2.drop_duplicates('a', keep=False)
```
Also, you can pass a list of columns to identify duplications.

```python
df2.duplicated(['a', 'b'])
df2.drop_duplicates(['a', 'b'])
```
To drop duplicates by index value, use ``Index.duplicated`` then perform slicing.
The same set of options are available for the ``keep`` parameter.

```python
df3 = pd.DataFrame({'a': np.arange(6),
                    'b': np.random.randn(6)},
                   index=['a', 'a', 'b', 'c', 'b', 'a'])
df3
df3.index.duplicated()
df3[~df3.index.duplicated()]
df3[~df3.index.duplicated(keep='last')]
df3[~df3.index.duplicated(keep=False)]
```


## Dictionary-like `~pandas.DataFrame.get` method
Each of Series or DataFrame have a ``get`` method which can return a
default value.

```python
s = pd.Series([1, 2, 3], index=['a', 'b', 'c'])
s.get('a')  # equivalent to s['a']
s.get('x', default=-1)
```


## Looking up values by index/column labels
Sometimes you want to extract a set of values given a sequence of row labels
and column labels, this can be achieved by ``pandas.factorize``  and NumPy indexing.

For heterogeneous column types, we subset columns to avoid unnecessary NumPy conversions:

```python
def pd_lookup_het(df, row_labels, col_labels):
   rows = df.index.get_indexer(row_labels)
   cols = df.columns.get_indexer(col_labels)
   sub = df.take(np.unique(cols), axis=1)
   sub = sub.take(np.unique(rows), axis=0)
   rows = sub.index.get_indexer(row_labels)
   values = sub.melt()["value"]
   cols = sub.columns.get_indexer(col_labels)
   flat_index = rows + cols * len(sub)
   result = values[flat_index]
   return result
```
For homogeneous column types, it is fastest to skip column subsetting and go directly to NumPy:

```python
def pd_lookup_hom(df, row_labels, col_labels):
    rows = df.index.get_indexer(row_labels)
    df = df.loc[:, sorted(set(col_labels))]
    cols = df.columns.get_indexer(col_labels)
    result = df.to_numpy()[rows, cols]
    return result
```
Formerly this could be achieved with the dedicated ``DataFrame.lookup`` method
which was deprecated in version 1.2.0 and removed in version 2.0.0.



## Index objects
The pandas `~pandas.Index` class and its subclasses can be viewed as
implementing an *ordered multiset*. Duplicates are allowed.

`~pandas.Index` also provides the infrastructure necessary for
lookups, data alignment, and reindexing. The easiest way to create an
`~pandas.Index` directly is to pass a ``list`` or other sequence to
`~pandas.Index`:

```python
index = pd.Index(['e', 'd', 'a', 'b'])
index
'd' in index
```
or using numbers:

```python
index = pd.Index([1, 5, 12])
index
5 in index
```
If no dtype is given, ``Index`` tries to infer the dtype from the data.
It is also possible to give an explicit dtype when instantiating an `Index`:

```python
index = pd.Index(['e', 'd', 'a', 'b'], dtype="string")
index
index = pd.Index([1, 5, 12], dtype="int8")
index
index = pd.Index([1, 5, 12], dtype="float32")
index
```
You can also pass a ``name`` to be stored in the index:

```python
index = pd.Index(['e', 'd', 'a', 'b'], name='something')
index.name
```
The name, if set, will be shown in the console display:

```python
index = pd.Index(list(range(5)), name='rows')
columns = pd.Index(['A', 'B', 'C'], name='cols')
df = pd.DataFrame(np.random.randn(5, 3), index=index, columns=columns)
df
df['A']
```


### Setting metadata
Indexes are "mostly immutable", but it is possible to set and change their
``name`` attribute. You can use the ``rename``, ``set_names`` to set these attributes
directly, and they default to returning a copy.

See `Advanced Indexing <advanced>` for usage of MultiIndexes.

```python
ind = pd.Index([1, 2, 3])
ind.rename("apple")
ind
ind = ind.set_names(["apple"])
ind.name = "bob"
ind
```
``set_names``, ``set_levels``, and ``set_codes`` also take an optional
``level`` argument

```python
index = pd.MultiIndex.from_product([range(3), ['one', 'two']], names=['first', 'second'])
index
index.levels[1]
index.set_levels(["a", "b"], level=1)
```


### Set operations on Index objects
The two main operations are ``union`` and ``intersection``.
Difference is provided via the ``.difference()`` method.

```python
a = pd.Index(['c', 'b', 'a'])
b = pd.Index(['c', 'e', 'd'])
a.difference(b)
```
Also available is the ``symmetric_difference`` operation, which returns elements
that appear in either ``idx1`` or ``idx2``, but not in both. This is
equivalent to the Index created by ``idx1.difference(idx2).union(idx2.difference(idx1))``,
with duplicates dropped.

```python
idx1 = pd.Index([1, 2, 3, 4])
idx2 = pd.Index([2, 3, 4, 5])
idx1.symmetric_difference(idx2)
```
> **note.capitalize():**
   The resulting index from a set operation will be sorted in ascending order.

When performing `Index.union` between indexes with different dtypes, the indexes
must be cast to a common dtype. Typically, though not always, this is object dtype. The
exception is when performing a union between integer and float data. In this case, the
integer values are converted to float

```python
idx1 = pd.Index([0, 1, 2])
idx2 = pd.Index([0.5, 1.5])
idx1.union(idx2)
```


### Missing values


   Even though ``Index`` can hold missing values (``NaN``), it should be avoided
   if you do not want any unexpected results. For example, some operations
   exclude missing values implicitly.

``Index.fillna`` fills missing values with specified scalar value.

```python
idx1 = pd.Index([1, np.nan, 3, 4])
idx1
idx1.fillna(2)

idx2 = pd.DatetimeIndex([pd.Timestamp('2011-01-01'),
                         pd.NaT,
                         pd.Timestamp('2011-01-03')])
idx2
idx2.fillna(pd.Timestamp('2011-01-02'))
```
## Set / reset index
Occasionally you will load or create a data set into a DataFrame and want to
add an index after you've already done so. There are a couple of different
ways.



### Set an index
DataFrame has a `~DataFrame.set_index` method which takes a column name
(for a regular ``Index``) or a list of column names (for a ``MultiIndex``).
To create a new, re-indexed DataFrame:

```python
data = pd.DataFrame({'a': ['bar', 'bar', 'foo', 'foo'],
                     'b': ['one', 'two', 'one', 'two'],
                     'c': ['z', 'y', 'x', 'w'],
                     'd': [1., 2., 3, 4]})
data
indexed1 = data.set_index('c')
indexed1
indexed2 = data.set_index(['a', 'b'])
indexed2
```
The ``append`` keyword option allow you to keep the existing index and append
the given columns to a MultiIndex:

```python
frame = data.set_index('c', drop=False)
frame = frame.set_index(['a', 'b'], append=True)
frame
```
Other options in ``set_index`` allow you not drop the index columns.

```python
data.set_index('c', drop=False)
```
### Reset the index
As a convenience, there is a new function on DataFrame called
`~DataFrame.reset_index` which transfers the index values into the
DataFrame's columns and sets a simple integer index.
This is the inverse operation of `~DataFrame.set_index`.


```python
data
data.reset_index()
```
The output is more similar to a SQL table or a record array. The names for the
columns derived from the index are the ones stored in the ``names`` attribute.

You can use the ``level`` keyword to remove only a portion of the index:

```python
frame
frame.reset_index(level=1)
```
``reset_index`` takes an optional parameter ``drop`` which if true simply
discards the index, instead of putting index values in the DataFrame's columns.

### Adding an ad hoc index
You can assign a custom index to the ``index`` attribute:

```python
df_idx = pd.DataFrame(range(4))
df_idx.index = pd.Index([10, 20, 30, 40], name="a")
df_idx
```
### Why does assignment fail when using chained indexing?
`Copy-on-Write <copy_on_write>` is the new default with pandas 3.0.
This means that chained indexing will never work.
See `this section <copy_on_write_chained_assignment>`
for more context.



## Series Assignment and Index Alignment
When assigning a Series to a DataFrame column, pandas performs automatic alignment
based on index labels. This is a fundamental behavior that can be surprising to
new users who might expect positional assignment.

### Key Points:
* Series values are matched to DataFrame rows by index label
* Position/order in the Series doesn't matter
* Missing index labels result in NaN values
* This behavior is consistent across df[col] = series and df.loc[:, col] = series

Examples:
```python
import pandas as pd

# Create a DataFrame
df = pd.DataFrame({'values': [1, 2, 3]}, index=['x', 'y', 'z'])

# Series with matching indices (different order)
s1 = pd.Series([10, 20, 30], index=['z', 'x', 'y'])
df['aligned'] = s1  # Aligns by index, not position
print(df)

# Series with partial index match
s2 = pd.Series([100, 200], index=['x', 'z'])
df['partial'] = s2  # Missing 'y' gets NaN
print(df)

# Series with non-matching indices
s3 = pd.Series([1000, 2000], index=['a', 'b'])
df['nomatch'] = s3  # All values become NaN
print(df)


#Avoiding Confusion:
#If you want positional assignment instead of index alignment:
# reset the Series index to match DataFrame index
df['s1_values'] = s1.reindex(df.index)
```

---

# MultiIndex / advanced indexing
This section covers `indexing with a MultiIndex <advanced.hierarchical>`
and `other advanced indexing features <advanced.index_types>`.

See the `Indexing and Selecting Data <indexing>` for general indexing documentation.

See the `cookbook<cookbook.selection>` for some advanced strategies.



## Hierarchical indexing (MultiIndex)
Hierarchical / Multi-level indexing is very exciting as it opens the door to some
quite sophisticated data analysis and manipulation, especially for working with
higher dimensional data. In essence, it enables you to store and manipulate
data with an arbitrary number of dimensions in lower dimensional data
structures like ``Series`` (1d) and ``DataFrame`` (2d).

In this section, we will show what exactly we mean by "hierarchical" indexing
and how it integrates with all of the pandas indexing functionality
described above and in prior sections. Later, when discussing group by
 and `pivoting and reshaping data <reshaping>`, we'll show
non-trivial applications to illustrate how it aids in structuring data for
analysis.

See the `cookbook<cookbook.multi_index>` for some advanced strategies.

### Creating a MultiIndex (hierarchical index) object
The `MultiIndex` object is the hierarchical analogue of the standard
`Index` object which typically stores the axis labels in pandas objects. You
can think of ``MultiIndex`` as an array of tuples where each tuple is unique. A
``MultiIndex`` can be created from a list of arrays (using
`MultiIndex.from_arrays`), an array of tuples (using
`MultiIndex.from_tuples`), a crossed set of iterables (using
`MultiIndex.from_product`), or a `DataFrame` (using
`MultiIndex.from_frame`).  The ``Index`` constructor will attempt to return
a ``MultiIndex`` when it is passed a list of tuples.  The following examples
demonstrate different ways to initialize MultiIndexes.


```python
arrays = [
    ["bar", "bar", "baz", "baz", "foo", "foo", "qux", "qux"],
    ["one", "two", "one", "two", "one", "two", "one", "two"],
]
tuples = list(zip(*arrays))
tuples

index = pd.MultiIndex.from_tuples(tuples, names=["first", "second"])
index

s = pd.Series(np.random.randn(8), index=index)
s
```
When you want every pairing of the elements in two iterables, it can be easier
to use the `MultiIndex.from_product` method:

```python
iterables = [["bar", "baz", "foo", "qux"], ["one", "two"]]
pd.MultiIndex.from_product(iterables, names=["first", "second"])
```
You can also construct a ``MultiIndex`` from a ``DataFrame`` directly, using
the method `MultiIndex.from_frame`. This is a complementary method to
`MultiIndex.to_frame`.

```python
df = pd.DataFrame(
    [["bar", "one"], ["bar", "two"], ["foo", "one"], ["foo", "two"]],
    columns=["first", "second"],
)
pd.MultiIndex.from_frame(df)
```
As a convenience, you can pass a list of arrays directly into ``Series`` or
``DataFrame`` to construct a ``MultiIndex`` automatically:

```python
arrays = [
    np.array(["bar", "bar", "baz", "baz", "foo", "foo", "qux", "qux"]),
    np.array(["one", "two", "one", "two", "one", "two", "one", "two"]),
]
s = pd.Series(np.random.randn(8), index=arrays)
s
df = pd.DataFrame(np.random.randn(8, 4), index=arrays)
df
```
All of the ``MultiIndex`` constructors accept a ``names`` argument which stores
string names for the levels themselves. If no names are provided, ``None`` will
be assigned:

```python
df.index.names
```
This index can back any axis of a pandas object, and the number of **levels**
of the index is up to you:

```python
df = pd.DataFrame(np.random.randn(3, 8), index=["A", "B", "C"], columns=index)
df
pd.DataFrame(np.random.randn(6, 6), index=index[:6], columns=index[:6])
```
We've "sparsified" the higher levels of the indexes to make the console output a
bit easier on the eyes. Note that how the index is displayed can be controlled using the
``multi_sparse`` option in ``pandas.set_options()``:

```python
with pd.option_context("display.multi_sparse", False):
    df
```
It's worth keeping in mind that there's nothing preventing you from using
tuples as atomic labels on an axis:

```python
pd.Series(np.random.randn(8), index=tuples)
```
The reason that the ``MultiIndex`` matters is that it can allow you to do
grouping, selection, and reshaping operations as we will describe below and in
subsequent areas of the documentation. As you will see in later sections, you
can find yourself working with hierarchically-indexed data without creating a
``MultiIndex`` explicitly yourself. However, when loading data from a file, you
may wish to generate your own ``MultiIndex`` when preparing the data set.



### Reconstructing the level labels
The method `~MultiIndex.get_level_values` will return a vector of the labels for each
location at a particular level:

```python
index.get_level_values(0)
index.get_level_values("second")
```
### Basic indexing on axis with MultiIndex
One of the important features of hierarchical indexing is that you can select
data by a "partial" label identifying a subgroup in the data. **Partial**
selection "drops" levels of the hierarchical index in the result in a
completely analogous way to selecting a column in a regular DataFrame:

```python
df["bar"]
df["bar", "one"]
df["bar"]["one"]
s["qux"]
```
See `Cross-section with hierarchical index <advanced.xs>` for how to select
on a deeper level.



### Defined levels
The `MultiIndex` keeps all the defined levels of an index, even
if they are not actually used. When slicing an index, you may notice this.
For example:

```python
df.columns.levels  # original MultiIndex

  df[["foo","qux"]].columns.levels  # sliced
```
This is done to avoid a recomputation of the levels in order to make slicing
highly performant. If you want to see only the used levels, you can use the
`~MultiIndex.get_level_values` method.

```python
df[["foo", "qux"]].columns.to_numpy()

# for a specific level
df[["foo", "qux"]].columns.get_level_values(0)
```
To reconstruct the ``MultiIndex`` with only the used levels, the
`~MultiIndex.remove_unused_levels` method may be used.

```python
new_mi = df[["foo", "qux"]].columns.remove_unused_levels()
new_mi.levels
```
### Data alignment and using ``reindex``
Operations between differently-indexed objects having ``MultiIndex`` on the
axes will work as you expect; data alignment will work the same as an Index of
tuples:

```python
s + s[:-2]
s + s[::2]
```
The `~DataFrame.reindex` method of ``Series``/``DataFrames`` can be
called with another ``MultiIndex``, or even a list or array of tuples:

```python
s.reindex(index[:3])
s.reindex([("foo", "two"), ("bar", "one"), ("qux", "one"), ("baz", "one")])
```


## Advanced indexing with hierarchical index
Syntactically integrating ``MultiIndex`` in advanced indexing with ``.loc`` is a
bit challenging, but we've made every effort to do so. In general, MultiIndex
keys take the form of tuples. For example, the following works as you would expect:

```python
df = df.T
df
df.loc[("bar", "two")]
```
Note that ``df.loc['bar', 'two']`` would also work in this example, but this shorthand
notation can lead to ambiguity in general.

If you also want to index a specific column with ``.loc``, you must use a tuple
like this:

```python
df.loc[("bar", "two"), "A"]
```
You don't have to specify all levels of the ``MultiIndex`` by passing only the
first elements of the tuple. For example, you can use "partial" indexing to
get all elements with ``bar`` in the first level as follows:

```python
df.loc["bar"]
```
This is a shortcut for the slightly more verbose notation ``df.loc[('bar',),]`` (equivalent
to ``df.loc['bar',]`` in this example).

"Partial" slicing also works quite nicely.

```python
df.loc["baz":"foo"]
```
You can slice with a 'range' of values, by providing a slice of tuples.

```python
df.loc[("baz", "two"):("qux", "one")]
df.loc[("baz", "two"):"foo"]
```
Passing a list of labels or tuples works similar to reindexing:

```python
df.loc[[("bar", "two"), ("qux", "one")]]
```
> **note.capitalize():**
   It is important to note that tuples and lists are not treated identically
   in pandas when it comes to indexing. Whereas a tuple is interpreted as one
   multi-level key, a list is used to specify several keys. Or in other words,
   tuples go horizontally (traversing levels), lists go vertically (scanning levels).

Importantly, a list of tuples indexes several complete ``MultiIndex`` keys,
whereas a tuple of lists refer to several values within a level:

```python
s = pd.Series(
    [1, 2, 3, 4, 5, 6],
    index=pd.MultiIndex.from_product([["A", "B"], ["c", "d", "e"]]),
)
s.loc[[("A", "c"), ("B", "d")]]  # list of tuples
s.loc[(["A", "B"], ["c", "d"])]  # tuple of lists
```


### Using slicers
You can slice a ``MultiIndex`` by providing multiple indexers.

You can provide any of the selectors as if you are indexing by label, see `Selection by Label <indexing.label>`,
including slices, lists of labels, labels, and boolean indexers.

You can use ``slice(None)`` to select all the contents of *that* level. You do not need to specify all the
*deeper* levels, they will be implied as ``slice(None)``.

As usual, **both sides** of the slicers are included as this is label indexing.

> **warning.capitalize():**
   You should specify all axes in the ``.loc`` specifier, meaning the indexer for the **index** and
   for the **columns**. There are some ambiguous cases where the passed indexer could be misinterpreted
   as indexing *both* axes, rather than into say the ``MultiIndex`` for the rows.

   You should do this:

   ```python
df.loc[(slice("A1", "A3"), ...), :]  # noqa: E999

   You should **not** do this:
 
   .. code-block:: python

      df.loc[(slice("A1", "A3"), ...)]  # noqa: E999
```
```python
def mklbl(prefix, n):
    return ["%s%s" % (prefix, i) for i in range(n)]


miindex = pd.MultiIndex.from_product(
    [mklbl("A", 4), mklbl("B", 2), mklbl("C", 4), mklbl("D", 2)]
)
micolumns = pd.MultiIndex.from_tuples(
    [("a", "foo"), ("a", "bar"), ("b", "foo"), ("b", "bah")], names=["lvl0", "lvl1"]
)
dfmi = (
    pd.DataFrame(
        np.arange(len(miindex) * len(micolumns)).reshape(
            (len(miindex), len(micolumns))
        ),
        index=miindex,
        columns=micolumns,
    )
    .sort_index()
    .sort_index(axis=1)
)
dfmi
```
Basic MultiIndex slicing using slices, lists, and labels.

```python
dfmi.loc[(slice("A1", "A3"), slice(None), ["C1", "C3"]), :]
```
You can use `pandas.IndexSlice` to facilitate a more natural syntax
using ``:``, rather than using ``slice(None)``.

```python
idx = pd.IndexSlice
dfmi.loc[idx[:, :, ["C1", "C3"]], idx[:, "foo"]]
```
It is possible to perform quite complicated selections using this method on multiple
axes at the same time.

```python
dfmi.loc["A1", (slice(None), "foo")]
dfmi.loc[idx[:, :, ["C1", "C3"]], idx[:, "foo"]]
```
Using a boolean indexer you can provide selection related to the *values*.

```python
mask = dfmi[("a", "foo")] > 200
dfmi.loc[idx[mask, :, ["C1", "C3"]], idx[:, "foo"]]
```
You can also specify the ``axis`` argument to ``.loc`` to interpret the passed
slicers on a single axis.

```python
dfmi.loc(axis=0)[:, :, ["C1", "C3"]]
```
Furthermore, you can *set* the values using the following methods.

```python
:okwarning:

df2 = dfmi.copy()
df2.loc(axis=0)[:, :, ["C1", "C3"]] = -10
df2
```
You can use a right-hand-side of an alignable object as well.

```python
df2 = dfmi.copy()
df2.loc[idx[:, :, ["C1", "C3"]], :] = df2 * 1000
df2
```


### Cross-section
The `~DataFrame.xs` method of ``DataFrame`` additionally takes a level argument to make
selecting data at a particular level of a ``MultiIndex`` easier.

```python
df
df.xs("one", level="second")
```
```python
# using the slicers
df.loc[(slice(None), "one"), :]
```
You can also select on the columns with ``xs``, by
providing the axis argument.

```python
df = df.T
df.xs("one", level="second", axis=1)
```
```python
# using the slicers
df.loc[:, (slice(None), "one")]
```
``xs`` also allows selection with multiple keys.

```python
df.xs(("one", "bar"), level=("second", "first"), axis=1)
```
```python
# using the slicers
df.loc[:, ("bar", "one")]
```
You can pass ``drop_level=False`` to ``xs`` to retain
the level that was selected.

```python
df.xs("one", level="second", axis=1, drop_level=False)
```
Compare the above with the result using ``drop_level=True`` (the default value).

```python
df.xs("one", level="second", axis=1, drop_level=True)
```


### Advanced reindexing and alignment
Using the parameter ``level`` in the `~DataFrame.reindex` and
`~DataFrame.align` methods of pandas objects is useful to broadcast
values across a level. For instance:

```python
midx = pd.MultiIndex(
    levels=[["zero", "one"], ["x", "y"]], codes=[[1, 1, 0, 0], [1, 0, 1, 0]]
)
df = pd.DataFrame(np.random.randn(4, 2), index=midx)
df
df2 = df.groupby(level=0).mean()
df2
df2.reindex(df.index, level=0)

# aligning
df_aligned, df2_aligned = df.align(df2, level=0)
df_aligned
df2_aligned
```
### Swapping levels with ``swaplevel``
The `~MultiIndex.swaplevel` method can switch the order of two levels:

```python
df[:5]
df[:5].swaplevel(0, 1, axis=0)
```


### Reordering levels with ``reorder_levels``
The `~MultiIndex.reorder_levels` method generalizes the ``swaplevel``
method, allowing you to permute the hierarchical index levels in one step:

```python
df[:5].reorder_levels([1, 0], axis=0)
```


### Renaming names of an ``Index`` or ``MultiIndex``
The `~DataFrame.rename` method is used to rename the labels of a
``MultiIndex``, and is typically used to rename the columns of a ``DataFrame``.
The ``columns`` argument of ``rename`` allows a dictionary to be specified
that includes only the columns you wish to rename.

```python
df.rename(columns={0: "col0", 1: "col1"})
```
This method can also be used to rename specific labels of the main index
of the ``DataFrame``.

```python
df.rename(index={"one": "two", "y": "z"})
```
The `~DataFrame.rename_axis` method is used to rename the name of a
``Index`` or ``MultiIndex``. In particular, the names of the levels of a
``MultiIndex`` can be specified, which is useful if ``reset_index()`` is later
used to move the values from the ``MultiIndex`` to a column.

```python
df.rename_axis(index=["abc", "def"])
```
Note that the columns of a ``DataFrame`` are an index, so that using
``rename_axis`` with the ``columns`` argument will change the name of that
index.

```python
df.rename_axis(columns="Cols").columns
```
Both ``rename`` and ``rename_axis`` support specifying a dictionary,
``Series`` or a mapping function to map labels/names to new values.

When working with an ``Index`` object directly, rather than via a ``DataFrame``,
`Index.set_names` can be used to change the names.

```python
mi = pd.MultiIndex.from_product([[1, 2], ["a", "b"]], names=["x", "y"])
mi.names

mi2 = mi.rename("new name", level=0)
mi2
```
You cannot set the names of the MultiIndex via a level.

```python
:okexcept:

mi.levels[0].name = "name via level"
```
Use `Index.set_names` instead.

## Sorting a ``MultiIndex``
For `MultiIndex`-ed objects to be indexed and sliced effectively,
they need to be sorted. As with any index, you can use `~DataFrame.sort_index`.

```python
import random

random.shuffle(tuples)
s = pd.Series(np.random.randn(8), index=pd.MultiIndex.from_tuples(tuples))
s
s.sort_index()
s.sort_index(level=0)
s.sort_index(level=1)
```


You may also pass a level name to ``sort_index`` if the ``MultiIndex`` levels
are named.

```python
s.index = s.index.set_names(["L1", "L2"])
s.sort_index(level="L1")
s.sort_index(level="L2")
```
On higher dimensional objects, you can sort any of the other axes by level if
they have a ``MultiIndex``:

```python
df.T.sort_index(level=1, axis=1)
```
Indexing will work even if the data are not sorted, but will be rather
inefficient (and show a ``PerformanceWarning``). It will also
return a copy of the data rather than a view:

```python
:okwarning:

dfm = pd.DataFrame(
    {"jim": [0, 0, 1, 1], "joe": ["x", "x", "z", "y"], "jolie": np.random.rand(4)}
)
dfm = dfm.set_index(["jim", "joe"])
dfm
dfm.loc[(1, 'z')]
```


Furthermore, if you try to index something that is not fully lexsorted, this can raise:

```python
:okexcept:

dfm.loc[(0, 'y'):(1, 'z')]
```
The `~MultiIndex.is_monotonic_increasing` method on a ``MultiIndex`` shows if the
index is sorted:

```python
dfm.index.is_monotonic_increasing
```
```python
dfm = dfm.sort_index()
dfm
dfm.index.is_monotonic_increasing
```
And now selection works as expected.

```python
dfm.loc[(0, "y"):(1, "z")]
```
## Take methods


Similar to NumPy ndarrays, pandas ``Index``, ``Series``, and ``DataFrame`` also provides
the `~DataFrame.take` method that retrieves elements along a given axis at the given
indices. The given indices must be either a list or an ndarray of integer
index positions. ``take`` will also accept negative integers as relative positions to the end of the object.

```python
index = pd.Index(np.random.randint(0, 1000, 10))
index

positions = [0, 9, 3]

index[positions]
index.take(positions)

ser = pd.Series(np.random.randn(10))

ser.iloc[positions]
ser.take(positions)
```
For DataFrames, the given indices should be a 1d list or ndarray that specifies
row or column positions.

```python
frm = pd.DataFrame(np.random.randn(5, 3))

frm.take([1, 4, 3])

frm.take([0, 2], axis=1)
```
It is important to note that the ``take`` method on pandas objects are not
intended to work on boolean indices and may return unexpected results.

```python
arr = np.random.randn(10)
arr.take([False, False, True, True])
arr[[0, 1]]

ser = pd.Series(np.random.randn(10))
ser.take([False, False, True, True])
ser.iloc[[0, 1]]
```
Finally, as a small note on performance, because the ``take`` method handles
a narrower range of inputs, it can offer performance that is a good deal
faster than fancy indexing.

```python
arr = np.random.randn(10000, 5)
indexer = np.arange(10000)
random.shuffle(indexer)

%timeit arr[indexer]
%timeit arr.take(indexer, axis=0)
```
```python
ser = pd.Series(arr[:, 0])
%timeit ser.iloc[indexer]
%timeit ser.take(indexer)
```


## Index types
We have discussed ``MultiIndex`` in the previous sections pretty extensively.
Documentation about ``DatetimeIndex`` and ``PeriodIndex`` are shown `here <timeseries.overview>`,
and documentation about ``TimedeltaIndex`` is found `here <timedeltas.index>[.

In the following sub-sections we will highlight some other index types.



### CategoricalIndex
`CategoricalIndex` is a type of index that is useful for supporting
indexing with duplicates. This is a container around a `Categorical`
and allows efficient indexing and storage of an index with a large number of duplicated elements.

```python
from pandas.api.types import CategoricalDtype

df = pd.DataFrame({"A": np.arange(6), "B": list("aabbca")})
df["B"] = df["B"].astype(CategoricalDtype(list("cab")))
df
df.dtypes
df["B"].cat.categories
```
Setting the index will create a ``CategoricalIndex``.

```python
df2 = df.set_index("B")
df2.index
```
Indexing with ``__getitem__/.iloc/.loc`` works similarly to an ``Index`` with duplicates.
The indexers **must** be in the category or the operation will raise a ``KeyError``.

```python
df2.loc["a"]
```
The ``CategoricalIndex`` is **preserved** after indexing:

```python
df2.loc["a"].index
```
Sorting the index will sort by the order of the categories (recall that we
created the index with ``CategoricalDtype(list('cab'))``, so the sorted
order is ``cab``).

```python
df2.sort_index()
```
Groupby operations on the index will preserve the index nature as well.

```python
df2.groupby(level=0, observed=True).sum()
df2.groupby(level=0, observed=True).sum().index
```
Reindexing operations will return a resulting index based on the type of the passed
indexer. Passing a list will return a plain-old ``Index``; indexing with
a ``Categorical`` will return a ``CategoricalIndex``, indexed according to the categories
of the **passed** ``Categorical`` dtype. This allows one to arbitrarily index these even with
values **not** in the categories, similarly to how you can reindex **any** pandas index.

```python
df3 = pd.DataFrame(
    {"A": np.arange(3), "B": pd.Series(list("abc")).astype("category")}
)
df3 = df3.set_index("B")
df3
```
```python
df3.reindex(["a", "e"])
df3.reindex(["a", "e"]).index
df3.reindex(pd.Categorical(["a", "e"], categories=list("abe")))
df3.reindex(pd.Categorical(["a", "e"], categories=list("abe"))).index
```
> **warning.capitalize():**
   Reshaping and Comparison operations on a ``CategoricalIndex`` must have the same categories
   or a ``TypeError`` will be raised.

   ```python
df4 = pd.DataFrame({"A": np.arange(2), "B": list("ba")})
   df4["B"] = df4["B"].astype(CategoricalDtype(list("ab")))
   df4 = df4.set_index("B")
   df4.index

   df5 = pd.DataFrame({"A": np.arange(2), "B": list("bc")})
   df5["B"] = df5["B"].astype(CategoricalDtype(list("bc")))
   df5 = df5.set_index("B")
   df5.index


   :okexcept:

   pd.concat([df4, df5])
```


### RangeIndex
`RangeIndex` is a sub-class of `Index`  that provides the default index for all `DataFrame` and `Series` objects.
``RangeIndex`` is an optimized version of ``Index`` that can represent a monotonic ordered set. These are analogous to Python `range types](https://docs.python.org/3/library/stdtypes.html#typesseq-range)_.
A ``RangeIndex`` will always have an ``int64`` dtype.

```python
idx = pd.RangeIndex(5)
idx
```
``RangeIndex`` is the default index for all `DataFrame` and `Series` objects:

```python
ser = pd.Series([1, 2, 3])
ser.index
df = pd.DataFrame([[1, 2], [3, 4]])
df.index
df.columns
```
A ``RangeIndex`` will behave similarly to a `Index` with an ``int64`` dtype and operations on a ``RangeIndex``,
whose result cannot be represented by a ``RangeIndex``, but should have an integer dtype, will be converted to an ``Index`` with ``int64``.
For example:

```python
idx[[0, 2]]
```


### IntervalIndex
`IntervalIndex` together with its own dtype, `~pandas.api.types.IntervalDtype`
as well as the `Interval` scalar type,  allow first-class support in pandas
for interval notation.

The ``IntervalIndex`` allows some unique indexing and is also used as a
return type for the categories in `cut` and `qcut`.

#### Indexing with an ``IntervalIndex``
An ``IntervalIndex`` can be used in ``Series`` and in ``DataFrame`` as the index.

```python
df = pd.DataFrame(
    {"A": [1, 2, 3, 4]}, index=pd.IntervalIndex.from_breaks([0, 1, 2, 3, 4])
)
df
```
Label based indexing via ``.loc`` along the edges of an interval works as you would expect,
selecting that particular interval.

```python
df.loc[2]
df.loc[[2, 3]]
```
If you select a label *contained* within an interval, this will also select the interval.

```python
df.loc[2.5]
df.loc[[2.5, 3.5]]
```
Selecting using an ``Interval`` will only return exact matches.

```python
df.loc[pd.Interval(1, 2)]
```
Trying to select an ``Interval`` that is not exactly contained in the ``IntervalIndex`` will raise a ``KeyError``.

```python
:okexcept:

df.loc[pd.Interval(0.5, 2.5)]
```
Selecting all ``Intervals`` that overlap a given ``Interval`` can be performed using the
`~IntervalIndex.overlaps` method to create a boolean indexer.

```python
idxr = df.index.overlaps(pd.Interval(0.5, 2.5))
idxr
df[idxr]
```
#### Binning data with ``cut`` and ``qcut``
`cut` and `qcut` both return a ``Categorical`` object, and the bins they
create are stored as an ``IntervalIndex`` in its ``.categories`` attribute.

```python
c = pd.cut(range(4), bins=2)
c
c.categories
```
`cut` also accepts an ``IntervalIndex`` for its ``bins`` argument, which enables
a useful pandas idiom. First, We call `cut` with some data and ``bins`` set to a
fixed number, to generate the bins. Then, we pass the values of ``.categories`` as the
``bins`` argument in subsequent calls to `cut`, supplying new data which will be
binned into the same bins.

```python
pd.cut([0, 3, 5, 1], bins=c.categories)
```
Any value which falls outside all bins will be assigned a ``NaN`` value.

#### Generating ranges of intervals
If we need intervals on a regular frequency, we can use the `interval_range` function
to create an ``IntervalIndex`` using various combinations of ``start``, ``end``, and ``periods``.
The default frequency for ``interval_range`` is a 1 for numeric intervals, and calendar day for
datetime-like intervals:

```python
pd.interval_range(start=0, end=5)

pd.interval_range(start=pd.Timestamp("2017-01-01"), periods=4)

pd.interval_range(end=pd.Timedelta("3 days"), periods=3)
```
The ``freq`` parameter can used to specify non-default frequencies, and can utilize a variety
of `frequency aliases <timeseries.offset_aliases>` with datetime-like intervals:

```python
pd.interval_range(start=0, periods=5, freq=1.5)

pd.interval_range(start=pd.Timestamp("2017-01-01"), periods=4, freq="W")

pd.interval_range(start=pd.Timedelta("0 days"), periods=3, freq="9h")
```
Additionally, the ``closed`` parameter can be used to specify which side(s) the intervals
are closed on.  Intervals are closed on the right side by default.

```python
pd.interval_range(start=0, end=4, closed="both")

pd.interval_range(start=0, end=4, closed="neither")
```
Specifying ``start``, ``end``, and ``periods`` will generate a range of evenly spaced
intervals from ``start`` to ``end`` inclusively, with ``periods`` number of elements
in the resulting ``IntervalIndex``:

```python
pd.interval_range(start=0, end=6, periods=4)

pd.interval_range(pd.Timestamp("2018-01-01"), pd.Timestamp("2018-02-28"), periods=3)
```
## Miscellaneous indexing FAQ
### Integer indexing
Label-based indexing with integer axis labels is a thorny topic. It has been
discussed heavily on mailing lists and among various members of the scientific
Python community. In pandas, our general viewpoint is that labels matter more
than integer locations. Therefore, with an integer axis index *only*
label-based indexing is possible with the standard tools like ``.loc``. The
following code will generate exceptions:

```python
:okexcept:

s = pd.Series(range(5))
s[-1]
df = pd.DataFrame(np.random.randn(5, 4))
df
df.loc[-2:]
```
This deliberate decision was made to prevent ambiguities and subtle bugs (many
users reported finding bugs when the API change was made to stop "falling back"
on position-based indexing).

### Non-monotonic indexes require exact matches
If the index of a ``Series`` or ``DataFrame`` is monotonically increasing or decreasing, then the bounds
of a label-based slice can be outside the range of the index, much like slice indexing a
normal Python ``list``. Monotonicity of an index can be tested with the `~Index.is_monotonic_increasing` and
`~Index.is_monotonic_decreasing` attributes.

```python
df = pd.DataFrame(index=[2, 3, 3, 4, 5], columns=["data"], data=list(range(5)))
df.index.is_monotonic_increasing

# no rows 0 or 1, but still returns rows 2, 3 (both of them), and 4:
df.loc[0:4, :]

# slice is are outside the index, so empty DataFrame is returned
df.loc[13:15, :]
```
On the other hand, if the index is not monotonic, then both slice bounds must be
*unique* members of the index.

```python
df = pd.DataFrame(index=[2, 3, 1, 4, 3, 5], columns=["data"], data=list(range(6)))
df.index.is_monotonic_increasing

# OK because 2 and 4 are in the index
df.loc[2:4, :]
```
```python
:okexcept:

 # 0 is not in the index
 df.loc[0:4, :]

 # 3 is not a unique label
 df.loc[2:3, :]
```
``Index.is_monotonic_increasing`` and ``Index.is_monotonic_decreasing`` only check that
an index is weakly monotonic. To check for strict monotonicity, you can combine one of those with
the `~Index.is_unique` attribute.

```python
weakly_monotonic = pd.Index(["a", "b", "c", "c"])
weakly_monotonic
weakly_monotonic.is_monotonic_increasing
weakly_monotonic.is_monotonic_increasing & weakly_monotonic.is_unique
```


### Endpoints are inclusive
Compared with standard Python sequence slicing in which the slice endpoint is
not inclusive, label-based slicing in pandas **is inclusive**. The primary
reason for this is that it is often not possible to easily determine the
"successor" or next element after a particular label in an index. For example,
consider the following ``Series``:

```python
s = pd.Series(np.random.randn(6), index=list("abcdef"))
s
```
Suppose we wished to slice from ``c`` to ``e``, using integers this would be
accomplished as such:

```python
s[2:5]
```
However, if you only had ``c`` and ``e``, determining the next element in the
index can be somewhat complicated. For example, the following does not work:

```python
:okexcept:

 s.loc['c':'e' + 1]
```
A very common use case is to limit a time series to start and end at two
specific dates. To enable this, we made the design choice to make label-based
slicing include both endpoints:

```python
s.loc["c":"e"]
```
This is most definitely a "practicality beats purity" sort of thing, but it is
something to watch out for if you expect label-based slicing to behave exactly
in the way that standard Python integer slicing works.


### Indexing potentially changes underlying Series dtype
The different indexing operation can potentially change the dtype of a ``Series``.

```python
series1 = pd.Series([1, 2, 3])
series1.dtype
res = series1.reindex([0, 4])
res.dtype
res
```
```python
series2 = pd.Series([True])
series2.dtype
res = series2.reindex_like(series1)
res.dtype
res
```
This is because the (re)indexing operations above silently inserts ``NaNs`` and the ``dtype``
changes accordingly.  This can cause some issues when using ``numpy`` ``ufuncs``
such as ``numpy.logical_and``.

See the `2388` for a more
detailed discussion.

---

# Copy-on-Write (CoW)
> **note.capitalize():**
    Copy-on-Write is now the default with pandas 3.0.

Copy-on-Write was first introduced in version 1.5.0. Starting from version 2.0 most of the
optimizations that become possible through CoW are implemented and supported. All possible
optimizations are supported starting from pandas 2.1.

CoW will lead to more predictable behavior since it is not possible to update more than
one object with one statement, e.g. indexing operations or methods won't have side-effects. Additionally, through
delaying copies as long as possible, the average performance and memory usage will improve.

## Previous behavior
pandas indexing behavior is tricky to understand. Some operations return views while
other return copies. Depending on the result of the operation, mutating one object
might accidentally mutate another:



    In [1]: df = pd.DataFrame({"foo": [1, 2, 3], "bar": [4, 5, 6]})
    In [2]: subset = df["foo"]
    In [3]: subset.iloc[0] = 100
    In [4]: df
    Out[4]:
       foo  bar
    0  100    4
    1    2    5
    2    3    6


Mutating ``subset``, e.g. updating its values, also updated ``df``. The exact behavior was
hard to predict. Copy-on-Write solves accidentally modifying more than one object,
it explicitly disallows this. ``df`` is unchanged:

```python
df = pd.DataFrame({"foo": [1, 2, 3], "bar": [4, 5, 6]})
subset = df["foo"]
subset.iloc[0] = 100
df
```
The following sections will explain what this means and how it impacts existing
applications.



## Migrating to Copy-on-Write
Copy-on-Write is the default and only mode in pandas 3.0. This means that users
need to migrate their code to be compliant with CoW rules.

The default mode in pandas < 3.0 raises warnings for certain cases that will actively
change behavior and thus change user intended behavior.

pandas 2.2 has a warning mode

```python
pd.options.mode.copy_on_write = "warn"
```
that will warn for every operation that will change behavior with CoW. We expect this mode
to be very noisy, since many cases that we don't expect that they will influence users will
also emit a warning. We recommend checking this mode and analyzing the warnings, but it is
not necessary to address all of these warning. The first two items of the following lists
are the only cases that need to be addressed to make existing code work with CoW.

The following few items describe the user visible changes:

**Chained assignment will never work**

``loc`` should be used as an alternative. Check the
`chained assignment section <copy_on_write_chained_assignment>` for more details.

**Accessing the underlying array of a pandas object will return a read-only view**

```python
ser = pd.Series([1, 2, 3])
ser.to_numpy()
```
This example returns a NumPy array that is a view of the Series object. This view can
be modified and thus also modify the pandas object. This is not compliant with CoW
rules. The returned array is set to non-writeable to protect against this behavior.
Creating a copy of this array allows modification. You can also make the array
writeable again if you don't care about the pandas object anymore.

See the section about `read-only NumPy arrays <copy_on_write_read_only_na>`
for more details.

**Only one pandas object is updated at once**

The following code snippet updated both ``df`` and ``subset`` without CoW:



    In [1]: df = pd.DataFrame({"foo": [1, 2, 3], "bar": [4, 5, 6]})
    In [2]: subset = df["foo"]
    In [3]: subset.iloc[0] = 100
    In [4]: df
    Out[4]:
       foo  bar
    0  100    4
    1    2    5
    2    3    6

This is not possible anymore with CoW, since the CoW rules explicitly forbid this.
This includes updating a single column as a `Series` and relying on the change
propagating back to the parent `DataFrame`.
This statement can be rewritten into a single statement with ``loc`` or ``iloc`` if
this behavior is necessary. `DataFrame.where` is another suitable alternative
for this case.

Updating a column selected from a `DataFrame` with an inplace method will
also not work anymore.

```python
:okwarning:

df = pd.DataFrame({"foo": [1, 2, 3], "bar": [4, 5, 6]})
df["foo"].replace(1, 5, inplace=True)
df
```
This is another form of chained assignment. This can generally be rewritten in 2
different forms:

```python
df = pd.DataFrame({"foo": [1, 2, 3], "bar": [4, 5, 6]})
df.replace({"foo": {1: 5}}, inplace=True)
df
```
A different alternative would be to not use ``inplace``:

```python
df = pd.DataFrame({"foo": [1, 2, 3], "bar": [4, 5, 6]})
df["foo"] = df["foo"].replace(1, 5)
df
```
**Constructors now copy NumPy arrays by default**

The Series and DataFrame constructors now copies a NumPy array by default when not
otherwise specified. This was changed to avoid mutating a pandas object when the
NumPy array is changed inplace outside of pandas. You can set ``copy=False`` to
avoid this copy.

## Description
CoW means that any DataFrame or Series derived from another in any way always
behaves as a copy. As a consequence, we can only change the values of an object
through modifying the object itself. CoW disallows updating a DataFrame or a Series
that shares data with another DataFrame or Series object inplace.

This avoids side-effects when modifying values and hence, most methods can avoid
actually copying the data and only trigger a copy when necessary.

The following example will operate inplace:

```python
df = pd.DataFrame({"foo": [1, 2, 3], "bar": [4, 5, 6]})
df.iloc[0, 0] = 100
df
```
The object ``df`` does not share any data with any other object and hence no
copy is triggered when updating the values. In contrast, the following operation
triggers a copy of the data under CoW:


```python
df = pd.DataFrame({"foo": [1, 2, 3], "bar": [4, 5, 6]})
df2 = df.reset_index(drop=True)
df2.iloc[0, 0] = 100

df
df2
```
``reset_index`` returns a lazy copy with CoW while it copies the data without CoW.
Since both objects, ``df`` and ``df2`` share the same data, a copy is triggered
when modifying ``df2``. The object ``df`` still has the same values as initially
while ``df2`` was modified.

If the object ``df`` isn't needed anymore after performing the ``reset_index`` operation,
you can emulate an inplace-like operation through assigning the output of ``reset_index``
to the same variable:

```python
df = pd.DataFrame({"foo": [1, 2, 3], "bar": [4, 5, 6]})
df = df.reset_index(drop=True)
df.iloc[0, 0] = 100
df
```
The initial object gets out of scope as soon as the result of ``reset_index`` is
reassigned and hence ``df`` does not share data with any other object. No copy
is necessary when modifying the object. This is generally true for all methods
listed in `Copy-on-Write optimizations <copy_on_write.optimizations>`.

Previously, when operating on views, the view and the parent object was modified:



    In [1]: df = pd.DataFrame({"foo": [1, 2, 3], "bar": [4, 5, 6]})
    In [2]: subset = df["foo"]
    In [3]: subset.iloc[0] = 100
    In [4]: df
    Out[4]:
       foo  bar
    0  100    4
    1    2    5
    2    3    6

CoW triggers a copy when ``df`` is changed to avoid mutating ``view`` as well:

```python
df = pd.DataFrame({"foo": [1, 2, 3], "bar": [4, 5, 6]})
view = df[:]
df.iloc[0, 0] = 100

df
view
```


## Chained Assignment
Chained assignment references a technique where an object is updated through
two subsequent indexing operations, e.g.



    In [1]: df = pd.DataFrame({"foo": [1, 2, 3], "bar": [4, 5, 6]})
    In [2]: df["foo"][df["bar"] > 5] = 100
    In [3]: df
    Out[3]:
       foo  bar
    0  100    4
    1    2    5
    2    3    6

The column ``foo`` was updated where the column ``bar`` is greater than 5.
This violated the CoW principles though, because it would have to modify the
view ``df["foo"]`` and ``df`` in one step. Hence, chained assignment will
consistently never work and raise a ``ChainedAssignmentError`` warning
with CoW enabled:

```python
:okwarning:

df = pd.DataFrame({"foo": [1, 2, 3], "bar": [4, 5, 6]})
df["foo"][df["bar"] > 5] = 100
```
With copy on write this can be done by using ``loc``.

```python
df.loc[df["bar"] > 5, "foo"] = 100
```


## Read-only NumPy arrays
Accessing the underlying NumPy array of a DataFrame will return a read-only array if the array
shares data with the initial DataFrame:

The array is a copy if the initial DataFrame consists of more than one array:

```python
df = pd.DataFrame({"a": [1, 2], "b": [1.5, 2.5]})
df.to_numpy()
```
The array shares data with the DataFrame if the DataFrame consists of only one NumPy array:

```python
df = pd.DataFrame({"a": [1, 2], "b": [3, 4]})
df.to_numpy()
```
This array is read-only, which means that it can't be modified inplace:

```python
:okexcept:

arr = df.to_numpy()
arr[0, 0] = 100
```
The same holds true for a Series, since a Series always consists of a single array.

There are two potential solutions to this:

- Trigger a copy manually if you want to avoid updating DataFrames that share memory with your array.
- Make the array writeable. This is a more performant solution but circumvents Copy-on-Write rules, so
  it should be used with caution.

```python
arr = df.to_numpy()
arr.flags.writeable = True
arr[0, 0] = 100
arr
```
## Patterns to avoid
No defensive copy will be performed if two objects share the same data while
you are modifying one object inplace.

```python
df = pd.DataFrame({"a": [1, 2, 3], "b": [4, 5, 6]})
df2 = df.reset_index(drop=True)
df2.iloc[0, 0] = 100
```
This creates two objects that share data and thus the setitem operation will trigger a
copy. This is not necessary if the initial object ``df`` isn't needed anymore.
Simply reassigning to the same variable will invalidate the reference that is
held by the object.

```python
df = pd.DataFrame({"a": [1, 2, 3], "b": [4, 5, 6]})
df = df.reset_index(drop=True)
df.iloc[0, 0] = 100
```
No copy is necessary in this example.
Creating multiple references keeps unnecessary references alive
and thus will hurt performance with Copy-on-Write.



## Copy-on-Write optimizations
A new lazy copy mechanism that defers the copy until the object in question is modified
and only if this object shares data with another object. This mechanism was added to
methods that don't require a copy of the underlying data. Popular examples are `DataFrame.drop` for ``axis=1``
and `DataFrame.rename`.

These methods return views when Copy-on-Write is enabled, which provides a significant
performance improvement compared to the regular execution.

---

```python
:suppress:

from matplotlib import pyplot as plt
import pandas.util._doctools as doctools

p = doctools.TablePlotter()
```
# Merge, join, concatenate and compare
pandas provides various methods for combining and comparing `Series` or
`DataFrame`.

* `~pandas.concat`: Merge multiple `Series` or `DataFrame` objects along a shared index or column
* `DataFrame.join`: Merge multiple `DataFrame` objects along the columns
* `DataFrame.combine_first`: Update missing values with non-missing values in the same location
* `~pandas.merge`: Combine two `Series` or `DataFrame` objects with SQL-style joining
* `~pandas.merge_ordered`: Combine two `Series` or `DataFrame` objects along an ordered axis
* `~pandas.merge_asof`: Combine two `Series` or `DataFrame` objects by near instead of exact matching keys
* `Series.compare` and `DataFrame.compare`: Show differences in values between two `Series` or `DataFrame` objects



## `~pandas.concat`
The `~pandas.concat` function concatenates an arbitrary amount of
`Series` or `DataFrame` objects along an axis while
performing optional set logic (union or intersection) of the indexes on
the other axes. Like ``numpy.concatenate``, `~pandas.concat`
takes a list or dict of homogeneously-typed objects and concatenates them.

```python
df1 = pd.DataFrame(
    {
        "A": ["A0", "A1", "A2", "A3"],
        "B": ["B0", "B1", "B2", "B3"],
        "C": ["C0", "C1", "C2", "C3"],
        "D": ["D0", "D1", "D2", "D3"],
    },
    index=[0, 1, 2, 3],
)

df2 = pd.DataFrame(
    {
        "A": ["A4", "A5", "A6", "A7"],
        "B": ["B4", "B5", "B6", "B7"],
        "C": ["C4", "C5", "C6", "C7"],
        "D": ["D4", "D5", "D6", "D7"],
    },
    index=[4, 5, 6, 7],
)

df3 = pd.DataFrame(
    {
        "A": ["A8", "A9", "A10", "A11"],
        "B": ["B8", "B9", "B10", "B11"],
        "C": ["C8", "C9", "C10", "C11"],
        "D": ["D8", "D9", "D10", "D11"],
    },
    index=[8, 9, 10, 11],
)

frames = [df1, df2, df3]
result = pd.concat(frames)
result
```
```python
:suppress:

@savefig merging_concat_basic.png
p.plot(frames, result, labels=["df1", "df2", "df3"], vertical=True);
plt.close("all");
```
> **note.capitalize():**
   `~pandas.concat` makes a full copy of the data, and iteratively
   reusing `~pandas.concat` can create unnecessary copies. Collect all
   `DataFrame` or `Series` objects in a list before using
   `~pandas.concat`.

   ```python
frames = [process_your_file(f) for f in files]
result = pd.concat(frames)
```
> **note.capitalize():**
   When concatenating `DataFrame` with named axes, pandas will attempt to preserve
   these index/column names whenever possible. In the case where all inputs share a
   common name, this name will be assigned to the result. When the input names do
   not all agree, the result will be unnamed. The same is true for `MultiIndex`,
   but the logic is applied separately on a level-by-level basis.


### Joining logic of the resulting axis
The ``join`` keyword specifies how to handle axis values that don't exist in the first
`DataFrame`.

``join='outer'`` takes the union of all axis values.

```python
df4 = pd.DataFrame(
    {
        "B": ["B2", "B3", "B6", "B7"],
        "D": ["D2", "D3", "D6", "D7"],
        "F": ["F2", "F3", "F6", "F7"],
    },
    index=[2, 3, 6, 7],
)
result = pd.concat([df1, df4], axis=1)
result
```
```python
:suppress:

@savefig merging_concat_axis1.png
p.plot([df1, df4], result, labels=["df1", "df4"], vertical=False);
plt.close("all");
```
``join='inner'`` takes the intersection of the axis values.

```python
result = pd.concat([df1, df4], axis=1, join="inner")
result
```
```python
:suppress:

@savefig merging_concat_axis1_inner.png
p.plot([df1, df4], result, labels=["df1", "df4"], vertical=False);
plt.close("all");
```
To perform an effective "left" join using the *exact index* from the original
`DataFrame`, result can be reindexed.

```python
result = pd.concat([df1, df4], axis=1).reindex(df1.index)
result
```
```python
:suppress:

@savefig merging_concat_axis1_join_axes.png
p.plot([df1, df4], result, labels=["df1", "df4"], vertical=False);
plt.close("all");
```


### Ignoring indexes on the concatenation axis
For `DataFrame` objects which don't have a meaningful index, the ``ignore_index``
ignores overlapping indexes.

```python
result = pd.concat([df1, df4], ignore_index=True, sort=False)
result
```
```python
:suppress:

@savefig merging_concat_ignore_index.png
p.plot([df1, df4], result, labels=["df1", "df4"], vertical=True);
plt.close("all");
```


### Concatenating `Series` and `DataFrame` together
You can concatenate a mix of `Series` and `DataFrame` objects. The
`Series` will be transformed to `DataFrame` with the column name as
the name of the `Series`.

```python
s1 = pd.Series(["X0", "X1", "X2", "X3"], name="X")
result = pd.concat([df1, s1], axis=1)
result
```
```python
:suppress:

@savefig merging_concat_mixed_ndim.png
p.plot([df1, s1], result, labels=["df1", "s1"], vertical=False);
plt.close("all");
```
Unnamed `Series` will be numbered consecutively.

```python
s2 = pd.Series(["_0", "_1", "_2", "_3"])
result = pd.concat([df1, s2, s2, s2], axis=1)
result
```
```python
:suppress:

@savefig merging_concat_unnamed_series.png
p.plot([df1, s2], result, labels=["df1", "s2"], vertical=False);
plt.close("all");
```
``ignore_index=True`` will drop all name references.

```python
result = pd.concat([df1, s1], axis=1, ignore_index=True)
result
```
```python
:suppress:

@savefig merging_concat_series_ignore_index.png
p.plot([df1, s1], result, labels=["df1", "s1"], vertical=False);
plt.close("all");
```
### Resulting ``keys``
The ``keys`` argument adds another axis level to the resulting index or column (creating
a `MultiIndex`) associate specific keys with each original `DataFrame`.

```python
result = pd.concat(frames, keys=["x", "y", "z"])
result
result.loc["y"]
```
```python
:suppress:

@savefig merging_concat_keys.png
p.plot(frames, result, labels=["df1", "df2", "df3"], vertical=True)
plt.close("all");
```
The ``keys`` argument can override the column names
when creating a new `DataFrame` based on existing `Series`.

```python
s3 = pd.Series([0, 1, 2, 3], name="foo")
s4 = pd.Series([0, 1, 2, 3])
s5 = pd.Series([0, 1, 4, 5])

pd.concat([s3, s4, s5], axis=1)
pd.concat([s3, s4, s5], axis=1, keys=["red", "blue", "yellow"])
```
You can also pass a dict to `concat` in which case the dict keys will be used
for the ``keys`` argument unless other ``keys`` argument is specified:

```python
pieces = {"x": df1, "y": df2, "z": df3}
result = pd.concat(pieces)
result
```
```python
:suppress:

@savefig merging_concat_dict.png
p.plot([df1, df2, df3], result, labels=["df1", "df2", "df3"], vertical=True);
plt.close("all");
```
```python
result = pd.concat(pieces, keys=["z", "y"])
result
```
```python
:suppress:

@savefig merging_concat_dict_keys.png
p.plot([df1, df2, df3], result, labels=["df1", "df2", "df3"], vertical=True);
plt.close("all");
```
The `MultiIndex` created has levels that are constructed from the passed keys and
the index of the `DataFrame` pieces:

```python
result.index.levels
```
``levels`` argument allows specifying resulting levels associated with the ``keys``.

```python
result = pd.concat(
    pieces, keys=["x", "y", "z"], levels=[["z", "y", "x", "w"]], names=["group_key"]
)
result
```
```python
:suppress:

@savefig merging_concat_dict_keys_names.png
p.plot([df1, df2, df3], result, labels=["df1", "df2", "df3"], vertical=True);
plt.close("all");
```
```python
result.index.levels
```


### Appending rows to a `DataFrame`
If you have a `Series` that you want to append as a single row to a `DataFrame`, you can convert the row into a
`DataFrame` and use `concat`.

```python
s2 = pd.Series(["X0", "X1", "X2", "X3"], index=["A", "B", "C", "D"])
result = pd.concat([df1, s2.to_frame().T], ignore_index=True)
result
```
```python
:suppress:

@savefig merging_append_series_as_row.png
p.plot([df1, s2], result, labels=["df1", "s2"], vertical=True);
plt.close("all");
```


## `~pandas.merge`
`~pandas.merge` performs join operations similar to relational databases like SQL.
Users who are familiar with SQL but new to pandas can reference a
`comparison with SQL<compare_with_sql.join>`.

### Merge types
`~pandas.merge` implements common SQL style joining operations.

* **one-to-one**: joining two `DataFrame` objects on
  their indexes which must contain unique values.
* **many-to-one**: joining a unique index to one or
  more columns in a different `DataFrame`.
* **many-to-many**: joining columns on columns.

> **note.capitalize():**
   When joining columns on columns, potentially a many-to-many join, any
   indexes on the passed `DataFrame` objects **will be discarded**.


For a **many-to-many** join, if a key combination appears
more than once in both tables, the `DataFrame` will have the **Cartesian
product** of the associated data.

```python
left = pd.DataFrame(
    {
        "key": ["K0", "K1", "K2", "K3"],
        "A": ["A0", "A1", "A2", "A3"],
        "B": ["B0", "B1", "B2", "B3"],
    }
)

right = pd.DataFrame(
    {
        "key": ["K0", "K1", "K2", "K3"],
        "C": ["C0", "C1", "C2", "C3"],
        "D": ["D0", "D1", "D2", "D3"],
    }
)
result = pd.merge(left, right, on="key")
result
```
```python
:suppress:

@savefig merging_merge_on_key.png
p.plot([left, right], result, labels=["left", "right"], vertical=False);
plt.close("all");
```
The ``how`` argument to `~pandas.merge` specifies which keys are
included in the resulting table. If a key combination **does not appear** in
either the left or right tables, the values in the joined table will be
``NA``. Here is a summary of the ``how`` options and their SQL equivalent names:


    :header: "Merge method", "SQL Join Name", "Description"
    :widths: 20, 20, 60

    ``left``, ``LEFT OUTER JOIN``, Use keys from left frame only
    ``right``, ``RIGHT OUTER JOIN``, Use keys from right frame only
    ``outer``, ``FULL OUTER JOIN``, Use union of keys from both frames
    ``inner``, ``INNER JOIN``, Use intersection of keys from both frames
    ``cross``, ``CROSS JOIN``, Create the cartesian product of rows of both frames

```python
left = pd.DataFrame(
   {
      "key1": ["K0", "K0", "K1", "K2"],
      "key2": ["K0", "K1", "K0", "K1"],
      "A": ["A0", "A1", "A2", "A3"],
      "B": ["B0", "B1", "B2", "B3"],
   }
)
right = pd.DataFrame(
   {
      "key1": ["K0", "K1", "K1", "K2"],
      "key2": ["K0", "K0", "K0", "K0"],
      "C": ["C0", "C1", "C2", "C3"],
      "D": ["D0", "D1", "D2", "D3"],
   }
)
result = pd.merge(left, right, how="left", on=["key1", "key2"])
result
```
```python
:suppress:

@savefig merging_merge_on_key_left.png
p.plot([left, right], result, labels=["left", "right"], vertical=False);
plt.close("all");
```
```python
result = pd.merge(left, right, how="right", on=["key1", "key2"])
result
```
```python
:suppress:

@savefig merging_merge_on_key_right.png
p.plot([left, right], result, labels=["left", "right"], vertical=False);
```
```python
result = pd.merge(left, right, how="outer", on=["key1", "key2"])
result
```
```python
:suppress:

@savefig merging_merge_on_key_outer.png
p.plot([left, right], result, labels=["left", "right"], vertical=False);
plt.close("all");
```
```python
result = pd.merge(left, right, how="inner", on=["key1", "key2"])
result
```
```python
:suppress:

@savefig merging_merge_on_key_inner.png
p.plot([left, right], result, labels=["left", "right"], vertical=False);
plt.close("all");
```
```python
result = pd.merge(left, right, how="cross")
result
```
```python
:suppress:

@savefig merging_merge_cross.png
p.plot([left, right], result, labels=["left", "right"], vertical=False);
plt.close("all");
```
You can merge `Series` and a `DataFrame` with a `MultiIndex` if the names of
the `MultiIndex` correspond to the columns from the `DataFrame`. You can also
transform the `Series` to a `DataFrame` using `Series.reset_index`
before merging:

```python
df = pd.DataFrame({"Let": ["A", "B", "C"], "Num": [1, 2, 3]})
df

ser = pd.Series(
    ["a", "b", "c", "d", "e", "f"],
    index=pd.MultiIndex.from_arrays(
        [["A", "B", "C"] * 2, [1, 2, 3, 4, 5, 6]], names=["Let", "Num"]
    ),
)
ser

pd.merge(df, ser.reset_index(), on=["Let", "Num"])
```
Performing an outer join with duplicate join keys in `DataFrame`:

```python
left = pd.DataFrame({"A": [1, 2], "B": [2, 2]})

right = pd.DataFrame({"A": [4, 5, 6], "B": [2, 2, 2]})

result = pd.merge(left, right, on="B", how="outer")
result
```
```python
:suppress:

@savefig merging_merge_on_key_dup.png
p.plot([left, right], result, labels=["left", "right"], vertical=False);
plt.close("all");
```
> **warning.capitalize():**
  Merging on duplicate keys significantly increase the dimensions of the result
  and can cause a memory overflow.



### Merge key uniqueness
The ``validate`` argument checks whether the uniqueness of merge keys.
Key uniqueness is checked before merge operations and can protect against memory overflows
and unexpected key duplication.

```python
:okexcept:

left = pd.DataFrame({"A": [1, 2], "B": [1, 2]})
right = pd.DataFrame({"A": [4, 5, 6], "B": [2, 2, 2]})
result = pd.merge(left, right, on="B", how="outer", validate="one_to_one")
```
If the user is aware of the duplicates in the right `DataFrame` but wants to
ensure there are no duplicates in the left `DataFrame`, one can use the
``validate='one_to_many'`` argument instead, which will not raise an exception.

```python
pd.merge(left, right, on="B", how="outer", validate="one_to_many")
```


### Merge result indicator
`~pandas.merge` accepts the argument ``indicator``. If ``True``, a
Categorical-type column called ``_merge`` will be added to the output object
that takes on values:

  ===================================   ================
  Observation Origin                    ``_merge`` value
  ===================================   ================
  Merge key only in ``'left'`` frame    ``left_only``
  Merge key only in ``'right'`` frame   ``right_only``
  Merge key in both frames              ``both``
  ===================================   ================

```python
df1 = pd.DataFrame({"col1": [0, 1], "col_left": ["a", "b"]})
df2 = pd.DataFrame({"col1": [1, 2, 2], "col_right": [2, 2, 2]})
pd.merge(df1, df2, on="col1", how="outer", indicator=True)
```
A string argument to ``indicator`` will use the value as the name for the indicator column.

```python
pd.merge(df1, df2, on="col1", how="outer", indicator="indicator_column")
```
### Overlapping value columns
The merge ``suffixes`` argument takes a tuple or list of strings to append to
overlapping column names in the input `DataFrame` to disambiguate the result
columns:

```python
left = pd.DataFrame({"k": ["K0", "K1", "K2"], "v": [1, 2, 3]})
right = pd.DataFrame({"k": ["K0", "K0", "K3"], "v": [4, 5, 6]})

result = pd.merge(left, right, on="k")
result
```
```python
:suppress:

@savefig merging_merge_overlapped.png
p.plot([left, right], result, labels=["left", "right"], vertical=False);
plt.close("all");
```
```python
result = pd.merge(left, right, on="k", suffixes=("_l", "_r"))
result
```
```python
:suppress:

@savefig merging_merge_overlapped_suffix.png
p.plot([left, right], result, labels=["left", "right"], vertical=False);
plt.close("all");
```
## `DataFrame.join`
`DataFrame.join` combines the columns of multiple,
potentially differently-indexed `DataFrame` into a single result
`DataFrame`.

```python
left = pd.DataFrame(
    {"A": ["A0", "A1", "A2"], "B": ["B0", "B1", "B2"]}, index=["K0", "K1", "K2"]
)

right = pd.DataFrame(
    {"C": ["C0", "C2", "C3"], "D": ["D0", "D2", "D3"]}, index=["K0", "K2", "K3"]
)

result = left.join(right)
result
```
```python
:suppress:

@savefig merging_join.png
p.plot([left, right], result, labels=["left", "right"], vertical=False);
plt.close("all");
```
```python
result = left.join(right, how="outer")
result
```
```python
:suppress:

@savefig merging_join_outer.png
p.plot([left, right], result, labels=["left", "right"], vertical=False);
plt.close("all");
```
```python
result = left.join(right, how="inner")
result
```
```python
:suppress:

@savefig merging_join_inner.png
p.plot([left, right], result, labels=["left", "right"], vertical=False);
plt.close("all");
```
`DataFrame.join` takes an optional ``on`` argument which may be a column
or multiple column names that the passed `DataFrame` is to be
aligned.

```python
left = pd.DataFrame(
    {
        "A": ["A0", "A1", "A2", "A3"],
        "B": ["B0", "B1", "B2", "B3"],
        "key": ["K0", "K1", "K0", "K1"],
    }
)

right = pd.DataFrame({"C": ["C0", "C1"], "D": ["D0", "D1"]}, index=["K0", "K1"])

result = left.join(right, on="key")
result
```
```python
:suppress:

@savefig merging_join_key_columns.png
p.plot([left, right], result, labels=["left", "right"], vertical=False);
plt.close("all");
```
```python
result = pd.merge(
    left, right, left_on="key", right_index=True, how="left", sort=False
)
result
```
```python
:suppress:

@savefig merging_merge_key_columns.png
p.plot([left, right], result, labels=["left", "right"], vertical=False);
plt.close("all");
```


To join on multiple keys, the passed `DataFrame` must have a `MultiIndex`:

```python
left = pd.DataFrame(
    {
        "A": ["A0", "A1", "A2", "A3"],
        "B": ["B0", "B1", "B2", "B3"],
        "key1": ["K0", "K0", "K1", "K2"],
        "key2": ["K0", "K1", "K0", "K1"],
    }
)

index = pd.MultiIndex.from_tuples(
    [("K0", "K0"), ("K1", "K0"), ("K2", "K0"), ("K2", "K1")]
)
right = pd.DataFrame(
    {"C": ["C0", "C1", "C2", "C3"], "D": ["D0", "D1", "D2", "D3"]}, index=index
)
result = left.join(right, on=["key1", "key2"])
result
```
```python
:suppress:

@savefig merging_join_multikeys.png
p.plot([left, right], result, labels=["left", "right"], vertical=False);
plt.close("all");
```


The default for `DataFrame.join` is to perform a left join
which uses only the keys found in the
calling `DataFrame`. Other join types can be specified with ``how``.

```python
result = left.join(right, on=["key1", "key2"], how="inner")
result
```
```python
:suppress:

@savefig merging_join_multikeys_inner.png
p.plot([left, right], result, labels=["left", "right"], vertical=False);
plt.close("all");
```


### Joining a single Index to a MultiIndex
You can join a `DataFrame` with a `Index` to a `DataFrame` with a `MultiIndex` on a level.
The ``name`` of the `Index` will match the level name of the `MultiIndex`.



    left = pd.DataFrame(
        {"A": ["A0", "A1", "A2"], "B": ["B0", "B1", "B2"]},
        index=pd.Index(["K0", "K1", "K2"], name="key"),
    )

    index = pd.MultiIndex.from_tuples(
        [("K0", "Y0"), ("K1", "Y1"), ("K2", "Y2"), ("K2", "Y3")],
        names=["key", "Y"],
    )
    right = pd.DataFrame(
        {"C": ["C0", "C1", "C2", "C3"], "D": ["D0", "D1", "D2", "D3"]},
        index=index,
    )

    result = left.join(right, how="inner")
    result


```python
:suppress:

@savefig merging_join_multiindex_inner.png
p.plot([left, right], result, labels=["left", "right"], vertical=False);
plt.close("all");
```


### Joining with two `MultiIndex`
The `MultiIndex` of the input argument must be completely used
in the join and is a subset of the indices in the left argument.

```python
leftindex = pd.MultiIndex.from_product(
    [list("abc"), list("xy"), [1, 2]], names=["abc", "xy", "num"]
)
left = pd.DataFrame({"v1": range(12)}, index=leftindex)
left

rightindex = pd.MultiIndex.from_product(
    [list("abc"), list("xy")], names=["abc", "xy"]
)
right = pd.DataFrame({"v2": [100 * i for i in range(1, 7)]}, index=rightindex)
right

left.join(right, on=["abc", "xy"], how="inner")
```
```python
leftindex = pd.MultiIndex.from_tuples(
    [("K0", "X0"), ("K0", "X1"), ("K1", "X2")], names=["key", "X"]
)
left = pd.DataFrame(
    {"A": ["A0", "A1", "A2"], "B": ["B0", "B1", "B2"]}, index=leftindex
)

rightindex = pd.MultiIndex.from_tuples(
    [("K0", "Y0"), ("K1", "Y1"), ("K2", "Y2"), ("K2", "Y3")], names=["key", "Y"]
)
right = pd.DataFrame(
    {"C": ["C0", "C1", "C2", "C3"], "D": ["D0", "D1", "D2", "D3"]}, index=rightindex
)

result = pd.merge(
    left.reset_index(), right.reset_index(), on=["key"], how="inner"
).set_index(["key", "X", "Y"])
result
```
```python
:suppress:

@savefig merging_merge_two_multiindex.png
p.plot([left, right], result, labels=["left", "right"], vertical=False);
plt.close("all");
```


### Merging on a combination of columns and index levels
Strings passed as the ``on``, ``left_on``, and ``right_on`` parameters
may refer to either column names or index level names.  This enables merging
`DataFrame` instances on a combination of index levels and columns without
resetting indexes.

```python
left_index = pd.Index(["K0", "K0", "K1", "K2"], name="key1")

left = pd.DataFrame(
    {
        "A": ["A0", "A1", "A2", "A3"],
        "B": ["B0", "B1", "B2", "B3"],
        "key2": ["K0", "K1", "K0", "K1"],
    },
    index=left_index,
)

right_index = pd.Index(["K0", "K1", "K2", "K2"], name="key1")

right = pd.DataFrame(
    {
        "C": ["C0", "C1", "C2", "C3"],
        "D": ["D0", "D1", "D2", "D3"],
        "key2": ["K0", "K0", "K0", "K1"],
    },
    index=right_index,
)

result = left.merge(right, on=["key1", "key2"])
result
```
```python
:suppress:

@savefig merge_on_index_and_column.png
p.plot([left, right], result, labels=["left", "right"], vertical=False);
plt.close("all");
```
> **note.capitalize():**
   When `DataFrame` are joined on a string that matches an index level in both
   arguments, the index level is preserved as an index level in the resulting
   `DataFrame`.

> **note.capitalize():**
   When `DataFrame` are joined using only some of the levels of a `MultiIndex`,
   the extra levels will be dropped from the resulting join. To
   preserve those levels, use `DataFrame.reset_index` on those level
   names to move those levels to columns prior to the join.



### Joining multiple `DataFrame`
A list or tuple of `DataFrame` can also be passed to `~DataFrame.join`
to join them together on their indexes.

```python
right2 = pd.DataFrame({"v": [7, 8, 9]}, index=["K1", "K1", "K2"])
result = left.join([right, right2])
```
```python
:suppress:

@savefig merging_join_multi_df.png
p.plot(
    [left, right, right2],
    result,
    labels=["left", "right", "right2"],
    vertical=False,
);
plt.close("all");
```


### `DataFrame.combine_first`
`DataFrame.combine_first` update missing values from one `DataFrame`
with the non-missing values in another `DataFrame` in the corresponding
location.

```python
df1 = pd.DataFrame(
    [[np.nan, 3.0, 5.0], [-4.6, np.nan, np.nan], [np.nan, 7.0, np.nan]]
)
df2 = pd.DataFrame([[-42.6, np.nan, -8.2], [-5.0, 1.6, 4]], index=[1, 2])
result = df1.combine_first(df2)
result
```
```python
:suppress:

@savefig merging_combine_first.png
p.plot([df1, df2], result, labels=["df1", "df2"], vertical=False);
plt.close("all");
```


## `merge_ordered`
`merge_ordered` combines ordered data such as numeric or time series data
with optional filling of missing data with ``fill_method``.

```python
left = pd.DataFrame(
    {"k": ["K0", "K1", "K1", "K2"], "lv": [1, 2, 3, 4], "s": ["a", "b", "c", "d"]}
)

right = pd.DataFrame({"k": ["K1", "K2", "K4"], "rv": [1, 2, 3]})

pd.merge_ordered(left, right, fill_method="ffill", left_by="s")
```


## `merge_asof`
`merge_asof` is similar to an ordered left-join except that matches are on the
nearest key rather than equal keys. For each row in the ``left`` `DataFrame`,
the last row in the ``right`` `DataFrame` are selected where the ``on`` key is less
than the left's key. Both `DataFrame` must be sorted by the key.

Optionally `merge_asof` can perform a group-wise merge by matching the
``by`` key in addition to the nearest match on the ``on`` key.

```python
trades = pd.DataFrame(
    {
        "time": pd.to_datetime(
            [
                "20160525 13:30:00.023",
                "20160525 13:30:00.038",
                "20160525 13:30:00.048",
                "20160525 13:30:00.048",
                "20160525 13:30:00.048",
            ]
        ),
        "ticker": ["MSFT", "MSFT", "GOOG", "GOOG", "AAPL"],
        "price": [51.95, 51.95, 720.77, 720.92, 98.00],
        "quantity": [75, 155, 100, 100, 100],
    },
    columns=["time", "ticker", "price", "quantity"],
)

quotes = pd.DataFrame(
    {
        "time": pd.to_datetime(
            [
                "20160525 13:30:00.023",
                "20160525 13:30:00.023",
                "20160525 13:30:00.030",
                "20160525 13:30:00.041",
                "20160525 13:30:00.048",
                "20160525 13:30:00.049",
                "20160525 13:30:00.072",
                "20160525 13:30:00.075",
            ]
        ),
        "ticker": ["GOOG", "MSFT", "MSFT", "MSFT", "GOOG", "AAPL", "GOOG", "MSFT"],
        "bid": [720.50, 51.95, 51.97, 51.99, 720.50, 97.99, 720.50, 52.01],
        "ask": [720.93, 51.96, 51.98, 52.00, 720.93, 98.01, 720.88, 52.03],
    },
    columns=["time", "ticker", "bid", "ask"],
)
trades
quotes
pd.merge_asof(trades, quotes, on="time", by="ticker")
```
`merge_asof` within ``2ms`` between the quote time and the trade time.

```python
pd.merge_asof(trades, quotes, on="time", by="ticker", tolerance=pd.Timedelta("2ms"))
```
`merge_asof` within ``10ms`` between the quote time and the trade time and
exclude exact matches on time. Note that though we exclude the exact matches
(of the quotes), prior quotes **do** propagate to that point in time.

```python
pd.merge_asof(
    trades,
    quotes,
    on="time",
    by="ticker",
    tolerance=pd.Timedelta("10ms"),
    allow_exact_matches=False,
)
```


## `~Series.compare`
The `Series.compare` and `DataFrame.compare` methods allow you to
compare two `DataFrame` or `Series`, respectively, and summarize their differences.

```python
df = pd.DataFrame(
    {
        "col1": ["a", "a", "b", "b", "a"],
        "col2": [1.0, 2.0, 3.0, np.nan, 5.0],
        "col3": [1.0, 2.0, 3.0, 4.0, 5.0],
    },
    columns=["col1", "col2", "col3"],
)
df
df2 = df.copy()
df2.loc[0, "col1"] = "c"
df2.loc[2, "col3"] = 4.0
df2
df.compare(df2)
```
By default, if two corresponding values are equal, they will be shown as ``NaN``.
Furthermore, if all values in an entire row / column are equal, that row / column will be
omitted from the result. The remaining differences will be aligned on columns.

Stack the differences on rows.

```python
df.compare(df2, align_axis=0)
```
Keep all original rows and columns with ``keep_shape=True``.

```python
df.compare(df2, keep_shape=True)
```
Keep all the original values even if they are equal.

```python
df.compare(df2, keep_shape=True, keep_equal=True)
```

---

# Reshaping and pivot tables



pandas provides methods for manipulating a `Series` and `DataFrame` to alter the
representation of the data for further data processing or data summarization.

* `~pandas.pivot` and `~pandas.pivot_table`: Group unique values within one or more discrete categories.
* `~DataFrame.stack` and `~DataFrame.unstack`: Pivot a column or row level to the opposite axis respectively.
* `~pandas.melt` and `~pandas.wide_to_long`: Unpivot a wide `DataFrame` to a long format.
* `~pandas.get_dummies` and `~pandas.from_dummies`: Conversions with indicator variables.
* `~Series.explode`: Convert a column of list-like values to individual rows.
* `~pandas.crosstab`: Calculate a cross-tabulation of multiple 1 dimensional factor arrays.
* `~pandas.cut`: Transform continuous variables to discrete, categorical values
* `~pandas.factorize`: Encode 1 dimensional variables into integer labels.


## `~pandas.pivot` and `~pandas.pivot_table`


### `~pandas.pivot`
Data is often stored in so-called "stacked" or "record" format. In a "record" or "wide" format,
typically there is one row for each subject. In the "stacked" or "long" format there are
multiple rows for each subject where applicable.

```python
data = {
   "value": range(12),
   "variable": ["A"] * 3 + ["B"] * 3 + ["C"] * 3 + ["D"] * 3,
   "date": pd.to_datetime(["2020-01-03", "2020-01-04", "2020-01-05"] * 4)
}
df = pd.DataFrame(data)
```
To perform time series operations with each unique variable, a better
representation would be where the ``columns`` are the unique variables and an
``index`` of dates identifies individual observations. To reshape the data into
this form, we use the `DataFrame.pivot` method (also implemented as a
top level function `~pandas.pivot`):

```python
pivoted = df.pivot(index="date", columns="variable", values="value")
pivoted
```
If the ``values`` argument is omitted, and the input `DataFrame` has more than
one column of values which are not used as column or index inputs to `~DataFrame.pivot`,
then the resulting "pivoted" `DataFrame` will have hierarchical columns
 whose topmost level indicates the respective value
column:

```python
df["value2"] = df["value"] * 2
pivoted = df.pivot(index="date", columns="variable")
pivoted
```
You can then select subsets from the pivoted `DataFrame`:

```python
pivoted["value2"]
```
Note that this returns a view on the underlying data in the case where the data
are homogeneously-typed.

> **note.capitalize():**
   `~pandas.pivot` can only handle unique rows specified by ``index`` and ``columns``.
   If you data contains duplicates, use `~pandas.pivot_table`.




### `~pandas.pivot_table`
While `~DataFrame.pivot` provides general purpose pivoting with various
data types, pandas also provides `~pandas.pivot_table` or `~DataFrame.pivot_table`
for pivoting with aggregation of numeric data.

The function `~pandas.pivot_table` can be used to create spreadsheet-style
pivot tables. See the `cookbook<cookbook.pivot>` for some advanced
strategies.

```python
import datetime

df = pd.DataFrame(
    {
        "A": ["one", "one", "two", "three"] * 6,
        "B": ["A", "B", "C"] * 8,
        "C": ["foo", "foo", "foo", "bar", "bar", "bar"] * 4,
        "D": np.random.randn(24),
        "E": np.random.randn(24),
        "F": [datetime.datetime(2013, i, 1) for i in range(1, 13)]
        + [datetime.datetime(2013, i, 15) for i in range(1, 13)],
    }
)
df
pd.pivot_table(df, values="D", index=["A", "B"], columns=["C"])
pd.pivot_table(
    df, values=["D", "E"],
    index=["B"],
    columns=["A", "C"],
    aggfunc="sum",
)
pd.pivot_table(
    df, values="E",
    index=["B", "C"],
    columns=["A"],
    aggfunc=["sum", "mean"],
)
```
The result is a `DataFrame` potentially having a `MultiIndex` on the
index or column. If the ``values`` column name is not given, the pivot table
will include all of the data in an additional level of hierarchy in the columns:

```python
pd.pivot_table(df[["A", "B", "C", "D", "E"]], index=["A", "B"], columns=["C"])
```
Also, you can use `Grouper` for ``index`` and ``columns`` keywords. For detail of `Grouper`, see `Grouping with a Grouper specification <groupby.specify>`.

```python
pd.pivot_table(df, values="D", index=pd.Grouper(freq="ME", key="F"), columns="C")
```


#### Adding margins
Passing ``margins=True`` to `~DataFrame.pivot_table` will add a row and column with an
``All`` label with partial group aggregates across the categories on the
rows and columns:

```python
table = df.pivot_table(
    index=["A", "B"],
    columns="C",
    values=["D", "E"],
    margins=True,
    aggfunc="std"
)
table
```
Additionally, you can call `DataFrame.stack` to display a pivoted DataFrame
as having a multi-level index:

```python
table.stack()
```


## `~DataFrame.stack` and `~DataFrame.unstack`


Closely related to the `~DataFrame.pivot` method are the related
`~DataFrame.stack` and `~DataFrame.unstack` methods available on
`Series` and `DataFrame`. These methods are designed to work together with
`MultiIndex` objects (see the section on hierarchical indexing
).

* `~DataFrame.stack`: "pivot" a level of the (possibly hierarchical) column labels,
  returning a `DataFrame` with an index with a new inner-most level of row
  labels.
* `~DataFrame.unstack`: (inverse operation of `~DataFrame.stack`) "pivot" a level of the
  (possibly hierarchical) row index to the column axis, producing a reshaped
  `DataFrame` with a new inner-most level of column labels.



```python
tuples = [
   ["bar", "bar", "baz", "baz", "foo", "foo", "qux", "qux"],
   ["one", "two", "one", "two", "one", "two", "one", "two"],
]
index = pd.MultiIndex.from_arrays(tuples, names=["first", "second"])
df = pd.DataFrame(np.random.randn(8, 2), index=index, columns=["A", "B"])
df2 = df[:4]
df2
```
The `~DataFrame.stack` function "compresses" a level in the `DataFrame` columns to
produce either:

* A `Series`, in the case of a `Index` in the columns.
* A `DataFrame`, in the case of a `MultiIndex` in the columns.

If the columns have a `MultiIndex`, you can choose which level to stack. The
stacked level becomes the new lowest level in a `MultiIndex` on the columns:

```python
stacked = df2.stack()
stacked
```
With a "stacked" `DataFrame` or `Series` (having a `MultiIndex` as the
``index``), the inverse operation of `~DataFrame.stack` is `~DataFrame.unstack`, which by default
unstacks the **last level**:

```python
stacked.unstack()
stacked.unstack(1)
stacked.unstack(0)
```




If the indexes have names, you can use the level names instead of specifying
the level numbers:

```python
stacked.unstack("second")
```


Notice that the `~DataFrame.stack` and `~DataFrame.unstack` methods implicitly sort the index
levels involved. Hence a call to `~DataFrame.stack` and then `~DataFrame.unstack`, or vice versa,
will result in a **sorted** copy of the original `DataFrame` or `Series`:

```python
index = pd.MultiIndex.from_product([[2, 1], ["a", "b"]])
df = pd.DataFrame(np.random.randn(4), index=index, columns=["A"])
df
all(df.unstack().stack() == df.sort_index())
```


### Multiple levels
You may also stack or unstack more than one level at a time by passing a list
of levels, in which case the end result is as if each level in the list were
processed individually.

```python
columns = pd.MultiIndex.from_tuples(
    [
        ("A", "cat", "long"),
        ("B", "cat", "long"),
        ("A", "dog", "short"),
        ("B", "dog", "short"),
    ],
    names=["exp", "animal", "hair_length"],
)
df = pd.DataFrame(np.random.randn(4, 4), columns=columns)
df

df.stack(level=["animal", "hair_length"])
```
The list of levels can contain either level names or level numbers but
not a mixture of the two.

```python
# df.stack(level=['animal', 'hair_length'])
# from above is equivalent to:
df.stack(level=[1, 2])
```
### Missing data
Unstacking can result in missing values if subgroups do not have the same
set of labels. By default, missing values will be replaced with the default
fill value for that data type.

```python
columns = pd.MultiIndex.from_tuples(
    [
        ("A", "cat"),
        ("B", "dog"),
        ("B", "cat"),
        ("A", "dog"),
    ],
    names=["exp", "animal"],
)
index = pd.MultiIndex.from_product(
    [("bar", "baz", "foo", "qux"), ("one", "two")], names=["first", "second"]
)
df = pd.DataFrame(np.random.randn(8, 4), index=index, columns=columns)
df3 = df.iloc[[0, 1, 4, 7], [1, 2]]
df3
df3.unstack()
```
The missing value can be filled with a specific value with the ``fill_value`` argument.

```python
df3.unstack(fill_value=-1e9)
```


## `~pandas.melt` and `~pandas.wide_to_long`


The top-level `~pandas.melt` function and the corresponding `DataFrame.melt`
are useful to reshape a `DataFrame` into a format where one or more columns
are *identifier variables*, while all other columns, considered *measured
variables*, are "unpivoted" to the row axis, leaving just two non-identifier
columns, "variable" and "value". The names of those columns can be customized
by supplying the ``var_name`` and ``value_name`` parameters.

```python
cheese = pd.DataFrame(
    {
        "first": ["John", "Mary"],
        "last": ["Doe", "Bo"],
        "height": [5.5, 6.0],
        "weight": [130, 150],
    }
)
cheese
cheese.melt(id_vars=["first", "last"])
cheese.melt(id_vars=["first", "last"], var_name="quantity")
```
When transforming a DataFrame using `~pandas.melt`, the index will be ignored.
The original index values can be kept by setting the ``ignore_index=False`` parameter to ``False`` (default is ``True``).
``ignore_index=False`` will however duplicate index values.

```python
index = pd.MultiIndex.from_tuples([("person", "A"), ("person", "B")])
cheese = pd.DataFrame(
    {
        "first": ["John", "Mary"],
        "last": ["Doe", "Bo"],
        "height": [5.5, 6.0],
        "weight": [130, 150],
    },
    index=index,
)
cheese
cheese.melt(id_vars=["first", "last"])
cheese.melt(id_vars=["first", "last"], ignore_index=False)
```
`~pandas.wide_to_long` is similar to `~pandas.melt` with more customization for
column matching.

```python
dft = pd.DataFrame(
    {
        "A1970": {0: "a", 1: "b", 2: "c"},
        "A1980": {0: "d", 1: "e", 2: "f"},
        "B1970": {0: 2.5, 1: 1.2, 2: 0.7},
        "B1980": {0: 3.2, 1: 1.3, 2: 0.1},
        "X": dict(zip(range(3), np.random.randn(3))),
    }
)
dft["id"] = dft.index
dft
pd.wide_to_long(dft, ["A", "B"], i="id", j="year")
```


## `~pandas.get_dummies` and `~pandas.from_dummies`
To convert categorical variables of a `Series` into a "dummy" or "indicator",
`~pandas.get_dummies` creates a new `DataFrame` with columns of the unique
variables and the values representing the presence of those variables per row.

```python
df = pd.DataFrame({"key": list("bbacab"), "data1": range(6)})

pd.get_dummies(df["key"])
df["key"].str.get_dummies()
```
``prefix`` adds a prefix to the column names which is useful for merging the result
with the original `DataFrame`:

```python
dummies = pd.get_dummies(df["key"], prefix="key")
dummies

df[["data1"]].join(dummies)
```
This function is often used along with discretization functions like `~pandas.cut`:

```python
values = np.random.randn(10)
values

bins = [0, 0.2, 0.4, 0.6, 0.8, 1]

pd.get_dummies(pd.cut(values, bins))
```
`get_dummies` also accepts a `DataFrame`. By default, ``object``, ``string``,
or ``categorical`` type columns are encoded as dummy variables with other columns unaltered.

```python
df = pd.DataFrame({"A": ["a", "b", "a"], "B": ["c", "c", "b"], "C": [1, 2, 3]})
pd.get_dummies(df)
```
Specifying the ``columns`` keyword will encode a column of any type.

```python
pd.get_dummies(df, columns=["A"])
```
As with the `Series` version, you can pass values for the ``prefix`` and
``prefix_sep``. By default the column name is used as the prefix and ``_`` as
the prefix separator. You can specify ``prefix`` and ``prefix_sep`` in 3 ways:

* string: Use the same value for ``prefix`` or ``prefix_sep`` for each column
  to be encoded.
* list: Must be the same length as the number of columns being encoded.
* dict: Mapping column name to prefix.

```python
simple = pd.get_dummies(df, prefix="new_prefix")
simple
from_list = pd.get_dummies(df, prefix=["from_A", "from_B"])
from_list
from_dict = pd.get_dummies(df, prefix={"B": "from_B", "A": "from_A"})
from_dict
```
To avoid collinearity when feeding the result to statistical models,
specify ``drop_first=True``.

```python
s = pd.Series(list("abcaa"))

pd.get_dummies(s)

pd.get_dummies(s, drop_first=True)
```
When a column contains only one level, it will be omitted in the result.

```python
df = pd.DataFrame({"A": list("aaaaa"), "B": list("ababc")})

pd.get_dummies(df)

pd.get_dummies(df, drop_first=True)
```
The values can be cast to a different type using the ``dtype`` argument.

```python
df = pd.DataFrame({"A": list("abc"), "B": [1.1, 2.2, 3.3]})

pd.get_dummies(df, dtype=np.float32).dtypes
```


`~pandas.from_dummies` converts the output of `~pandas.get_dummies` back into
a `Series` of categorical values from indicator values.

```python
df = pd.DataFrame({"prefix_a": [0, 1, 0], "prefix_b": [1, 0, 1]})
df

pd.from_dummies(df, sep="_")
```
Dummy coded data only requires ``k - 1`` categories to be included, in this case
the last category is the default category. The default category can be modified with
``default_category``.

```python
df = pd.DataFrame({"prefix_a": [0, 1, 0]})
df

pd.from_dummies(df, sep="_", default_category="b")
```


## `~Series.explode`
For a `DataFrame` column with nested, list-like values, `~Series.explode` will transform
each list-like value to a separate row. The resulting `Index` will be duplicated corresponding
to the index label from the original row:

```python
keys = ["panda1", "panda2", "panda3"]
values = [["eats", "shoots"], ["shoots", "leaves"], ["eats", "leaves"]]
df = pd.DataFrame({"keys": keys, "values": values})
df
df["values"].explode()
```
`DataFrame.explode` can also explode the column in the `DataFrame`.

```python
df.explode("values")
```
`Series.explode` will replace empty lists with a missing value indicator and preserve scalar entries.

```python
s = pd.Series([[1, 2, 3], "foo", [], ["a", "b"]])
s
s.explode()
```
A comma-separated string value can be split into individual values in a list and then exploded to a new row.

```python
df = pd.DataFrame([{"var1": "a,b,c", "var2": 1}, {"var1": "d,e,f", "var2": 2}])
df.assign(var1=df.var1.str.split(",")).explode("var1")
```


## `~pandas.crosstab`
Use `~pandas.crosstab` to compute a cross-tabulation of two (or more)
factors. By default `~pandas.crosstab` computes a frequency table of the factors
unless an array of values and an aggregation function are passed.

Any `Series` passed will have their name attributes used unless row or column
names for the cross-tabulation are specified

```python
a = np.array(["foo", "foo", "bar", "bar", "foo", "foo"], dtype=object)
b = np.array(["one", "one", "two", "one", "two", "one"], dtype=object)
c = np.array(["dull", "dull", "shiny", "dull", "dull", "shiny"], dtype=object)
pd.crosstab(a, [b, c], rownames=["a"], colnames=["b", "c"])
```
If `~pandas.crosstab` receives only two `Series`, it will provide a frequency table.

```python
df = pd.DataFrame(
    {"A": [1, 2, 2, 2, 2], "B": [3, 3, 4, 4, 4], "C": [1, 1, np.nan, 1, 1]}
)
df

pd.crosstab(df["A"], df["B"])
```
`~pandas.crosstab` can also summarize to `Categorical` data.

```python
foo = pd.Categorical(["a", "b"], categories=["a", "b", "c"])
bar = pd.Categorical(["d", "e"], categories=["d", "e", "f"])
pd.crosstab(foo, bar)
```
For `Categorical` data, to include **all** of data categories even if the actual data does
not contain any instances of a particular category, use ``dropna=False``.

```python
pd.crosstab(foo, bar, dropna=False)
```
### Normalization
Frequency tables can also be normalized to show percentages rather than counts
using the ``normalize`` argument:

```python
pd.crosstab(df["A"], df["B"], normalize=True)
```
``normalize`` can also normalize values within each row or within each column:

```python
pd.crosstab(df["A"], df["B"], normalize="columns")
```
`~pandas.crosstab` can also accept a third `Series` and an aggregation function
(``aggfunc``) that will be applied to the values of the third `Series` within
each group defined by the first two `Series`:

```python
pd.crosstab(df["A"], df["B"], values=df["C"], aggfunc="sum")
```
### Adding margins
``margins=True`` will add a row and column with an ``All`` label with partial group aggregates
across the categories on the rows and columns:

```python
pd.crosstab(
    df["A"], df["B"], values=df["C"], aggfunc="sum", normalize=True, margins=True
)
```



## `~pandas.cut`
The `~pandas.cut` function computes groupings for the values of the input
array and is often used to transform continuous variables to discrete or
categorical variables:


An integer ``bins`` will form equal-width bins.

```python
ages = np.array([10, 15, 13, 12, 23, 25, 28, 59, 60])

pd.cut(ages, bins=3)
```
A list of ordered bin edges will assign an interval for each variable.

```python
pd.cut(ages, bins=[0, 18, 35, 70])
```
If the ``bins`` keyword is an `IntervalIndex`, then these will be
used to bin the passed data.

```python
pd.cut(ages, bins=pd.IntervalIndex.from_breaks([0, 40, 70]))
```


## `~pandas.factorize`
`~pandas.factorize` encodes 1 dimensional values into integer labels. Missing values
are encoded as ``-1``.

```python
x = pd.Series(["A", "A", np.nan, "B", 3.14, np.inf])
x
labels, uniques = pd.factorize(x)
labels
uniques
```
`Categorical` will similarly encode 1 dimensional values for further
categorical operations

```python
pd.Categorical(x)
```

---

# 
# Working with text data


## Text data types
There are two ways to store text data in pandas:

1. ``object`` dtype NumPy array.
2. `StringDtype` extension type.

We recommend using `StringDtype` to store text data.

Prior to pandas 1.0, ``object`` dtype was the only option. This was unfortunate
for many reasons:

1. You can accidentally store a *mixture* of strings and non-strings in an
   ``object`` dtype array. It's better to have a dedicated dtype.
2. ``object`` dtype breaks dtype-specific operations like `DataFrame.select_dtypes`.
   There isn't a clear way to select *just* text while excluding non-text
   but still object-dtype columns.
3. When reading code, the contents of an ``object`` dtype array is less clear
   than ``'string'``.

Currently, the performance of ``object`` dtype arrays of strings and
`arrays.StringArray` are about the same. We expect future enhancements
to significantly increase the performance and lower the memory overhead of
`~arrays.StringArray`.

> **warning.capitalize():**
   ``StringArray`` is currently considered experimental. The implementation
   and parts of the API may change without warning.

For backwards-compatibility, ``object`` dtype remains the default type we
infer a list of strings to:

```python
pd.Series(["a", "b", "c"])
```
To explicitly request ``string`` dtype, specify the ``dtype``:

```python
pd.Series(["a", "b", "c"], dtype="string")
pd.Series(["a", "b", "c"], dtype=pd.StringDtype())
```
Or ``astype`` after the ``Series`` or ``DataFrame`` is created:

```python
s = pd.Series(["a", "b", "c"])
s
s.astype("string")
```
You can also use `StringDtype`/``"string"`` as the dtype on non-string data and
it will be converted to ``string`` dtype:

```python
s = pd.Series(["a", 2, np.nan], dtype="string")
s
type(s[1])
```
or convert from existing pandas data:

```python
s1 = pd.Series([1, 2, np.nan], dtype="Int64")
s1
s2 = s1.astype("string")
s2
type(s2[0])
```


#### Behavior differences
These are places where the behavior of ``StringDtype`` objects differ from
``object`` dtype:

1. For ``StringDtype``, `string accessor methods<api.series.str>`
   that return **numeric** output will always return a nullable integer dtype,
   rather than either int or float dtype, depending on the presence of NA values.
   Methods returning **boolean** output will return a nullable boolean dtype.

   ```python
s = pd.Series(["a", None, "b"], dtype="string")
   s
   s.str.count("a")
   s.dropna().str.count("a")

Both outputs are ``Int64`` dtype. Compare that with object-dtype:



   s2 = pd.Series(["a", None, "b"], dtype="object")
   s2.str.count("a")
   s2.dropna().str.count("a")

When NA values are present, the output dtype is float64. Similarly for
methods returning boolean values.



   s.str.isdigit()
   s.str.match("a")
```
2. Some string methods, like `Series.str.decode` are not available
   on ``StringArray`` because ``StringArray`` only holds strings, not
   bytes.
3. In comparison operations, `arrays.StringArray` and ``Series`` backed
   by a ``StringArray`` will return an object with `BooleanDtype`,
   rather than a ``bool`` dtype object. Missing values in a ``StringArray``
   will propagate in comparison operations, rather than always comparing
   unequal like `numpy.nan`.

Everything else that follows in the rest of this document applies equally to
``string`` and ``object`` dtype.



## String methods
Series and Index are equipped with a set of string processing methods
that make it easy to operate on each element of the array. Perhaps most
importantly, these methods exclude missing/NA values automatically. These are
accessed via the ``str`` attribute and generally have names matching
the equivalent (scalar) built-in string methods:

```python
s = pd.Series(
    ["A", "B", "C", "Aaba", "Baca", np.nan, "CABA", "dog", "cat"], dtype="string"
)
s.str.lower()
s.str.upper()
s.str.len()
```
```python
idx = pd.Index([" jack", "jill ", " jesse ", "frank"])
idx.str.strip()
idx.str.lstrip()
idx.str.rstrip()
```
The string methods on Index are especially useful for cleaning up or
transforming DataFrame columns. For instance, you may have columns with
leading or trailing whitespace:

```python
df = pd.DataFrame(
    np.random.randn(3, 2), columns=[" Column A ", " Column B "], index=range(3)
)
df
```
Since ``df.columns`` is an Index object, we can use the ``.str`` accessor

```python
df.columns.str.strip()
df.columns.str.lower()
```
These string methods can then be used to clean up the columns as needed.
Here we are removing leading and trailing whitespaces, lower casing all names,
and replacing any remaining whitespaces with underscores:

```python
df.columns = df.columns.str.strip().str.lower().str.replace(" ", "_")
df
```
> **note.capitalize():**
    If you have a ``Series`` where lots of elements are repeated
    (i.e. the number of unique elements in the ``Series`` is a lot smaller than the length of the
    ``Series``), it can be faster to convert the original ``Series`` to one of type
    ``category`` and then use ``.str.<method>`` or ``.dt.<property>`` on that.
    The performance difference comes from the fact that, for ``Series`` of type ``category``, the
    string operations are done on the ``.categories`` and not on each element of the
    ``Series``.

    Please note that a ``Series`` of type ``category`` with string ``.categories`` has
    some limitations in comparison to ``Series`` of type string (e.g. you can't add strings to
    each other: ``s + " " + s`` won't work if ``s`` is a ``Series`` of type ``category``). Also,
    ``.str`` methods which operate on elements of type ``list`` are not available on such a
    ``Series``.



> **warning.capitalize():**
    The type of the Series is inferred and is one among the allowed types (i.e. strings).

    Generally speaking, the ``.str`` accessor is intended to work only on strings. With very few
    exceptions, other uses are not supported, and may be disabled at a later point.



## Splitting and replacing strings
Methods like ``split`` return a Series of lists:

```python
s2 = pd.Series(["a_b_c", "c_d_e", np.nan, "f_g_h"], dtype="string")
s2.str.split("_")
```
Elements in the split lists can be accessed using ``get`` or ``[]`` notation:

```python
s2.str.split("_").str.get(1)
s2.str.split("_").str[1]
```
It is easy to expand this to return a DataFrame using ``expand``.

```python
s2.str.split("_", expand=True)
```
When original ``Series`` has `StringDtype`, the output columns will all
be `StringDtype` as well.

It is also possible to limit the number of splits:

```python
s2.str.split("_", expand=True, n=1)
```
``rsplit`` is similar to ``split`` except it works in the reverse direction,
i.e., from the end of the string to the beginning of the string:

```python
s2.str.rsplit("_", expand=True, n=1)
```
``replace`` optionally uses `regular expressions
<https://docs.python.org/3/library/re.html>`__:

```python
s3 = pd.Series(
    ["A", "B", "C", "Aaba", "Baca", "", np.nan, "CABA", "dog", "cat"],
    dtype="string",
)
s3
s3.str.replace("^.a|dog", "XX-XX ", case=False, regex=True)
```


Single character pattern with ``regex=True`` will also be treated as regular expressions:

```python
s4 = pd.Series(["a.b", ".", "b", np.nan, ""], dtype="string")
s4
s4.str.replace(".", "a", regex=True)
```
If you want literal replacement of a string (equivalent to `str.replace`), you
can set the optional ``regex`` parameter to ``False``, rather than escaping each
character. In this case both ``pat`` and ``repl`` must be strings:

```python
dollars = pd.Series(["12", "-$10", "$10,000"], dtype="string")

# These lines are equivalent
dollars.str.replace(r"-\$", "-", regex=True)
dollars.str.replace("-$", "-", regex=False)
```
The ``replace`` method can also take a callable as replacement. It is called
on every ``pat`` using `re.sub`. The callable should expect one
positional argument (a regex object) and return a string.

```python
# Reverse every lowercase alphabetic word
pat = r"[a-z]+"

def repl(m):
    return m.group(0)[::-1]

pd.Series(["foo 123", "bar baz", np.nan], dtype="string").str.replace(
    pat, repl, regex=True
)

# Using regex groups
pat = r"(?P<one>\w+) (?P<two>\w+) (?P<three>\w+)"

def repl(m):
    return m.group("two").swapcase()

pd.Series(["Foo Bar Baz", np.nan], dtype="string").str.replace(
    pat, repl, regex=True
)
[``
The ``replace`` method also accepts a compiled regular expression object
from `re.compile` as a pattern. All flags should be included in the
compiled regular expression object.

```python
import re

regex_pat = re.compile(r"^.a|dog", flags=re.IGNORECASE)
s3.str.replace(regex_pat, "XX-XX ", regex=True)
```
Including a ``flags`` argument when calling ``replace`` with a compiled
regular expression object will raise a ``ValueError``.



    @verbatim
    In [1]: s3.str.replace(regex_pat, 'XX-XX ', flags=re.IGNORECASE)
    ---------------------------------------------------------------------------
    ValueError: case and flags cannot be set when pat is a compiled regex

``removeprefix`` and ``removesuffix`` have the same effect as ``str.removeprefix`` and ``str.removesuffix`` added in
`Python 3.9](https://docs.python.org/3/library/stdtypes.html#str.removeprefix)_:



```python
s = pd.Series(["str_foo", "str_bar", "no_prefix"])
s.str.removeprefix("str_")

s = pd.Series(["foo_str", "bar_str", "no_suffix"])
s.str.removesuffix("_str")
```


## Concatenation
There are several ways to concatenate a ``Series`` or ``Index``, either with itself or others, all based on `~Series.str.cat`,
resp. ``Index.str.cat``.

#### Concatenating a single Series into a string
The content of a ``Series`` (or ``Index``) can be concatenated:

```python
s = pd.Series(["a", "b", "c", "d"], dtype="string")
s.str.cat(sep=",")
```
If not specified, the keyword ``sep`` for the separator defaults to the empty string, ``sep=''``:

```python
s.str.cat()
```
By default, missing values are ignored. Using ``na_rep``, they can be given a representation:

```python
t = pd.Series(["a", "b", np.nan, "d"], dtype="string")
t.str.cat(sep=",")
t.str.cat(sep=",", na_rep="-")
```
#### Concatenating a Series and something list-like into a Series
The first argument to `~Series.str.cat` can be a list-like object, provided that it matches the length of the calling ``Series`` (or ``Index``).

```python
s.str.cat(["A", "B", "C", "D"])
```
Missing values on either side will result in missing values in the result as well, *unless* ``na_rep`` is specified:

```python
s.str.cat(t)
s.str.cat(t, na_rep="-")
```
#### Concatenating a Series and something array-like into a Series
The parameter ``others`` can also be two-dimensional. In this case, the number or rows must match the lengths of the calling ``Series`` (or ``Index``).

```python
d = pd.concat([t, s], axis=1)
s
d
s.str.cat(d, na_rep="-")
```
#### Concatenating a Series and an indexed object into a Series, with alignment
For concatenation with a ``Series`` or ``DataFrame``, it is possible to align the indexes before concatenation by setting
the ``join``-keyword.

```python
:okwarning:

u = pd.Series(["b", "d", "a", "c"], index=[1, 3, 0, 2], dtype="string")
s
u
s.str.cat(u)
s.str.cat(u, join="left")
```
The usual options are available for ``join`` (one of ``'left', 'outer', 'inner', 'right'``).
In particular, alignment also means that the different lengths do not need to coincide anymore.

```python
v = pd.Series(["z", "a", "b", "d", "e"], index=[-1, 0, 1, 3, 4], dtype="string")
s
v
s.str.cat(v, join="left", na_rep="-")
s.str.cat(v, join="outer", na_rep="-")
```
The same alignment can be used when ``others`` is a ``DataFrame``:

```python
f = d.loc[[3, 2, 1, 0], :]
s
f
s.str.cat(f, join="left", na_rep="-")
```
#### Concatenating a Series and many objects into a Series
Several array-like items (specifically: ``Series``, ``Index``, and 1-dimensional variants of ``np.ndarray``)
can be combined in a list-like container (including iterators, ``dict``-views, etc.).

```python
s
u
s.str.cat([u, u.to_numpy()], join="left")
```
All elements without an index (e.g. ``np.ndarray``) within the passed list-like must match in length to the calling ``Series`` (or ``Index``),
but ``Series`` and ``Index`` may have arbitrary length (as long as alignment is not disabled with ``join=None``):

```python
v
s.str.cat([v, u, u.to_numpy()], join="outer", na_rep="-")
```
If using ``join='right'`` on a list-like of ``others`` that contains different indexes,
the union of these indexes will be used as the basis for the final concatenation:

```python
u.loc[[3]]
v.loc[[-1, 0]]
s.str.cat([u.loc[[3]], v.loc[[-1, 0]]], join="right", na_rep="-")
```
## Indexing with ``.str``


You can use ``[]`` notation to directly index by position locations. If you index past the end
of the string, the result will be a ``NaN``.


```python
s = pd.Series(
    ["A", "B", "C", "Aaba", "Baca", np.nan, "CABA", "dog", "cat"], dtype="string"
)

s.str[0]
s.str[1]
```
## Extracting substrings


#### Extract first match in each subject (extract)
The ``extract`` method accepts a `regular expression
<https://docs.python.org/3/library/re.html>`__ with at least one
capture group.

Extracting a regular expression with more than one group returns a
DataFrame with one column per group.

```python
pd.Series(
    ["a1", "b2", "c3"],
    dtype="string",
).str.extract(r"([ab])(\d)", expand=False)
```
Elements that do not match return a row filled with ``NaN``. Thus, a
Series of messy strings can be "converted" into a like-indexed Series
or DataFrame of cleaned-up or more useful strings, without
necessitating ``get()`` to access tuples or ``re.match`` objects. The
dtype of the result is always object, even if no match is found and
the result only contains ``NaN``.

Named groups like

```python
pd.Series(["a1", "b2", "c3"], dtype="string").str.extract(
    r"(?P<letter>[ab])(?P<digit>\d)", expand=False
)
```
and optional groups like

```python
pd.Series(
    ["a1", "b2", "3"],
    dtype="string",
).str.extract(r"([ab])?(\d)", expand=False)
```
can also be used. Note that any capture group names in the regular
expression will be used for column names; otherwise capture group
numbers will be used.

Extracting a regular expression with one group returns a ``DataFrame``
with one column if ``expand=True``.

```python
pd.Series(["a1", "b2", "c3"], dtype="string").str.extract(r"[ab](\d)", expand=True)
```
It returns a Series if ``expand=False``.

```python
pd.Series(["a1", "b2", "c3"], dtype="string").str.extract(r"[ab](\d)", expand=False)
```
Calling on an ``Index`` with a regex with exactly one capture group
returns a ``DataFrame`` with one column if ``expand=True``.

```python
s = pd.Series(["a1", "b2", "c3"], ["A11", "B22", "C33"], dtype="string")
s
s.index.str.extract("(?P<letter>[a-zA-Z])", expand=True)
```
It returns an ``Index`` if ``expand=False``.

```python
s.index.str.extract("(?P<letter>[a-zA-Z])", expand=False)
```
Calling on an ``Index`` with a regex with more than one capture group
returns a ``DataFrame`` if ``expand=True``.

```python
s.index.str.extract("(?P<letter>[a-zA-Z])([0-9]+)", expand=True)
```
It raises ``ValueError`` if ``expand=False``.

```python
:okexcept:

 s.index.str.extract("(?P<letter>[a-zA-Z])([0-9]+)", expand=False)
```
The table below summarizes the behavior of ``extract(expand=False)``
(input subject in first column, number of groups in regex in
first row)

+--------+---------+------------+
|        | 1 group | >1 group   |
+--------+---------+------------+
| Index  | Index   | ValueError |
+--------+---------+------------+
| Series | Series  | DataFrame  |
+--------+---------+------------+

#### Extract all matches in each subject (extractall)


Unlike ``extract`` (which returns only the first match),

```python
s = pd.Series(["a1a2", "b1", "c1"], index=["A", "B", "C"], dtype="string")
s
two_groups = "(?P<letter>[a-z])(?P<digit>[0-9])"
s.str.extract(two_groups, expand=True)
[``
the ``extractall`` method returns every match. The result of
``extractall`` is always a ``DataFrame`` with a ``MultiIndex`` on its
rows. The last level of the ``MultiIndex`` is named ``match`` and
indicates the order in the subject.

```python
s.str.extractall(two_groups)
```
When each subject string in the Series has exactly one match,

```python
s = pd.Series(["a3", "b3", "c2"], dtype="string")
s
```
then ``extractall(pat).xs(0, level='match')`` gives the same result as
``extract(pat)``.

```python
extract_result = s.str.extract(two_groups, expand=True)
extract_result
extractall_result = s.str.extractall(two_groups)
extractall_result
extractall_result.xs(0, level="match")
```
``Index`` also supports ``.str.extractall``. It returns a ``DataFrame`` which has the
same result as a ``Series.str.extractall`` with a default index (starts from 0).

```python
pd.Index(["a1a2", "b1", "c1"]).str.extractall(two_groups)

pd.Series(["a1a2", "b1", "c1"], dtype="string").str.extractall(two_groups)
```
## Testing for strings that match or contain a pattern
You can check whether elements contain a pattern:

```python
pattern = r"[0-9][a-z]"
pd.Series(
    ["1", "2", "3a", "3b", "03c", "4dx"],
    dtype="string",
).str.contains(pattern)
```
Or whether elements match a pattern:

```python
pd.Series(
    ["1", "2", "3a", "3b", "03c", "4dx"],
    dtype="string",
).str.match(pattern)
```
```python
pd.Series(
    ["1", "2", "3a", "3b", "03c", "4dx"],
    dtype="string",
).str.fullmatch(pattern)
```
> **note.capitalize():**
    The distinction between ``match``, ``fullmatch``, and ``contains`` is strictness:
    ``fullmatch`` tests whether the entire string matches the regular expression;
    ``match`` tests whether there is a match of the regular expression that begins
    at the first character of the string; and ``contains`` tests whether there is
    a match of the regular expression at any position within the string.

    The corresponding functions in the ``re`` package for these three match modes are
    `re.fullmatch](https://docs.python.org/3/library/re.html#re.fullmatch),
    [re.match](https://docs.python.org/3/library/re.html#re.match), and
    [re.search](https://docs.python.org/3/library/re.html#re.search),
    respectively.

Methods like ``match``, ``fullmatch``, ``contains``, ``startswith``, and
``endswith`` take an extra ``na`` argument so missing values can be considered
True or False:

```python
s4 = pd.Series(
    ["A", "B", "C", "Aaba", "Baca", np.nan, "CABA", "dog", "cat"], dtype="string"
)
s4.str.contains("A", na=False)
```


## Creating indicator variables
You can extract dummy variables from string columns.
For example if they are separated by a ``'|'``:

```python
s = pd.Series(["a", "a|b", np.nan, "a|c"], dtype="string")
s.str.get_dummies(sep="|")
```
String ``Index`` also supports ``get_dummies`` which returns a ``MultiIndex``.

```python
idx = pd.Index(["a", "a|b", np.nan, "a|c"])
idx.str.get_dummies(sep="|")
```
See also `~pandas.get_dummies`.

## Method summary



    :header: "Method", "Description"
    :widths: 20, 80

    `~Series.str.cat`,Concatenate strings
    `~Series.str.split`,Split strings on delimiter
    `~Series.str.rsplit`,Split strings on delimiter working from the end of the string
    `~Series.str.get`,Index into each element (retrieve i-th element)
    `~Series.str.join`,Join strings in each element of the Series with passed separator
    `~Series.str.get_dummies`,Split strings on the delimiter returning DataFrame of dummy variables
    `~Series.str.contains`,Return boolean array if each string contains pattern/regex
    `~Series.str.replace`,Replace occurrences of pattern/regex/string with some other string or the return value of a callable given the occurrence
    `~Series.str.removeprefix`,Remove prefix from string i.e. only remove if string starts with prefix.
    `~Series.str.removesuffix`,Remove suffix from string i.e. only remove if string ends with suffix.
    `~Series.str.repeat`,Duplicate values (``s.str.repeat(3)`` equivalent to ``x * 3``)
    `~Series.str.pad`,Add whitespace to the sides of strings
    `~Series.str.center`,Equivalent to ``str.center``
    `~Series.str.ljust`,Equivalent to ``str.ljust``
    `~Series.str.rjust`,Equivalent to ``str.rjust``
    `~Series.str.zfill`,Equivalent to ``str.zfill``
    `~Series.str.wrap`,Split long strings into lines with length less than a given width
    `~Series.str.slice`,Slice each string in the Series
    `~Series.str.slice_replace`,Replace slice in each string with passed value
    `~Series.str.count`,Count occurrences of pattern
    `~Series.str.startswith`,Equivalent to ``str.startswith(pat)`` for each element
    `~Series.str.endswith`,Equivalent to ``str.endswith(pat)`` for each element
    `~Series.str.findall`,Compute list of all occurrences of pattern/regex for each string
    `~Series.str.match`,Call ``re.match`` on each element returning matched groups as list
    `~Series.str.extract`,Call ``re.search`` on each element returning DataFrame with one row for each element and one column for each regex capture group
    `~Series.str.extractall`,Call ``re.findall`` on each element returning DataFrame with one row for each match and one column for each regex capture group
    `~Series.str.len`,Compute string lengths
    `~Series.str.strip`,Equivalent to ``str.strip``
    `~Series.str.rstrip`,Equivalent to ``str.rstrip``
    `~Series.str.lstrip`,Equivalent to ``str.lstrip``
    `~Series.str.partition`,Equivalent to ``str.partition``
    `~Series.str.rpartition`,Equivalent to ``str.rpartition``
    `~Series.str.lower`,Equivalent to ``str.lower``
    `~Series.str.casefold`,Equivalent to ``str.casefold``
    `~Series.str.upper`,Equivalent to ``str.upper``
    `~Series.str.find`,Equivalent to ``str.find``
    `~Series.str.rfind`,Equivalent to ``str.rfind``
    `~Series.str.index`,Equivalent to ``str.index``
    `~Series.str.rindex`,Equivalent to ``str.rindex``
    `~Series.str.capitalize`,Equivalent to ``str.capitalize``
    `~Series.str.swapcase`,Equivalent to ``str.swapcase``
    `~Series.str.normalize`,Return Unicode normal form. Equivalent to ``unicodedata.normalize``
    `~Series.str.translate`,Equivalent to ``str.translate``
    `~Series.str.isalnum`,Equivalent to ``str.isalnum``
    `~Series.str.isalpha`,Equivalent to ``str.isalpha``
    `~Series.str.isdigit`,Equivalent to ``str.isdigit``
    `~Series.str.isspace`,Equivalent to ``str.isspace``
    `~Series.str.islower`,Equivalent to ``str.islower``
    `~Series.str.isupper`,Equivalent to ``str.isupper``
    `~Series.str.istitle`,Equivalent to ``str.istitle``
    `~Series.str.isnumeric`,Equivalent to ``str.isnumeric``
    `~Series.str.isdecimal`,Equivalent to ``str.isdecimal``

---

# Working with missing data
### Values considered "missing"
pandas uses different sentinel values to represent a missing (also referred to as NA)
depending on the data type.

``numpy.nan`` for NumPy data types. The disadvantage of using NumPy data types
is that the original data type will be coerced to ``np.float64`` or ``object``.

```python
pd.Series([1, 2], dtype=np.int64).reindex([0, 1, 2])
pd.Series([True, False], dtype=np.bool_).reindex([0, 1, 2])
```
`NaT` for NumPy ``np.datetime64``, ``np.timedelta64``, and `PeriodDtype`. For typing applications,
use `api.typing.NaTType`.

```python
pd.Series([1, 2], dtype=np.dtype("timedelta64[ns]")).reindex([0, 1, 2])
pd.Series([1, 2], dtype=np.dtype("datetime64[ns]")).reindex([0, 1, 2])
pd.Series(["2020", "2020"], dtype=pd.PeriodDtype("D")).reindex([0, 1, 2])
```
`NA` for `StringDtype`, `Int64Dtype` (and other bit widths),
`Float64Dtype` (and other bit widths), `BooleanDtype` and `ArrowDtype`.
These types will maintain the original data type of the data.
For typing applications, use `api.typing.NAType`.

```python
pd.Series([1, 2], dtype="Int64").reindex([0, 1, 2])
pd.Series([True, False], dtype="boolean[pyarrow]").reindex([0, 1, 2])
```
To detect these missing value, use the `isna` or `notna` methods.

```python
ser = pd.Series([pd.Timestamp("2020-01-01"), pd.NaT])
ser
pd.isna(ser)
```
> **note.capitalize():**
   `isna` or `notna` will also consider ``None`` a missing value.

   ```python
ser = pd.Series([1, None], dtype=object)
ser
pd.isna(ser)
```
> **warning.capitalize():**
   Equality comparisons between ``np.nan``, `NaT`, and `NA`
   do not act like ``None``

   ```python
None == None  # noqa: E711
   np.nan == np.nan
   pd.NaT == pd.NaT
   pd.NA == pd.NA

Therefore, an equality comparison between a `DataFrame` or `Series`
with one of these missing values does not provide the same information as
`isna` or `notna`.



   ser = pd.Series([True, None], dtype="boolean[pyarrow]")
   ser == pd.NA
   pd.isna(ser)
```


### `NA` semantics
> **warning.capitalize():**
   Experimental: the behaviour of `NA` can still change without warning.

Starting from pandas 1.0, an experimental `NA` value (singleton) is
available to represent scalar missing values. The goal of `NA` is provide a
"missing" indicator that can be used consistently across data types
(instead of ``np.nan``, ``None`` or ``pd.NaT`` depending on the data type).

For example, when having missing values in a `Series` with the nullable integer
dtype, it will use `NA`:

```python
s = pd.Series([1, 2, None], dtype="Int64")
s
s[2]
s[2] is pd.NA
```
Currently, pandas does not use those data types using `NA` by default in
a `DataFrame` or `Series`, so you need to specify
the dtype explicitly. An easy way to convert to those dtypes is explained in the
`conversion section <missing_data.NA.conversion>`.

## Propagation in arithmetic and comparison operations
In general, missing values *propagate* in operations involving `NA`. When
one of the operands is unknown, the outcome of the operation is also unknown.

For example, `NA` propagates in arithmetic operations, similarly to
``np.nan``:

```python
pd.NA + 1
"a" * pd.NA
```
There are a few special cases when the result is known, even when one of the
operands is ``NA``.

```python
pd.NA ** 0
1 ** pd.NA
```
In equality and comparison operations, `NA` also propagates. This deviates
from the behaviour of ``np.nan``, where comparisons with ``np.nan`` always
return ``False``.

```python
pd.NA == 1
pd.NA == pd.NA
pd.NA < 2.5
```
To check if a value is equal to `NA`, use `isna`

```python
pd.isna(pd.NA)
```
> **note.capitalize():**
   An exception on this basic propagation rule are *reductions* (such as the
   mean or the minimum), where pandas defaults to skipping missing values. See the
   `calculation section <missing_data.calculations>[ for more.

## Logical operations
For logical operations, `NA` follows the rules of the
`three-valued logic](https://en.wikipedia.org/wiki/Three-valued_logic)_ (or
*Kleene logic*, similarly to R, SQL and Julia). This logic means to only
propagate missing values when it is logically required.

For example, for the logical "or" operation (``|``), if one of the operands
is ``True``, we already know the result will be ``True``, regardless of the
other value (so regardless the missing value would be ``True`` or ``False``).
In this case, `NA` does not propagate:

```python
True | False
True | pd.NA
pd.NA | True
```
On the other hand, if one of the operands is ``False``, the result depends
on the value of the other operand. Therefore, in this case `NA`
propagates:

```python
False | True
False | False
False | pd.NA
```
The behaviour of the logical "and" operation (``&``) can be derived using
similar logic (where now `NA` will not propagate if one of the operands
is already ``False``):

```python
False & True
False & False
False & pd.NA
```
```python
True & True
True & False
True & pd.NA
```
## ``NA`` in a boolean context
Since the actual value of an NA is unknown, it is ambiguous to convert NA
to a boolean value.

```python
:okexcept:

bool(pd.NA)
```
This also means that `NA` cannot be used in a context where it is
evaluated to a boolean, such as ``if condition: ...`` where ``condition`` can
potentially be `NA`. In such cases, `isna` can be used to check
for `NA` or ``condition`` being `NA` can be avoided, for example by
filling missing values beforehand.

A similar situation occurs when using `Series` or `DataFrame` objects in ``if``
statements, see `gotchas.truth`.

## NumPy ufuncs
`pandas.NA` implements NumPy's ``__array_ufunc__`` protocol. Most ufuncs
work with ``NA``, and generally return ``NA``:

```python
np.log(pd.NA)
np.add(pd.NA, 1)
```
> **warning.capitalize():**
   Currently, ufuncs involving an ndarray and ``NA`` will return an
   object-dtype filled with NA values.

   ```python
a = np.array([1, 2, 3])
   np.greater(a, pd.NA)

The return type here may change to return a different array type
in the future.
```
See `dsintro.numpy_interop` for more on ufuncs.



#### Conversion
If you have a `DataFrame` or `Series` using ``np.nan``,
`DataFrame.convert_dtypes` and `Series.convert_dtypes`, respectively,
will convert your data to use the nullable data types supporting `NA`,
such as `Int64Dtype` or `ArrowDtype`. This is especially helpful after reading
in data sets from IO methods where data types were inferred.

```python
import io
data = io.StringIO("a,b\n,True\n2,")
df = pd.read_csv(data)
df.dtypes
df_conv = df.convert_dtypes()
df_conv
df_conv.dtypes
```


### Inserting missing data
You can insert missing values by simply assigning to a `Series` or `DataFrame`.
The missing value sentinel used will be chosen based on the dtype.

```python
ser = pd.Series([1., 2., 3.])
ser.loc[0] = None
ser

ser = pd.Series([pd.Timestamp("2021"), pd.Timestamp("2021")])
ser.iloc[0] = np.nan
ser

ser = pd.Series([True, False], dtype="boolean[pyarrow]")
ser.iloc[0] = None
ser
```
For ``object`` types, pandas will use the value given:

```python
s = pd.Series(["a", "b", "c"], dtype=object)
s.loc[0] = None
s.loc[1] = np.nan
s
```


### Calculations with missing data
Missing values propagate through arithmetic operations between pandas objects.

```python
ser1 = pd.Series([np.nan, np.nan, 2, 3])
ser2 = pd.Series([np.nan, 1, np.nan, 4])
ser1
ser2
ser1 + ser2
```
The descriptive statistics and computational methods discussed in the
`data structure overview <basics.stats>` (and listed here
 and `here <api.dataframe.stats>[) all
account for missing data.

When summing data, NA values or empty data will be treated as zero.

```python
pd.Series([np.nan]).sum()
pd.Series([], dtype="float64").sum()
```
When taking the product, NA values or empty data will be treated as 1.

```python
pd.Series([np.nan]).prod()
pd.Series([], dtype="float64").prod()
```
Cumulative methods like `~DataFrame.cumsum` and `~DataFrame.cumprod`
ignore NA values by default, but preserve them in the resulting array. To override
this behaviour and include NA values in the calculation, use ``skipna=False``.


```python
ser = pd.Series([1, np.nan, 3, np.nan])
ser
ser.cumsum()
ser.cumsum(skipna=False)
```


### Dropping missing data
`~DataFrame.dropna` drops rows or columns with missing data.

```python
df = pd.DataFrame([[np.nan, 1, 2], [1, 2, np.nan], [1, 2, 3]])
df
df.dropna()
df.dropna(axis=1)

ser = pd.Series([1, pd.NA], dtype="int64[pyarrow]")
ser.dropna()
```
### Filling missing data


## Filling by value
`~DataFrame.fillna` replaces NA values with non-NA data.

Replace NA with a scalar value

```python
data = {"np": [1.0, np.nan, np.nan, 2], "arrow": pd.array([1.0, pd.NA, pd.NA, 2], dtype="float64[pyarrow]")}
df = pd.DataFrame(data)
df
df.fillna(0)
```
When the data has object dtype, you can control what type of NA values are present.

```python
df = pd.DataFrame({"a": [pd.NA, np.nan, None]}, dtype=object)
df
df.fillna(None)
df.fillna(np.nan)
df.fillna(pd.NA)
```
However when the dtype is not object, these will all be replaced with the proper NA value for the dtype.

```python
data = {"np": [1.0, np.nan, np.nan, 2], "arrow": pd.array([1.0, pd.NA, pd.NA, 2], dtype="float64[pyarrow]")}
df = pd.DataFrame(data)
df
df.fillna(None)
df.fillna(np.nan)
df.fillna(pd.NA)
```
Fill gaps forward or backward

```python
df.ffill()
df.bfill()
```


Limit the number of NA values filled

```python
df.ffill(limit=1)
```
NA values can be replaced with corresponding value from a `Series` or `DataFrame`
where the index and column aligns between the original object and the filled object.

```python
dff = pd.DataFrame(np.arange(30, dtype=np.float64).reshape(10, 3), columns=list("ABC"))
dff.iloc[3:5, 0] = np.nan
dff.iloc[4:6, 1] = np.nan
dff.iloc[5:8, 2] = np.nan
dff
dff.fillna(dff.mean())
```
> **note.capitalize():**
   `DataFrame.where` can also be used to fill NA values. Same result as above.

   ```python
dff.where(pd.notna(dff), dff.mean(), axis="columns")
```


## Interpolation
`DataFrame.interpolate` and `Series.interpolate` fills NA values
using various interpolation methods.

```python
df = pd.DataFrame(
    {
        "A": [1, 2.1, np.nan, 4.7, 5.6, 6.8],
        "B": [0.25, np.nan, np.nan, 4, 12.2, 14.4],
    }
)
df
df.interpolate()

idx = pd.date_range("2020-01-01", periods=10, freq="D")
data = np.random.default_rng(2).integers(0, 10, 10).astype(np.float64)
ts = pd.Series(data, index=idx)
ts.iloc[[1, 2, 5, 6, 9]] = np.nan

ts
@savefig series_before_interpolate.png
ts.plot()
```
```python
ts.interpolate()
@savefig series_interpolate.png
ts.interpolate().plot()
```
Interpolation relative to a `Timestamp` in the `DatetimeIndex`
is available by setting ``method="time"``

```python
ts2 = ts.iloc[[0, 1, 3, 7, 9]]
ts2
ts2.interpolate()
ts2.interpolate(method="time")
```
For a floating-point index, use ``method='values'``:

```python
idx = [0.0, 1.0, 10.0]
ser = pd.Series([0.0, np.nan, 10.0], idx)
ser
ser.interpolate()
ser.interpolate(method="values")
```
If you have scipy_ installed, you can pass the name of a 1-d interpolation routine to ``method``.
as specified in the scipy interpolation documentation_ and reference guide_.
The appropriate interpolation method will depend on the data type.



   If you are dealing with a time series that is growing at an increasing rate,
   use ``method='barycentric'``.

   If you have values approximating a cumulative distribution function,
   use ``method='pchip'``.

   To fill missing values with goal of smooth plotting use ``method='akima'``.

   ```python
df = pd.DataFrame(
   {
      "A": [1, 2.1, np.nan, 4.7, 5.6, 6.8],
      "B": [0.25, np.nan, np.nan, 4, 12.2, 14.4],
   }
)
df
df.interpolate(method="barycentric")
df.interpolate(method="pchip")
df.interpolate(method="akima")
```
When interpolating via a polynomial or spline approximation, you must also specify
the degree or order of the approximation:

```python
df.interpolate(method="spline", order=2)
df.interpolate(method="polynomial", order=2)
```
Comparing several methods.

```python
np.random.seed(2)

ser = pd.Series(np.arange(1, 10.1, 0.25) ** 2 + np.random.randn(37))
missing = np.array([4, 13, 14, 15, 16, 17, 18, 20, 29])
ser.iloc[missing] = np.nan
methods = ["linear", "quadratic", "cubic"]

df = pd.DataFrame({m: ser.interpolate(method=m) for m in methods})
@savefig compare_interpolations.png
df.plot()
```
Interpolating new observations from expanding data with `Series.reindex`.

```python
ser = pd.Series(np.sort(np.random.uniform(size=100)))

# interpolate at new_index
new_index = ser.index.union(pd.Index([49.25, 49.5, 49.75, 50.25, 50.5, 50.75]))
interp_s = ser.reindex(new_index).interpolate(method="pchip")
interp_s.loc[49:51]
```
.. _scipy: https://scipy.org/
.. _documentation: https://docs.scipy.org/doc/scipy/reference/interpolate.html#univariate-interpolation
.. _guide: https://docs.scipy.org/doc/scipy/tutorial/interpolate.html



#### Interpolation limits
`~DataFrame.interpolate` accepts a ``limit`` keyword
argument to limit the number of consecutive ``NaN`` values
filled since the last valid observation

```python
ser = pd.Series([np.nan, np.nan, 5, np.nan, np.nan, np.nan, 13, np.nan, np.nan])
ser
ser.interpolate()
ser.interpolate(limit=1)
```
By default, ``NaN`` values are filled in a ``forward`` direction. Use
``limit_direction`` parameter to fill ``backward`` or from ``both`` directions.

```python
ser.interpolate(limit=1, limit_direction="backward")
ser.interpolate(limit=1, limit_direction="both")
ser.interpolate(limit_direction="both")
```
By default, ``NaN`` values are filled whether they are surrounded by
existing valid values or outside existing valid values. The ``limit_area``
parameter restricts filling to either inside or outside values.

```python
# fill one consecutive inside value in both directions
ser.interpolate(limit_direction="both", limit_area="inside", limit=1)

# fill all consecutive outside values backward
ser.interpolate(limit_direction="backward", limit_area="outside")

# fill all consecutive outside values in both directions
ser.interpolate(limit_direction="both", limit_area="outside")
```


## Replacing values
`Series.replace` and `DataFrame.replace` can be used similar to
`Series.fillna` and `DataFrame.fillna` to replace or insert missing values.

```python
df = pd.DataFrame(np.eye(3))
df
df_missing = df.replace(0, np.nan)
df_missing
df_filled = df_missing.replace(np.nan, 2)
df_filled
```
Replacing more than one value is possible by passing a list.

```python
df_filled.replace([1, 44], [2, 28])
```
Replacing using a mapping dict.

```python
df_filled.replace({1: 44, 2: 28})
```


#### Regular expression replacement
> **note.capitalize():**
   Python strings prefixed with the ``r`` character such as ``r'hello world'``
   are `"raw" strings](https://docs.python.org/3/reference/lexical_analysis.html#string-and-bytes-literals).
   They have different semantics regarding backslashes than strings without this prefix.
   Backslashes in raw strings  will be interpreted as an escaped backslash, e.g., ``r'\' == '\\'``.

Replace the '.' with ``NaN``

```python
d = {"a": list(range(4)), "b": list("ab.."), "c": ["a", "b", np.nan, "d"]}
df = pd.DataFrame(d)
df.replace(".", np.nan)
```
Replace the '.' with ``NaN`` with regular expression that removes surrounding whitespace

```python
df.replace(r"\s*\.\s*", np.nan, regex=True)
```
Replace with a list of regexes.

```python
df.replace([r"\.", r"(a)"], ["dot", r"\1stuff"], regex=True)
```
Replace with a regex in a mapping dict.

```python
df.replace({"b": r"\s*\.\s*"}, {"b": np.nan}, regex=True)
```
Pass nested dictionaries of regular expressions that use the ``regex`` keyword.

```python
df.replace({"b": {"b": r""}}, regex=True)
df.replace(regex={"b": {r"\s*\.\s*": np.nan}})
df.replace({"b": r"\s*(\.)\s*"}, {"b": r"\1ty"}, regex=True)
```
Pass a list of regular expressions that will replace matches with a scalar.

```python
df.replace([r"\s*\.\s*", r"a|b"], "placeholder", regex=True)
```
All of the regular expression examples can also be passed with the
``to_replace`` argument as the ``regex`` argument. In this case the ``value``
argument must be passed explicitly by name or ``regex`` must be a nested
dictionary.

```python
df.replace(regex=[r"\s*\.\s*", r"a|b"], value="placeholder")
```
> **note.capitalize():**
   A regular expression object from ``re.compile`` is a valid input as well.

---

# Duplicate Labels
`Index` objects are not required to be unique; you can have duplicate row
or column labels. This may be a bit confusing at first. If you're familiar with
SQL, you know that row labels are similar to a primary key on a table, and you
would never want duplicates in a SQL table. But one of pandas' roles is to clean
messy, real-world data before it goes to some downstream system. And real-world
data has duplicates, even in fields that are supposed to be unique.

This section describes how duplicate labels change the behavior of certain
operations, and how prevent duplicates from arising during operations, or to
detect them if they do.

```python
import pandas as pd
import numpy as np
```
### Consequences of Duplicate Labels
Some pandas methods (`Series.reindex` for example) just don't work with
duplicates present. The output can't be determined, and so pandas raises.

```python
:okexcept:
:okwarning:

s1 = pd.Series([0, 1, 2], index=["a", "b", "b"])
s1.reindex(["a", "b", "c"])
```
Other methods, like indexing, can give very surprising results. Typically
indexing with a scalar will *reduce dimensionality*. Slicing a ``DataFrame``
with a scalar will return a ``Series``. Slicing a ``Series`` with a scalar will
return a scalar. But with duplicates, this isn't the case.

```python
df1 = pd.DataFrame([[0, 1, 2], [3, 4, 5]], columns=["A", "A", "B"])
df1
```
We have duplicates in the columns. If we slice ``'B'``, we get back a ``Series``

```python
df1["B"]  # a series
```
But slicing ``'A'`` returns a ``DataFrame``


```python
df1["A"]  # a DataFrame
```
This applies to row labels as well

```python
df2 = pd.DataFrame({"A": [0, 1, 2]}, index=["a", "a", "b"])
df2
df2.loc["b", "A"]  # a scalar
df2.loc["a", "A"]  # a Series
```
### Duplicate Label Detection
You can check whether an `Index` (storing the row or column labels) is
unique with `Index.is_unique`:

```python
df2
df2.index.is_unique
df2.columns.is_unique
```
> **note.capitalize():**
   Checking whether an index is unique is somewhat expensive for large datasets.
   pandas does cache this result, so re-checking on the same index is very fast.

`Index.duplicated` will return a boolean ndarray indicating whether a
label is repeated.

```python
df2.index.duplicated()
```
Which can be used as a boolean filter to drop duplicate rows.

```python
df2.loc[~df2.index.duplicated(), :]
```
If you need additional logic to handle duplicate labels, rather than just
dropping the repeats, using `~DataFrame.groupby` on the index is a common
trick. For example, we'll resolve duplicates by taking the average of all rows
with the same label.

```python
df2.groupby(level=0).mean()
```


### Disallowing Duplicate Labels


As noted above, handling duplicates is an important feature when reading in raw
data. That said, you may want to avoid introducing duplicates as part of a data
processing pipeline (from methods like `pandas.concat`,
`~DataFrame.rename`, etc.). Both `Series` and `DataFrame`
*disallow* duplicate labels by calling ``.set_flags(allows_duplicate_labels=False)``.
(the default is to allow them). If there are duplicate labels, an exception
will be raised.

```python
:okexcept:

pd.Series([0, 1, 2], index=["a", "b", "b"]).set_flags(allows_duplicate_labels=False)
```
This applies to both row and column labels for a `DataFrame`

```python
:okexcept:

pd.DataFrame([[0, 1, 2], [3, 4, 5]], columns=["A", "B", "C"],).set_flags(
    allows_duplicate_labels=False
)
```
This attribute can be checked or set with `~DataFrame.flags.allows_duplicate_labels`,
which indicates whether that object can have duplicate labels.

```python
df = pd.DataFrame({"A": [0, 1, 2, 3]}, index=["x", "y", "X", "Y"]).set_flags(
    allows_duplicate_labels=False
)
df
df.flags.allows_duplicate_labels
```
`DataFrame.set_flags` can be used to return a new ``DataFrame`` with attributes
like ``allows_duplicate_labels`` set to some value

```python
df2 = df.set_flags(allows_duplicate_labels=True)
df2.flags.allows_duplicate_labels
```
The new ``DataFrame`` returned is a view on the same data as the old ``DataFrame``.
Or the property can just be set directly on the same object


```python
df2.flags.allows_duplicate_labels = False
df2.flags.allows_duplicate_labels
```
When processing raw, messy data you might initially read in the messy data
(which potentially has duplicate labels), deduplicate, and then disallow duplicates
going forward, to ensure that your data pipeline doesn't introduce duplicates.


```python
>>> raw = pd.read_csv("...")
>>> deduplicated = raw.groupby(level=0).first()  # remove duplicates
>>> deduplicated.flags.allows_duplicate_labels = False  # disallow going forward
```
Setting ``allows_duplicate_labels=False`` on a ``Series`` or ``DataFrame`` with duplicate
labels or performing an operation that introduces duplicate labels on a ``Series`` or
``DataFrame`` that disallows duplicates will raise an
`errors.DuplicateLabelError`.

```python
:okexcept:

df.rename(str.upper)
```
This error message contains the labels that are duplicated, and the numeric positions
of all the duplicates (including the "original") in the ``Series`` or ``DataFrame``

#### Duplicate Label Propagation
In general, disallowing duplicates is "sticky". It's preserved through
operations.

```python
:okexcept:

s1 = pd.Series(0, index=["a", "b"]).set_flags(allows_duplicate_labels=False)
s1
s1.head().rename({"a": "b"})
```
> **warning.capitalize():**
   This is an experimental feature. Currently, many methods fail to
   propagate the ``allows_duplicate_labels`` value. In future versions
   it is expected that every method taking or returning one or more
   DataFrame or Series objects will propagate ``allows_duplicate_labels``.

---

# Categorical data
This is an introduction to pandas categorical data type, including a short comparison
with R's ``factor``.

``Categoricals`` are a pandas data type corresponding to categorical variables in
statistics. A categorical variable takes on a limited, and usually fixed,
number of possible values (``categories``; ``levels`` in R). Examples are gender,
social class, blood type, country affiliation, observation time or rating via
Likert scales.

In contrast to statistical categorical variables, categorical data might have an order (e.g.
'strongly agree' vs 'agree' or 'first observation' vs. 'second observation'), but numerical
operations (additions, divisions, ...) are not possible.

All values of categorical data are either in ``categories`` or ``np.nan``. Order is defined by
the order of ``categories``, not lexical order of the values. Internally, the data structure
consists of a ``categories`` array and an integer array of ``codes`` which point to the real value in
the ``categories`` array.

The categorical data type is useful in the following cases:

* A string variable consisting of only a few different values. Converting such a string
  variable to a categorical variable will save some memory, see `here <categorical.memory>`.
* The lexical order of a variable is not the same as the logical order ("one", "two", "three").
  By converting to a categorical and specifying an order on the categories, sorting and
  min/max will use the logical order instead of the lexical order, see `here <categorical.sort>`.
* As a signal to other Python libraries that this column should be treated as a categorical
  variable (e.g. to use suitable statistical methods or plot types).

See also the `API docs on categoricals<api.arrays.categorical>`.



## Object creation
### Series creation
Categorical ``Series`` or columns in a ``DataFrame`` can be created in several ways:

By specifying ``dtype="category"`` when constructing a ``Series``:

```python
s = pd.Series(["a", "b", "c", "a"], dtype="category")
s
```
By converting an existing ``Series`` or column to a ``category`` dtype:

```python
df = pd.DataFrame({"A": ["a", "b", "c", "a"]})
df["B"] = df["A"].astype("category")
df
```
By using special functions, such as `~pandas.cut`, which groups data into
discrete bins. See the `example on tiling <reshaping.tile.cut>` in the docs.

```python
df = pd.DataFrame({"value": np.random.randint(0, 100, 20)})
labels = ["{0} - {1}".format(i, i + 9) for i in range(0, 100, 10)]

df["group"] = pd.cut(df.value, range(0, 105, 10), right=False, labels=labels)
df.head(10)
```
By passing a `pandas.Categorical` object to a ``Series`` or assigning it to a ``DataFrame``.

```python
raw_cat = pd.Categorical(
    [None, "b", "c", None], categories=["b", "c", "d"], ordered=False
)
s = pd.Series(raw_cat)
s
df = pd.DataFrame({"A": ["a", "b", "c", "a"]})
df["B"] = raw_cat
df
```
Categorical data has a specific ``category`` `dtype <basics.dtypes>`:

```python
df.dtypes
```
### DataFrame creation
Similar to the previous section where a single column was converted to categorical, all columns in a
``DataFrame`` can be batch converted to categorical either during or after construction.

This can be done during construction by specifying ``dtype="category"`` in the ``DataFrame`` constructor:

```python
df = pd.DataFrame({"A": list("abca"), "B": list("bccd")}, dtype="category")
df.dtypes
```
Note that the categories present in each column differ; the conversion is done column by column, so
only labels present in a given column are categories:

```python
df["A"]
df["B"]
```
Analogously, all columns in an existing ``DataFrame`` can be batch converted using `DataFrame.astype`:

```python
df = pd.DataFrame({"A": list("abca"), "B": list("bccd")})
df_cat = df.astype("category")
df_cat.dtypes
```
This conversion is likewise done column by column:

```python
df_cat["A"]
df_cat["B"]
```
### Controlling behavior
In the examples above where we passed ``dtype='category'``, we used the default
behavior:

1. Categories are inferred from the data.
2. Categories are unordered.

To control those behaviors, instead of passing ``'category'``, use an instance
of `~pandas.api.types.CategoricalDtype`.

```python
from pandas.api.types import CategoricalDtype

s = pd.Series([None, "b", "c", None])
cat_type = CategoricalDtype(categories=["b", "c", "d"], ordered=True)
s_cat = s.astype(cat_type)
s_cat
```
Similarly, a ``CategoricalDtype`` can be used with a ``DataFrame`` to ensure that categories
are consistent among all columns.

```python
from pandas.api.types import CategoricalDtype

df = pd.DataFrame({"A": list("abca"), "B": list("bccd")})
cat_type = CategoricalDtype(categories=list("abcd"), ordered=True)
df_cat = df.astype(cat_type)
df_cat["A"]
df_cat["B"]
```
> **note.capitalize():**
    To perform table-wise conversion, where all labels in the entire ``DataFrame`` are used as
    categories for each column, the ``categories`` parameter can be determined programmatically by
    ``categories = pd.unique(df.to_numpy().ravel())``.

If you already have ``codes`` and ``categories``, you can use the
`~pandas.Categorical.from_codes` constructor to save the factorize step
during normal constructor mode:

```python
splitter = np.random.choice([0, 1], 5, p=[0.5, 0.5])
s = pd.Series(pd.Categorical.from_codes(splitter, categories=["train", "test"]))
```
### Regaining original data
To get back to the original ``Series`` or NumPy array, use
``Series.astype(original_dtype)`` or ``np.asarray(categorical)``:

```python
s = pd.Series(["a", "b", "c", "a"])
s
s2 = s.astype("category")
s2
s2.astype(str)
np.asarray(s2)
```
> **note.capitalize():**
    In contrast to R's ``factor`` function, categorical data is not converting input values to
    strings; categories will end up the same data type as the original values.

> **note.capitalize():**
    In contrast to R's ``factor`` function, there is currently no way to assign/change labels at
    creation time. Use ``categories`` to change the categories after creation time.



## CategoricalDtype
A categorical's type is fully described by

1. ``categories``: a sequence of unique values and no missing values
2. ``ordered``: a boolean

This information can be stored in a `~pandas.api.types.CategoricalDtype`.
The ``categories`` argument is optional, which implies that the actual categories
should be inferred from whatever is present in the data when the
`pandas.Categorical` is created. The categories are assumed to be unordered
by default.

```python
from pandas.api.types import CategoricalDtype

CategoricalDtype(["a", "b", "c"])
CategoricalDtype(["a", "b", "c"], ordered=True)
CategoricalDtype()
```
A `~pandas.api.types.CategoricalDtype` can be used in any place pandas
expects a ``dtype``. For example `pandas.read_csv`,
`pandas.DataFrame.astype`, or in the ``Series`` constructor.

> **note.capitalize():**
    As a convenience, you can use the string ``'category'`` in place of a
    `~pandas.api.types.CategoricalDtype` when you want the default behavior of
    the categories being unordered, and equal to the set values present in the
    array. In other words, ``dtype='category'`` is equivalent to
    ``dtype=CategoricalDtype()``.

### Equality semantics
Two instances of `~pandas.api.types.CategoricalDtype` compare equal
whenever they have the same categories and order. When comparing two
unordered categoricals, the order of the ``categories`` is not considered. Note
that categories with different dtypes are not the same.

```python
c1 = CategoricalDtype(["a", "b", "c"], ordered=False)

# Equal, since order is not considered when ordered=False
c1 == CategoricalDtype(["b", "c", "a"], ordered=False)

# Unequal, since the second CategoricalDtype is ordered
c1 == CategoricalDtype(["a", "b", "c"], ordered=True)
```
All instances of ``CategoricalDtype`` compare equal to the string ``'category'``.

```python
c1 == "category"
```
Notice that the ``categories_dtype`` should be considered, especially when comparing with
two empty ``CategoricalDtype`` instances.

```python
c2 = pd.Categorical(np.array([], dtype=object))
c3 = pd.Categorical(np.array([], dtype=float))

c2.dtype == c3.dtype
```
## Description
Using `~DataFrame.describe` on categorical data will produce similar
output to a ``Series`` or ``DataFrame`` of type ``string``.

```python
cat = pd.Categorical(["a", "c", "c", np.nan], categories=["b", "a", "c"])
df = pd.DataFrame({"cat": cat, "s": ["a", "c", "c", np.nan]})
df.describe()
df["cat"].describe()
```


## Working with categories
Categorical data has a ``categories`` and a ``ordered`` property, which list their
possible values and whether the ordering matters or not. These properties are
exposed as ``s.cat.categories`` and ``s.cat.ordered``. If you don't manually
specify categories and ordering, they are inferred from the passed arguments.

```python
s = pd.Series(["a", "b", "c", "a"], dtype="category")
s.cat.categories
s.cat.ordered
```
It's also possible to pass in the categories in a specific order:

```python
s = pd.Series(pd.Categorical(["a", "b", "c", "a"], categories=["c", "b", "a"]))
s.cat.categories
s.cat.ordered
```
> **note.capitalize():**
    New categorical data are **not** automatically ordered. You must explicitly
    pass ``ordered=True`` to indicate an ordered ``Categorical``.


> **note.capitalize():**
    The result of `~Series.unique` is not always the same as ``Series.cat.categories``,
    because ``Series.unique()`` has a couple of guarantees, namely that it returns categories
    in the order of appearance, and it only includes values that are actually present.

    ```python
s = pd.Series(list("babc")).astype(CategoricalDtype(list("abcd")))
s

# categories
s.cat.categories

# uniques
s.unique()
```
### Renaming categories
Renaming categories is done by using the
`~pandas.Categorical.rename_categories` method:


```python
s = pd.Series(["a", "b", "c", "a"], dtype="category")
s
new_categories = ["Group %s" % g for g in s.cat.categories]
s = s.cat.rename_categories(new_categories)
s
# You can also pass a dict-like object to map the renaming
s = s.cat.rename_categories({1: "x", 2: "y", 3: "z"})
s
```
> **note.capitalize():**
    In contrast to R's ``factor``, categorical data can have categories of other types than string.

Categories must be unique or a ``ValueError`` is raised:

```python
try:
    s = s.cat.rename_categories([1, 1, 1])
except ValueError as e:
    print("ValueError:", str(e))
```
Categories must also not be ``NaN`` or a ``ValueError`` is raised:

```python
try:
    s = s.cat.rename_categories([1, 2, np.nan])
except ValueError as e:
    print("ValueError:", str(e))
```
### Appending new categories
Appending categories can be done by using the
`~pandas.Categorical.add_categories` method:

```python
s = s.cat.add_categories([4])
s.cat.categories
s
```
### Removing categories
Removing categories can be done by using the
`~pandas.Categorical.remove_categories` method. Values which are removed
are replaced by ``np.nan``.:

```python
s = s.cat.remove_categories([4])
s
```
### Removing unused categories
Removing unused categories can also be done:

```python
s = pd.Series(pd.Categorical(["a", "b", "a"], categories=["a", "b", "c", "d"]))
s
s.cat.remove_unused_categories()
```
### Setting categories
If you want to do remove and add new categories in one step (which has some
speed advantage), or simply set the categories to a predefined scale,
use `~pandas.Categorical.set_categories`.


```python
s = pd.Series(["one", "two", "four", "-"], dtype="category")
s
s = s.cat.set_categories(["one", "two", "three", "four"])
s
```
> **note.capitalize():**
    Be aware that `Categorical.set_categories` cannot know whether some category is omitted
    intentionally or because it is misspelled or (under Python3) due to a type difference (e.g.,
    NumPy S1 dtype and Python strings). This can result in surprising behaviour!

## Sorting and order


If categorical data is ordered (``s.cat.ordered == True``), then the order of the categories has a
meaning and certain operations are possible. If the categorical is unordered, ``.min()/.max()`` will raise a ``TypeError``.

```python
s = pd.Series(pd.Categorical(["a", "b", "c", "a"], ordered=False))
s = s.sort_values()
s = pd.Series(["a", "b", "c", "a"]).astype(CategoricalDtype(ordered=True))
s = s.sort_values()
s
s.min(), s.max()
```
You can set categorical data to be ordered by using ``as_ordered()`` or unordered by using ``as_unordered()``. These will by
default return a *new* object.

```python
s.cat.as_ordered()
s.cat.as_unordered()
```
Sorting will use the order defined by categories, not any lexical order present on the data type.
This is even true for strings and numeric data:

```python
s = pd.Series([1, 2, 3, 1], dtype="category")
s = s.cat.set_categories([2, 3, 1], ordered=True)
s
s = s.sort_values()
s
s.min(), s.max()
```
### Reordering
Reordering the categories is possible via the `Categorical.reorder_categories` and
the `Categorical.set_categories` methods. For `Categorical.reorder_categories`, all
old categories must be included in the new categories and no new categories are allowed. This will
necessarily make the sort order the same as the categories order.

```python
s = pd.Series([1, 2, 3, 1], dtype="category")
s = s.cat.reorder_categories([2, 3, 1], ordered=True)
s
s = s.sort_values()
s
s.min(), s.max()
```
> **note.capitalize():**
    Note the difference between assigning new categories and reordering the categories: the first
    renames categories and therefore the individual values in the ``Series``, but if the first
    position was sorted last, the renamed value will still be sorted last. Reordering means that the
    way values are sorted is different afterwards, but not that individual values in the
    ``Series`` are changed.

> **note.capitalize():**
    If the ``Categorical`` is not ordered, `Series.min` and `Series.max` will raise
    ``TypeError``. Numeric operations like ``+``, ``-``, ``*``, ``/`` and operations based on them
    (e.g. `Series.median`, which would need to compute the mean between two values if the length
    of an array is even) do not work and raise a ``TypeError``.

### Multi column sorting
A categorical dtyped column will participate in a multi-column sort in a similar manner to other columns.
The ordering of the categorical is determined by the ``categories`` of that column.

```python
dfs = pd.DataFrame(
    {
        "A": pd.Categorical(
            list("bbeebbaa"),
            categories=["e", "a", "b"],
            ordered=True,
        ),
        "B": [1, 2, 1, 2, 2, 1, 2, 1],
    }
)
dfs.sort_values(by=["A", "B"])
```
Reordering the ``categories`` changes a future sort.

```python
dfs["A"] = dfs["A"].cat.reorder_categories(["a", "b", "e"])
dfs.sort_values(by=["A", "B"])
```
## Comparisons
Comparing categorical data with other objects is possible in three cases:

* Comparing equality (``==`` and ``!=``) to a list-like object (list, Series, array,
  ...) of the same length as the categorical data.
* All comparisons (``==``, ``!=``, ``>``, ``>=``, ``<``, and ``<=``) of categorical data to
  another categorical Series, when ``ordered==True`` and the ``categories`` are the same.
* All comparisons of a categorical data to a scalar.

All other comparisons, especially "non-equality" comparisons of two categoricals with different
categories or a categorical with any list-like object, will raise a ``TypeError``.

> **note.capitalize():**
    Any "non-equality" comparisons of categorical data with a ``Series``, ``np.array``, ``list`` or
    categorical data with different categories or ordering will raise a ``TypeError`` because custom
    categories ordering could be interpreted in two ways: one with taking into account the
    ordering and one without.

```python
cat = pd.Series([1, 2, 3]).astype(CategoricalDtype([3, 2, 1], ordered=True))
cat_base = pd.Series([2, 2, 2]).astype(CategoricalDtype([3, 2, 1], ordered=True))
cat_base2 = pd.Series([2, 2, 2]).astype(CategoricalDtype(ordered=True))

cat
cat_base
cat_base2
```
Comparing to a categorical with the same categories and ordering or to a scalar works:

```python
cat > cat_base
cat > 2
```
Equality comparisons work with any list-like object of same length and scalars:

```python
cat == cat_base
cat == np.array([1, 2, 3])
cat == 2
```
This doesn't work because the categories are not the same:

```python
try:
    cat > cat_base2
except TypeError as e:
    print("TypeError:", str(e))
```
If you want to do a "non-equality" comparison of a categorical series with a list-like object
which is not categorical data, you need to be explicit and convert the categorical data back to
the original values:

```python
base = np.array([1, 2, 3])

try:
    cat > base
except TypeError as e:
    print("TypeError:", str(e))

np.asarray(cat) > base
```
When you compare two unordered categoricals with the same categories, the order is not considered:

```python
c1 = pd.Categorical(["a", "b"], categories=["a", "b"], ordered=False)
c2 = pd.Categorical(["a", "b"], categories=["b", "a"], ordered=False)
c1 == c2
```
## Operations
Apart from `Series.min`, `Series.max` and `Series.mode`, the
following operations are possible with categorical data:

``Series`` methods like `Series.value_counts` will use all categories,
even if some categories are not present in the data:

```python
s = pd.Series(pd.Categorical(["a", "b", "c", "c"], categories=["c", "a", "b", "d"]))
s.value_counts()
```
``DataFrame`` methods like `DataFrame.sum` also show "unused" categories when ``observed=False``.

```python
columns = pd.Categorical(
    ["One", "One", "Two"], categories=["One", "Two", "Three"], ordered=True
)
df = pd.DataFrame(
    data=[[1, 2, 3], [4, 5, 6]],
    columns=pd.MultiIndex.from_arrays([["A", "B", "B"], columns]),
).T
df.groupby(level=1, observed=False).sum()
```
Groupby will also show "unused" categories when ``observed=False``:

```python
cats = pd.Categorical(
    ["a", "b", "b", "b", "c", "c", "c"], categories=["a", "b", "c", "d"]
)
df = pd.DataFrame({"cats": cats, "values": [1, 2, 2, 2, 3, 4, 5]})
df.groupby("cats", observed=False).mean()

cats2 = pd.Categorical(["a", "a", "b", "b"], categories=["a", "b", "c"])
df2 = pd.DataFrame(
    {
        "cats": cats2,
        "B": ["c", "d", "c", "d"],
        "values": [1, 2, 3, 4],
    }
)
df2.groupby(["cats", "B"], observed=False).mean()
```
Pivot tables:

```python
raw_cat = pd.Categorical(["a", "a", "b", "b"], categories=["a", "b", "c"])
df = pd.DataFrame({"A": raw_cat, "B": ["c", "d", "c", "d"], "values": [1, 2, 3, 4]})
pd.pivot_table(df, values="values", index=["A", "B"], observed=False)
```
## Data munging
The optimized pandas data access methods  ``.loc``, ``.iloc``, ``.at``, and ``.iat``,
work as normal. The only difference is the return type (for getting) and
that only values already in ``categories`` can be assigned.

### Getting
If the slicing operation returns either a ``DataFrame`` or a column of type
``Series``, the ``category`` dtype is preserved.

```python
idx = pd.Index(["h", "i", "j", "k", "l", "m", "n"])
cats = pd.Series(["a", "b", "b", "b", "c", "c", "c"], dtype="category", index=idx)
values = [1, 2, 2, 2, 3, 4, 5]
df = pd.DataFrame({"cats": cats, "values": values}, index=idx)
df.iloc[2:4, :]
df.iloc[2:4, :].dtypes
df.loc["h":"j", "cats"]
df[df["cats"] == "b"]
```
An example where the category type is not preserved is if you take one single
row: the resulting ``Series`` is of dtype ``object``:

```python
# get the complete "h" row as a Series
df.loc["h", :]
```
Returning a single item from categorical data will also return the value, not a categorical
of length "1".

```python
df.iat[0, 0]
df["cats"] = df["cats"].cat.rename_categories(["x", "y", "z"])
df.at["h", "cats"]  # returns a string
```
> **note.capitalize():**
    The is in contrast to R's ``factor`` function, where ``factor(c(1,2,3))[1]``
    returns a single value ``factor``.

To get a single value ``Series`` of type ``category``, you pass in a list with
a single value:

```python
df.loc[["h"], "cats"]
```
### String and datetime accessors
The accessors  ``.dt`` and ``.str`` will work if the ``s.cat.categories`` are of
an appropriate type:


```python
str_s = pd.Series(list("aabb"))
str_cat = str_s.astype("category")
str_cat
str_cat.str.contains("a")

date_s = pd.Series(pd.date_range("1/1/2015", periods=5))
date_cat = date_s.astype("category")
date_cat
date_cat.dt.day
```
> **note.capitalize():**
    The returned ``Series`` (or ``DataFrame``) is of the same type as if you used the
    ``.str.<method>`` / ``.dt.<method>`` on a ``Series`` of that type (and not of
    type ``category``!).

That means, that the returned values from methods and properties on the accessors of a
``Series`` and the returned values from methods and properties on the accessors of this
``Series`` transformed to one of type ``category`` will be equal:

```python
ret_s = str_s.str.contains("a")
ret_cat = str_cat.str.contains("a")
ret_s.dtype == ret_cat.dtype
ret_s == ret_cat
```
> **note.capitalize():**
    The work is done on the ``categories`` and then a new ``Series`` is constructed. This has
    some performance implication if you have a ``Series`` of type string, where lots of elements
    are repeated (i.e. the number of unique elements in the ``Series`` is a lot smaller than the
    length of the ``Series``). In this case it can be faster to convert the original ``Series``
    to one of type ``category`` and use ``.str.<method>`` or ``.dt.<property>`` on that.

### Setting
Setting values in a categorical column (or ``Series``) works as long as the
value is included in the ``categories``:

```python
idx = pd.Index(["h", "i", "j", "k", "l", "m", "n"])
cats = pd.Categorical(["a", "a", "a", "a", "a", "a", "a"], categories=["a", "b"])
values = [1, 1, 1, 1, 1, 1, 1]
df = pd.DataFrame({"cats": cats, "values": values}, index=idx)

df.iloc[2:4, :] = [["b", 2], ["b", 2]]
df
try:
    df.iloc[2:4, :] = [["c", 3], ["c", 3]]
except TypeError as e:
    print("TypeError:", str(e))
```
Setting values by assigning categorical data will also check that the ``categories`` match:

```python
df.loc["j":"k", "cats"] = pd.Categorical(["a", "a"], categories=["a", "b"])
df
try:
    df.loc["j":"k", "cats"] = pd.Categorical(["b", "b"], categories=["a", "b", "c"])
except TypeError as e:
    print("TypeError:", str(e))
```
Assigning a ``Categorical`` to parts of a column of other types will use the values:

```python
:okwarning:

df = pd.DataFrame({"a": [1, 1, 1, 1, 1], "b": ["a", "a", "a", "a", "a"]})
df.loc[1:2, "a"] = pd.Categorical([2, 2], categories=[2, 3])
df.loc[2:3, "b"] = pd.Categorical(["b", "b"], categories=["a", "b"])
df
df.dtypes
```



### Merging / concatenation
By default, combining ``Series`` or ``DataFrames`` which contain the same
categories results in ``category`` dtype, otherwise results will depend on the
dtype of the underlying categories. Merges that result in non-categorical
dtypes will likely have higher memory usage. Use ``.astype`` or
``union_categoricals`` to ensure ``category`` results.

```python
from pandas.api.types import union_categoricals

# same categories
s1 = pd.Series(["a", "b"], dtype="category")
s2 = pd.Series(["a", "b", "a"], dtype="category")
pd.concat([s1, s2])

# different categories
s3 = pd.Series(["b", "c"], dtype="category")
pd.concat([s1, s3])

# Output dtype is inferred based on categories values
int_cats = pd.Series([1, 2], dtype="category")
float_cats = pd.Series([3.0, 4.0], dtype="category")
pd.concat([int_cats, float_cats])

pd.concat([s1, s3]).astype("category")
union_categoricals([s1.array, s3.array])
```
The following table summarizes the results of merging ``Categoricals``:

+-------------------+------------------------+----------------------+-----------------------------+
| arg1              | arg2                   |      identical       | result                      |
+===================+========================+======================+=============================+
| category          | category               | True                 | category                    |
+-------------------+------------------------+----------------------+-----------------------------+
| category (object) | category (object)      | False                | object (dtype is inferred)  |
+-------------------+------------------------+----------------------+-----------------------------+
| category (int)    | category (float)       | False                | float (dtype is inferred)   |
+-------------------+------------------------+----------------------+-----------------------------+



### Unioning
If you want to combine categoricals that do not necessarily have the same
categories, the `~pandas.api.types.union_categoricals` function will
combine a list-like of categoricals. The new categories will be the union of
the categories being combined.

```python
from pandas.api.types import union_categoricals

a = pd.Categorical(["b", "c"])
b = pd.Categorical(["a", "b"])
union_categoricals([a, b])
```
By default, the resulting categories will be ordered as
they appear in the data. If you want the categories to
be lexsorted, use ``sort_categories=True`` argument.

```python
union_categoricals([a, b], sort_categories=True)
```
``union_categoricals`` also works with the "easy" case of combining two
categoricals of the same categories and order information
(e.g. what you could also ``append`` for).

```python
a = pd.Categorical(["a", "b"], ordered=True)
b = pd.Categorical(["a", "b", "a"], ordered=True)
union_categoricals([a, b])
```
The below raises ``TypeError`` because the categories are ordered and not identical.

```python
:okexcept:

a = pd.Categorical(["a", "b"], ordered=True)
b = pd.Categorical(["a", "b", "c"], ordered=True)
union_categoricals([a, b])
```
Ordered categoricals with different categories or orderings can be combined by
using the ``ignore_ordered=True`` argument.

```python
a = pd.Categorical(["a", "b", "c"], ordered=True)
b = pd.Categorical(["c", "b", "a"], ordered=True)
union_categoricals([a, b], ignore_order=True)
```
`~pandas.api.types.union_categoricals` also works with a
``CategoricalIndex``, or ``Series`` containing categorical data, but note that
the resulting array will always be a plain ``Categorical``:

```python
a = pd.Series(["b", "c"], dtype="category")
b = pd.Series(["a", "b"], dtype="category")
union_categoricals([a, b])
```
> **note.capitalize():**
   ``union_categoricals`` may recode the integer codes for categories
   when combining categoricals.  This is likely what you want,
   but if you are relying on the exact numbering of the categories, be
   aware.

   ```python
c1 = pd.Categorical(["b", "c"])
c2 = pd.Categorical(["a", "b"])

c1
# "b" is coded to 0
c1.codes

c2
# "b" is coded to 1
c2.codes

c = union_categoricals([c1, c2])
c
# "b" is coded to 0 throughout, same as c1, different from c2
c.codes
```
## Getting data in/out
You can write data that contains ``category`` dtypes to a ``HDFStore``.
See `here <io.hdf5-categorical>` for an example and caveats.

It is also possible to write data to and reading data from *Stata* format files.
See `here <io.stata-categorical>` for an example and caveats.

Writing to a CSV file will convert the data, effectively removing any information about the
categorical (categories and ordering). So if you read back the CSV file you have to convert the
relevant columns back to ``category`` and assign the right categories and categories ordering.

```python
import io

s = pd.Series(pd.Categorical(["a", "b", "b", "a", "a", "d"]))
# rename the categories
s = s.cat.rename_categories(["very good", "good", "bad"])
# reorder the categories and add missing categories
s = s.cat.set_categories(["very bad", "bad", "medium", "good", "very good"])
df = pd.DataFrame({"cats": s, "vals": [1, 2, 3, 4, 5, 6]})
csv = io.StringIO()
df.to_csv(csv)
df2 = pd.read_csv(io.StringIO(csv.getvalue()))
df2.dtypes
df2["cats"]
# Redo the category
df2["cats"] = df2["cats"].astype("category")
df2["cats"] = df2["cats"].cat.set_categories(
    ["very bad", "bad", "medium", "good", "very good"]
)
df2.dtypes
df2["cats"]
```
The same holds for writing to a SQL database with ``to_sql``.

## Missing data
pandas primarily uses the value ``np.nan`` to represent missing data. It is by
default not included in computations. See the Missing Data section
.

Missing values should **not** be included in the Categorical's ``categories``,
only in the ``values``.
Instead, it is understood that NaN is different, and is always a possibility.
When working with the Categorical's ``codes``, missing values will always have
a code of ``-1``.

```python
s = pd.Series(["a", "b", np.nan, "a"], dtype="category")
# only two categories
s
s.cat.codes
```
Methods for working with missing data, e.g. `~Series.isna`, `~Series.fillna`,
`~Series.dropna`, all work normally:

```python
s = pd.Series(["a", "b", np.nan], dtype="category")
s
pd.isna(s)
s.fillna("a")
```
## Differences to R's ``factor``
The following differences to R's factor functions can be observed:

* R's ``levels`` are named ``categories``.
* R's ``levels`` are always of type string, while ``categories`` in pandas can be of any dtype.
* It's not possible to specify labels at creation time. Use ``s.cat.rename_categories(new_labels)``
  afterwards.
* In contrast to R's ``factor`` function, using categorical data as the sole input to create a
  new categorical series will *not* remove unused categories but create a new categorical series
  which is equal to the passed in one!
* R allows for missing values to be included in its ``levels`` (pandas' ``categories``). pandas
  does not allow ``NaN`` categories, but missing values can still be in the ``values``.


## Gotchas


### Memory usage


The memory usage of a ``Categorical`` is proportional to the number of categories plus the length of the data. In contrast,
an ``object`` dtype is a constant times the length of the data.

```python
s = pd.Series(["foo", "bar"] * 1000)

# object dtype
s.nbytes

# category dtype
s.astype("category").nbytes
```
> **note.capitalize():**
   If the number of categories approaches the length of the data, the ``Categorical`` will use nearly the same or
   more memory than an equivalent ``object`` dtype representation.

   ```python
s = pd.Series(["foo%04d" % i for i in range(2000)])

# object dtype
s.nbytes

# category dtype
s.astype("category").nbytes
```
### ``Categorical`` is not a ``numpy`` array
Currently, categorical data and the underlying ``Categorical`` is implemented as a Python
object and not as a low-level NumPy array dtype. This leads to some problems.

NumPy itself doesn't know about the new ``dtype``:

```python
try:
    np.dtype("category")
except TypeError as e:
    print("TypeError:", str(e))

dtype = pd.Categorical(["a"]).dtype
try:
    np.dtype(dtype)
except TypeError as e:
    print("TypeError:", str(e))
```
Dtype comparisons work:

```python
dtype == np.str_
np.str_ == dtype
```
To check if a Series contains Categorical data, use ``hasattr(s, 'cat')``:

```python
hasattr(pd.Series(["a"], dtype="category"), "cat")
hasattr(pd.Series(["a"]), "cat")
```
Using NumPy functions on a ``Series`` of type ``category`` should not work as ``Categoricals``
are not numeric data (even in the case that ``.categories`` is numeric).

```python
s = pd.Series(pd.Categorical([1, 2, 3, 4]))
try:
    np.sum(s)
    # same with np.log(s),...
except TypeError as e:
    print("TypeError:", str(e))
```
> **note.capitalize():**
    If such a function works, please file a bug at https://github.com/pandas-dev/pandas!

### dtype in apply
pandas currently does not preserve the dtype in apply functions: If you apply along rows you get
a ``Series`` of ``object`` ``dtype`` (same as getting a row -> getting one element will return a
basic type) and applying along columns will also convert to object. ``NaN`` values are unaffected.
You can use ``fillna`` to handle missing values before applying a function.

```python
df = pd.DataFrame(
    {
        "a": [1, 2, 3, 4],
        "b": ["a", "b", "c", "d"],
        "cats": pd.Categorical([1, 2, 3, 2]),
    }
)
df.apply(lambda row: type(row["cats"]), axis=1)
df.apply(lambda col: col.dtype, axis=0)
```
### Categorical index
``CategoricalIndex`` is a type of index that is useful for supporting
indexing with duplicates. This is a container around a ``Categorical``
and allows efficient indexing and storage of an index with a large number of duplicated elements.
See the `advanced indexing docs <advanced.categoricalindex>` for a more detailed
explanation.

Setting the index will create a ``CategoricalIndex``:

```python
cats = pd.Categorical([1, 2, 3, 4], categories=[4, 2, 3, 1])
strings = ["a", "b", "c", "d"]
values = [4, 2, 3, 1]
df = pd.DataFrame({"strings": strings, "values": values}, index=cats)
df.index
# This now sorts by the categories order
df.sort_index()
```
### Side effects
Constructing a ``Series`` from a ``Categorical`` will not copy the input
``Categorical``. This means that changes to the ``Series`` will in most cases
change the original ``Categorical``:

```python
cat = pd.Categorical([1, 2, 3, 10], categories=[1, 2, 3, 4, 10])
s = pd.Series(cat, name="cat")
cat
s.iloc[0:2] = 10
cat
```
Use ``copy=True`` to prevent such a behaviour or simply don't reuse ``Categoricals``:

```python
cat = pd.Categorical([1, 2, 3, 10], categories=[1, 2, 3, 4, 10])
s = pd.Series(cat, name="cat", copy=True)
cat
s.iloc[0:2] = 10
cat
```
> **note.capitalize():**
    This also happens in some cases when you supply a NumPy array instead of a ``Categorical``:
    using an int array (e.g. ``np.array([1,2,3,4])``) will exhibit the same behavior, while using
    a string array (e.g. ``np.array(["a","b","c","a"])``) will not.

---

# Nullable integer data type
> **note.capitalize():**
   IntegerArray is currently experimental. Its API or implementation may
   change without warning. Uses `pandas.NA` as the missing value.

In `missing_data`, we saw that pandas primarily uses ``NaN`` to represent
missing data. Because ``NaN`` is a float, this forces an array of integers with
any missing values to become floating point. In some cases, this may not matter
much. But if your integer column is, say, an identifier, casting to float can
be problematic. Some integers cannot even be represented as floating point
numbers.

## Construction
pandas can represent integer data with possibly missing values using
`arrays.IntegerArray`. This is an `extension type <extending.extension-types>`
implemented within pandas.

```python
arr = pd.array([1, 2, None], dtype=pd.Int64Dtype())
arr
```
Or the string alias ``"Int64"`` (note the capital ``"I"``) to differentiate from
NumPy's ``'int64'`` dtype:

```python
pd.array([1, 2, np.nan], dtype="Int64")
```
All NA-like values are replaced with `pandas.NA`.

```python
pd.array([1, 2, np.nan, None, pd.NA], dtype="Int64")
```
This array can be stored in a `DataFrame` or `Series` like any
NumPy array.

```python
pd.Series(arr)
```
You can also pass the list-like object to the `Series` constructor
with the dtype.

> **warning.capitalize():**
   Currently `pandas.array` and `pandas.Series` use different
   rules for dtype inference. `pandas.array` will infer a
   nullable-integer dtype

   ```python
pd.array([1, None])
   pd.array([1, 2])

For backwards-compatibility, `Series` infers these as either
integer or float dtype.



   pd.Series([1, None])
   pd.Series([1, 2])

We recommend explicitly providing the dtype to avoid confusion.



   pd.array([1, None], dtype="Int64")
   pd.Series([1, None], dtype="Int64")

In the future, we may provide an option for `Series` to infer a
nullable-integer dtype.
```
If you create a column of ``NA`` values (for example to fill them later)
with ``df['new_col'] = pd.NA``, the ``dtype`` would be set to ``object`` in the
new column. The performance on this column will be worse than with
the appropriate type. It's better to use
``df['new_col'] = pd.Series(pd.NA, dtype="Int64")``
(or another ``dtype`` that supports ``NA``).

```python
df = pd.DataFrame()
df['objects'] = pd.NA
df.dtypes
```
## Operations
Operations involving an integer array will behave similar to NumPy arrays.
Missing values will be propagated, and the data will be coerced to another
dtype if needed.

```python
s = pd.Series([1, 2, None], dtype="Int64")

# arithmetic
s + 1

# comparison
s == 1

# slicing operation
s.iloc[1:3]

# operate with other dtypes
s + s.iloc[1:3].astype("Int8")

# coerce when needed
s + 0.01
```
These dtypes can operate as part of a ``DataFrame``.

```python
df = pd.DataFrame({"A": s, "B": [1, 1, 3], "C": list("aab")})
df
df.dtypes
```
These dtypes can be merged, reshaped & casted.

```python
pd.concat([df[["A"]], df[["B", "C"]]], axis=1).dtypes
df["A"].astype(float)
```
Reduction and groupby operations such as `~DataFrame.sum` work as well.

```python
df.sum(numeric_only=True)
df.sum()
df.groupby("B").A.sum()
```
## Scalar NA value
`arrays.IntegerArray` uses `pandas.NA` as its scalar
missing value. Slicing a single element that's missing will return
`pandas.NA`

```python
a = pd.array([1, None], dtype="Int64")
a[1]
```

---

```python
:suppress:

import pandas as pd
import numpy as np
```


# Nullable Boolean data type
> **note.capitalize():**
   BooleanArray is currently experimental. Its API or implementation may
   change without warning.



## Indexing with NA values
pandas allows indexing with ``NA`` values in a boolean array, which are treated as ``False``.

```python
:okexcept:

s = pd.Series([1, 2, 3])
mask = pd.array([True, False, pd.NA], dtype="boolean")
s[mask]
```
If you would prefer to keep the ``NA`` values you can manually fill them with ``fillna(True)``.

```python
s[mask.fillna(True)]
```
If you create a column of ``NA`` values (for example to fill them later)
with ``df['new_col'] = pd.NA``, the ``dtype`` would be set to ``object`` in the
new column. The performance on this column will be worse than with
the appropriate type. It's better to use
``df['new_col'] = pd.Series(pd.NA, dtype="boolean")``
(or another ``dtype`` that supports ``NA``).

```python
df = pd.DataFrame()
df['objects'] = pd.NA
df.dtypes
```


## Kleene logical operations
`arrays.BooleanArray` implements `Kleene Logic`_ (sometimes called three-value logic) for
logical operations like ``&`` (and), ``|`` (or) and ``^`` (exclusive-or).

This table demonstrates the results for every combination. These operations are symmetrical,
so flipping the left- and right-hand side makes no difference in the result.

================= =========
Expression        Result
================= =========
``True & True``   ``True``
``True & False``  ``False``
``True & NA``     ``NA``
``False & False`` ``False``
``False & NA``    ``False``
``NA & NA``       ``NA``
``True | True``   ``True``
``True | False``  ``True``
``True | NA``     ``True``
``False | False`` ``False``
``False | NA``    ``NA``
``NA | NA``       ``NA``
``True ^ True``   ``False``
``True ^ False``  ``True``
``True ^ NA``     ``NA``
``False ^ False`` ``False``
``False ^ NA``    ``NA``
``NA ^ NA``       ``NA``
================= =========

When an ``NA`` is present in an operation, the output value is ``NA`` only if
the result cannot be determined solely based on the other input. For example,
``True | NA`` is ``True``, because both ``True | True`` and ``True | False``
are ``True``. In that case, we don't actually need to consider the value
of the ``NA``.

On the other hand, ``True & NA`` is ``NA``. The result depends on whether
the ``NA`` really is ``True`` or ``False``, since ``True & True`` is ``True``,
but ``True & False`` is ``False``, so we can't determine the output.


This differs from how ``np.nan`` behaves in logical operations. pandas treated
``np.nan`` is *always false in the output*.

In ``or``

```python
pd.Series([True, False, np.nan], dtype="object") | True
pd.Series([True, False, np.nan], dtype="boolean") | True
```
In ``and``

```python
pd.Series([True, False, np.nan], dtype="object") & True
pd.Series([True, False, np.nan], dtype="boolean") & True
```
.. _Kleene Logic: https://en.wikipedia.org/wiki/Three-valued_logic#Kleene_and_Priest_logics

---

# Chart visualization
> **note.capitalize():**
   The examples below assume that you're using [Jupyter](https://jupyter.org/).

This section demonstrates visualization through charting. For information on
visualization of tabular data please see the section on [Table Visualization](style.ipynb).

We use the standard convention for referencing the matplotlib API:

[``python
import matplotlib.pyplot as plt

plt.close("all")
```
We provide the basics in pandas to easily create decent looking plots.
See `the ecosystem page](https://pandas.pydata.org/community/ecosystem.html) for visualization
libraries that go beyond the basics documented here.

> **note.capitalize():**
   All calls to ``np.random`` are seeded with 123456.



## Basic plotting: ``plot``
We will demonstrate the basics, see the `cookbook<cookbook.plotting>` for
some advanced strategies.

The ``plot`` method on Series and DataFrame is just a simple wrapper around
`plt.plot() <matplotlib.axes.Axes.plot>`:

```python
np.random.seed(123456)

ts = pd.Series(np.random.randn(1000), index=pd.date_range("1/1/2000", periods=1000))
ts = ts.cumsum()

@savefig series_plot_basic.png
ts.plot();
```
If the index consists of dates, it calls `gcf().autofmt_xdate() <matplotlib.figure.Figure.autofmt_xdate>`
to try to format the x-axis nicely as per above.

On DataFrame, `~DataFrame.plot` is a convenience to plot all of the columns with labels:

```python
:suppress:

plt.close("all")
np.random.seed(123456)
```
```python
df = pd.DataFrame(np.random.randn(1000, 4), index=ts.index, columns=list("ABCD"))
df = df.cumsum()

plt.figure();
@savefig frame_plot_basic.png
df.plot();
```
You can plot one column versus another using the ``x`` and ``y`` keywords in
`~DataFrame.plot`:

```python
:suppress:

plt.close("all")
plt.figure()
np.random.seed(123456)
```
```python
df3 = pd.DataFrame(np.random.randn(1000, 2), columns=["B", "C"]).cumsum()
df3["A"] = pd.Series(list(range(len(df))))

@savefig df_plot_xy.png
df3.plot(x="A", y="B");
```
> **note.capitalize():**
   For more formatting and styling options, see
   `formatting <visualization.formatting>` below.

```python
:suppress:

plt.close("all")
```


## Other plots
Plotting methods allow for a handful of plot styles other than the
default line plot. These methods can be provided as the ``kind``
keyword argument to `~DataFrame.plot`, and include:

* `'bar' <visualization.barplot>` or `'barh' <visualization.barplot>` for bar plots
* `'hist' <visualization.hist>` for histogram
* `'box' <visualization.box>` for boxplot
* `'kde' <visualization.kde>` or `'density' <visualization.kde>` for density plots
* `'area' <visualization.area_plot>` for area plots
* `'scatter' <visualization.scatter>` for scatter plots
* `'hexbin' <visualization.hexbin>` for hexagonal bin plots
* `'pie' <visualization.pie>` for pie plots

For example, a bar plot can be created the following way:

```python
plt.figure();

@savefig bar_plot_ex.png
df.iloc[5].plot(kind="bar");
```
You can also create these other plots using the methods ``DataFrame.plot.<kind>`` instead of providing the ``kind`` keyword argument. This makes it easier to discover plot methods and the specific arguments they use:


    :verbatim:

    In [14]: df = pd.DataFrame()

    In [15]: df.plot.<TAB>  # noqa: E225, E999
    df.plot.area     df.plot.barh     df.plot.density  df.plot.hist     df.plot.line     df.plot.scatter
    df.plot.bar      df.plot.box      df.plot.hexbin   df.plot.kde      df.plot.pie

In addition to these ``kind`` s, there are the `DataFrame.hist() <visualization.hist>`,
and `DataFrame.boxplot() <visualization.box>` methods, which use a separate interface.

Finally, there are several `plotting functions <visualization.tools>` in ``pandas.plotting``
that take a `Series` or `DataFrame` as an argument. These
include:

* `Scatter Matrix <visualization.scatter_matrix>`
* `Andrews Curves <visualization.andrews_curves>`
* `Parallel Coordinates <visualization.parallel_coordinates>`
* `Lag Plot <visualization.lag>`
* `Autocorrelation Plot <visualization.autocorrelation>`
* `Bootstrap Plot <visualization.bootstrap>`
* `RadViz <visualization.radviz>`

Plots may also be adorned with `errorbars <visualization.errorbars>`
or `tables <visualization.table>`.



### Bar plots
For labeled, non-time series data, you may wish to produce a bar plot:

```python
plt.figure();

@savefig bar_plot_ex.png
df.iloc[5].plot.bar();
plt.axhline(0, color="k");
```
Calling a DataFrame's `plot.bar() <DataFrame.plot.bar>` method produces a multiple
bar plot:

```python
:suppress:

plt.close("all")
plt.figure()
np.random.seed(123456)
```
```python
df2 = pd.DataFrame(np.random.rand(10, 4), columns=["a", "b", "c", "d"])

@savefig bar_plot_multi_ex.png
df2.plot.bar();
```
To produce a stacked bar plot, pass ``stacked=True``:

```python
:suppress:

plt.close("all")
plt.figure()
```
```python
@savefig bar_plot_stacked_ex.png
df2.plot.bar(stacked=True);
```
To get horizontal bar plots, use the ``barh`` method:

```python
:suppress:

plt.close("all")
plt.figure()
```
```python
@savefig barh_plot_stacked_ex.png
df2.plot.barh(stacked=True);
```


### Histograms
Histograms can be drawn by using the `DataFrame.plot.hist` and `Series.plot.hist` methods.

```python
df4 = pd.DataFrame(
    {
        "a": np.random.randn(1000) + 1,
        "b": np.random.randn(1000),
        "c": np.random.randn(1000) - 1,
    },
    columns=["a", "b", "c"],
)

plt.figure();

@savefig hist_new.png
df4.plot.hist(alpha=0.5);
```
```python
:suppress:

plt.close("all")
```
A histogram can be stacked using ``stacked=True``. Bin size can be changed
using the ``bins`` keyword.

```python
plt.figure();

@savefig hist_new_stacked.png
df4.plot.hist(stacked=True, bins=20);
```
```python
:suppress:

plt.close("all")
```
You can pass other keywords supported by matplotlib ``hist``. For example,
horizontal and cumulative histograms can be drawn by
``orientation='horizontal'`` and ``cumulative=True``.

```python
plt.figure();

@savefig hist_new_kwargs.png
df4["a"].plot.hist(orientation="horizontal", cumulative=True);
```
```python
:suppress:

plt.close("all")
```
See the `hist <matplotlib.axes.Axes.hist>[ method and the
`matplotlib hist documentation](https://matplotlib.org/stable/api/_as_gen/matplotlib.pyplot.hist.html)_ for more.


The existing interface ``DataFrame.hist`` to plot histogram still can be used.

```python
plt.figure();

@savefig hist_plot_ex.png
df["A"].diff().hist();
```
```python
:suppress:

plt.close("all")
```
`DataFrame.hist` plots the histograms of the columns on multiple
subplots:

```python
plt.figure();

@savefig frame_hist_ex.png
df.diff().hist(color="k", alpha=0.5, bins=50);
```
The ``by`` keyword can be specified to plot grouped histograms:

```python
:suppress:

plt.close("all")
plt.figure()
np.random.seed(123456)
```
```python
data = pd.Series(np.random.randn(1000))

@savefig grouped_hist.png
data.hist(by=np.random.randint(0, 4, 1000), figsize=(6, 4));
```
```python
:suppress:

plt.close("all")
np.random.seed(123456)
```
In addition, the ``by`` keyword can also be specified in `DataFrame.plot.hist`.



```python
data = pd.DataFrame(
    {
        "a": np.random.choice(["x", "y", "z"], 1000),
        "b": np.random.choice(["e", "f", "g"], 1000),
        "c": np.random.randn(1000),
        "d": np.random.randn(1000) - 1,
    },
)

@savefig grouped_hist_by.png
data.plot.hist(by=["a", "b"], figsize=(10, 5));
```
```python
:suppress:

plt.close("all")
```


### Box plots
Boxplot can be drawn calling `Series.plot.box` and `DataFrame.plot.box`,
or `DataFrame.boxplot` to visualize the distribution of values within each column.

For instance, here is a boxplot representing five trials of 10 observations of
a uniform random variable on [0,1).

```python
:suppress:

plt.close("all")
np.random.seed(123456)
```
```python
df = pd.DataFrame(np.random.rand(10, 5), columns=["A", "B", "C", "D", "E"])

@savefig box_plot_new.png
df.plot.box();
```
Boxplot can be colorized by passing ``color`` keyword. You can pass a ``dict``
whose keys are ``boxes``, ``whiskers``, ``medians`` and ``caps``.
If some keys are missing in the ``dict``, default colors are used
for the corresponding artists. Also, boxplot has ``sym`` keyword to specify fliers style.

When you pass other type of arguments via ``color`` keyword, it will be directly
passed to matplotlib for all the ``boxes``, ``whiskers``, ``medians`` and ``caps``
colorization.

The colors are applied to every boxes to be drawn. If you want
more complicated colorization, you can get each drawn artists by passing
`return_type <visualization.box.return>`.

```python
color = {
    "boxes": "DarkGreen",
    "whiskers": "DarkOrange",
    "medians": "DarkBlue",
    "caps": "Gray",
}

@savefig box_new_colorize.png
df.plot.box(color=color, sym="r+");
```
```python
:suppress:

plt.close("all")
```
Also, you can pass other keywords supported by matplotlib ``boxplot``.
For example, horizontal and custom-positioned boxplot can be drawn by
``vert=False`` and ``positions`` keywords.

```python
@savefig box_new_kwargs.png
df.plot.box(vert=False, positions=[1, 4, 5, 6, 8]);
```
See the `boxplot <matplotlib.axes.Axes.boxplot>[ method and the
`matplotlib boxplot documentation](https://matplotlib.org/stable/api/_as_gen/matplotlib.pyplot.boxplot.html)_ for more.


The existing interface ``DataFrame.boxplot`` to plot boxplot still can be used.

```python
:suppress:

plt.close("all")
np.random.seed(123456)
```
```python
:okwarning:

df = pd.DataFrame(np.random.rand(10, 5))
plt.figure();

@savefig box_plot_ex.png
bp = df.boxplot()
```
You can create a stratified boxplot using the ``by`` keyword argument to create
groupings.  For instance,

```python
:suppress:

plt.close("all")
np.random.seed(123456)
```
```python
:okwarning:

df = pd.DataFrame(np.random.rand(10, 2), columns=["Col1", "Col2"])
df["X"] = pd.Series(["A", "A", "A", "A", "A", "B", "B", "B", "B", "B"])

plt.figure();

@savefig box_plot_ex2.png
bp = df.boxplot(by="X")
```
You can also pass a subset of columns to plot, as well as group by multiple
columns:

```python
:suppress:

plt.close("all")
np.random.seed(123456)
```
```python
:okwarning:

df = pd.DataFrame(np.random.rand(10, 3), columns=["Col1", "Col2", "Col3"])
df["X"] = pd.Series(["A", "A", "A", "A", "A", "B", "B", "B", "B", "B"])
df["Y"] = pd.Series(["A", "B", "A", "B", "A", "B", "A", "B", "A", "B"])

plt.figure();

@savefig box_plot_ex3.png
bp = df.boxplot(column=["Col1", "Col2"], by=["X", "Y"])
```
```python
:suppress:

 plt.close("all")
```
You could also create groupings with `DataFrame.plot.box`, for instance:



```python
:suppress:

plt.close("all")
np.random.seed(123456)
```
```python
:okwarning:

df = pd.DataFrame(np.random.rand(10, 3), columns=["Col1", "Col2", "Col3"])
df["X"] = pd.Series(["A", "A", "A", "A", "A", "B", "B", "B", "B", "B"])

plt.figure();

@savefig box_plot_ex4.png
bp = df.plot.box(column=["Col1", "Col2"], by="X")
```
```python
:suppress:

 plt.close("all")
```


In ``boxplot``, the return type can be controlled by the ``return_type``, keyword. The valid choices are ``{"axes", "dict", "both", None}``.
Faceting, created by ``DataFrame.boxplot`` with the ``by``
keyword, will affect the output type as well:

================ ======= ==========================
``return_type``  Faceted Output type
================ ======= ==========================
``None``         No      axes
``None``         Yes     2-D ndarray of axes
``'axes'``       No      axes
``'axes'``       Yes     Series of axes
``'dict'``       No      dict of artists
``'dict'``       Yes     Series of dicts of artists
``'both'``       No      namedtuple
``'both'``       Yes     Series of namedtuples
================ ======= ==========================

``Groupby.boxplot`` always returns a ``Series`` of ``return_type``.

```python
:okwarning:

np.random.seed(1234)
df_box = pd.DataFrame(np.random.randn(50, 2))
df_box["g"] = np.random.choice(["A", "B"], size=50)
df_box.loc[df_box["g"] == "B", 1] += 3

@savefig boxplot_groupby.png
bp = df_box.boxplot(by="g")
```
```python
:suppress:

plt.close("all")
```
The subplots above are split by the numeric columns first, then the value of
the ``g`` column. Below the subplots are first split by the value of ``g``,
then by the numeric columns.

```python
:okwarning:

@savefig groupby_boxplot_vis.png
bp = df_box.groupby("g").boxplot()
```
```python
:suppress:

plt.close("all")
```


### Area plot
You can create area plots with `Series.plot.area` and `DataFrame.plot.area`.
Area plots are stacked by default. To produce stacked area plot, each column must be either all positive or all negative values.

When input data contains ``NaN``, it will be automatically filled by 0. If you want to drop or fill by different values, use `dataframe.dropna` or `dataframe.fillna` before calling ``plot``.

```python
:suppress:

np.random.seed(123456)
plt.figure()
```
```python
df = pd.DataFrame(np.random.rand(10, 4), columns=["a", "b", "c", "d"])

@savefig area_plot_stacked.png
df.plot.area();
```
To produce an unstacked plot, pass ``stacked=False``. Alpha value is set to 0.5 unless otherwise specified:

```python
:suppress:

plt.close("all")
plt.figure()
```
```python
@savefig area_plot_unstacked.png
df.plot.area(stacked=False);
```


### Scatter plot
Scatter plot can be drawn by using the `DataFrame.plot.scatter` method.
Scatter plot requires numeric columns for the x and y axes.
These can be specified by the ``x`` and ``y`` keywords.

```python
:suppress:

np.random.seed(123456)
plt.close("all")
plt.figure()
```
```python
df = pd.DataFrame(np.random.rand(50, 4), columns=["a", "b", "c", "d"])
df["species"] = pd.Categorical(
    ["setosa"] * 20 + ["versicolor"] * 20 + ["virginica"] * 10
)

@savefig scatter_plot.png
df.plot.scatter(x="a", y="b");
```
To plot multiple column groups in a single axes, repeat ``plot`` method specifying target ``ax``.
It is recommended to specify ``color`` and ``label`` keywords to distinguish each groups.

```python
:okwarning:

ax = df.plot.scatter(x="a", y="b", color="DarkBlue", label="Group 1")
@savefig scatter_plot_repeated.png
df.plot.scatter(x="c", y="d", color="DarkGreen", label="Group 2", ax=ax);
```
```python
:suppress:

plt.close("all")
```
The keyword ``c`` may be given as the name of a column to provide colors for
each point:

```python
@savefig scatter_plot_colored.png
df.plot.scatter(x="a", y="b", c="c", s=50);
```
```python
:suppress:

plt.close("all")
```
If a categorical column is passed to ``c``, then a discrete colorbar will be produced:



```python
@savefig scatter_plot_categorical.png
df.plot.scatter(x="a", y="b", c="species", cmap="viridis", s=50);
```
```python
:suppress:

plt.close("all")
```
You can pass other keywords supported by matplotlib
`scatter <matplotlib.axes.Axes.scatter>`. The example  below shows a
bubble chart using a column of the ``DataFrame`` as the bubble size.

```python
@savefig scatter_plot_bubble.png
df.plot.scatter(x="a", y="b", s=df["c"] * 200);
```
```python
:suppress:

plt.close("all")
```
See the `scatter <matplotlib.axes.Axes.scatter>[ method and the
`matplotlib scatter documentation](https://matplotlib.org/stable/api/_as_gen/matplotlib.pyplot.scatter.html)_ for more.



### Hexagonal bin plot
You can create hexagonal bin plots with `DataFrame.plot.hexbin`.
Hexbin plots can be a useful alternative to scatter plots if your data are
too dense to plot each point individually.

```python
:suppress:

plt.figure()
np.random.seed(123456)
```
```python
df = pd.DataFrame(np.random.randn(1000, 2), columns=["a", "b"])
df["b"] = df["b"] + np.arange(1000)

@savefig hexbin_plot.png
df.plot.hexbin(x="a", y="b", gridsize=25);
```
A useful keyword argument is ``gridsize``; it controls the number of hexagons
in the x-direction, and defaults to 100. A larger ``gridsize`` means more, smaller
bins.

By default, a histogram of the counts around each ``(x, y)`` point is computed.
You can specify alternative aggregations by passing values to the ``C`` and
``reduce_C_function`` arguments. ``C`` specifies the value at each ``(x, y)`` point
and ``reduce_C_function`` is a function of one argument that reduces all the
values in a bin to a single number (e.g. ``mean``, ``max``, ``sum``, ``std``).  In this
example the positions are given by columns ``a`` and ``b``, while the value is
given by column ``z``. The bins are aggregated with NumPy's ``max`` function.

```python
:suppress:

plt.close("all")
plt.figure()
np.random.seed(123456)
```
```python
df = pd.DataFrame(np.random.randn(1000, 2), columns=["a", "b"])
df["b"] = df["b"] + np.arange(1000)
df["z"] = np.random.uniform(0, 3, 1000)

@savefig hexbin_plot_agg.png
df.plot.hexbin(x="a", y="b", C="z", reduce_C_function=np.max, gridsize=25);
```
```python
:suppress:

plt.close("all")
```
See the `hexbin <matplotlib.axes.Axes.hexbin>[ method and the
`matplotlib hexbin documentation](https://matplotlib.org/stable/api/_as_gen/matplotlib.pyplot.hexbin.html)_ for more.



### Pie plot
You can create a pie plot with [DataFrame.plot.pie` or `Series.plot.pie`.
If your data includes any ``NaN``, they will be automatically filled with 0.
A ``ValueError`` will be raised if there are any negative values in your data.

```python
:suppress:

np.random.seed(123456)
plt.figure()
```
```python
:okwarning:

series = pd.Series(3 * np.random.rand(4), index=["a", "b", "c", "d"], name="series")

@savefig series_pie_plot.png
series.plot.pie(figsize=(6, 6));
```
```python
:suppress:

plt.close("all")
```
For pie plots it's best to use square figures, i.e. a figure aspect ratio 1.
You can create the figure with equal width and height, or force the aspect ratio
to be equal after plotting by calling ``ax.set_aspect('equal')`` on the returned
``axes`` object.

Note that pie plot with `DataFrame` requires that you either specify a
target column by the ``y`` argument or ``subplots=True``. When ``y`` is
specified, pie plot of selected column will be drawn. If ``subplots=True`` is
specified, pie plots for each column are drawn as subplots. A legend will be
drawn in each pie plots by default; specify ``legend=False`` to hide it.

```python
:suppress:

np.random.seed(123456)
plt.figure()
```
```python
df = pd.DataFrame(
    3 * np.random.rand(4, 2), index=["a", "b", "c", "d"], columns=["x", "y"]
)

@savefig df_pie_plot.png
df.plot.pie(subplots=True, figsize=(8, 4));
```
```python
:suppress:

plt.close("all")
```
You can use the ``labels`` and ``colors`` keywords to specify the labels and colors of each wedge.

> **warning.capitalize():**
   Most pandas plots use the ``label`` and ``color`` arguments (note the lack of "s" on those).
   To be consistent with `matplotlib.pyplot.pie` you must use ``labels`` and ``colors``.

If you want to hide wedge labels, specify ``labels=None``.
If ``fontsize`` is specified, the value will be applied to wedge labels.
Also, other keywords supported by `matplotlib.pyplot.pie` can be used.


```python
:suppress:

plt.figure()
```
```python
@savefig series_pie_plot_options.png
series.plot.pie(
    labels=["AA", "BB", "CC", "DD"],
    colors=["r", "g", "b", "c"],
    autopct="%.2f",
    fontsize=20,
    figsize=(6, 6),
);
```
If you pass values whose sum total is less than 1.0 they will be rescaled so that they sum to 1.

```python
:suppress:

plt.close("all")
plt.figure()
```
```python
:okwarning:

series = pd.Series([0.1] * 4, index=["a", "b", "c", "d"], name="series2")

@savefig series_pie_plot_semi.png
series.plot.pie(figsize=(6, 6));
```
See the `matplotlib pie documentation](https://matplotlib.org/stable/api/_as_gen/matplotlib.pyplot.pie.html)_ for more.

[``python
:suppress:

plt.close("all")
```


## Plotting with missing data
pandas tries to be pragmatic about plotting ``DataFrames`` or ``Series``
that contain missing data. Missing values are dropped, left out, or filled
depending on the plot type.

+----------------+--------------------------------------+
| Plot Type      | NaN Handling                         |
+================+======================================+
| Line           | Leave gaps at NaNs                   |
+----------------+--------------------------------------+
| Line (stacked) | Fill 0's                             |
+----------------+--------------------------------------+
| Bar            | Fill 0's                             |
+----------------+--------------------------------------+
| Scatter        | Drop NaNs                            |
+----------------+--------------------------------------+
| Histogram      | Drop NaNs (column-wise)              |
+----------------+--------------------------------------+
| Box            | Drop NaNs (column-wise)              |
+----------------+--------------------------------------+
| Area           | Fill 0's                             |
+----------------+--------------------------------------+
| KDE            | Drop NaNs (column-wise)              |
+----------------+--------------------------------------+
| Hexbin         | Drop NaNs                            |
+----------------+--------------------------------------+
| Pie            | Fill 0's                             |
+----------------+--------------------------------------+

If any of these defaults are not what you want, or if you want to be
explicit about how missing values are handled, consider using
`~pandas.DataFrame.fillna` or `~pandas.DataFrame.dropna`
before plotting.



## Plotting tools
These functions can be imported from ``pandas.plotting``
and take a `Series` or `DataFrame` as an argument.



### Scatter matrix plot
You can create a scatter plot matrix using the
``scatter_matrix`` method in ``pandas.plotting``:

```python
:suppress:

np.random.seed(123456)
```
```python
from pandas.plotting import scatter_matrix

df = pd.DataFrame(np.random.randn(1000, 4), columns=["a", "b", "c", "d"])

@savefig scatter_matrix_kde.png
scatter_matrix(df, alpha=0.2, figsize=(6, 6), diagonal="kde");
```
```python
:suppress:

plt.close("all")
```


### Density plot
You can create density plots using the `Series.plot.kde` and `DataFrame.plot.kde` methods.

```python
:suppress:

plt.figure()
np.random.seed(123456)
```
```python
ser = pd.Series(np.random.randn(1000))

@savefig kde_plot.png
ser.plot.kde();
```
```python
:suppress:

plt.close("all")
```


### Andrews curves
Andrews curves allow one to plot multivariate data as a large number
of curves that are created using the attributes of samples as coefficients
for Fourier series, see the `Wikipedia entry](https://en.wikipedia.org/wiki/Andrews_plot)_
for more information. By coloring these curves differently for each class
it is possible to visualize data clustering. Curves belonging to samples
of the same class will usually be closer together and form larger structures.

**Note**: The "Iris" dataset is available [here](https://raw.githubusercontent.com/pandas-dev/pandas/main/pandas/tests/io/data/csv/iris.csv)_.

[``python
from pandas.plotting import andrews_curves

data = pd.read_csv("data/iris.data")

plt.figure();

@savefig andrews_curves.png
andrews_curves(data, "Name");
```


### Parallel coordinates
Parallel coordinates is a plotting technique for plotting multivariate data,
see the `Wikipedia entry](https://en.wikipedia.org/wiki/Parallel_coordinates)_
for an introduction.
Parallel coordinates allows one to see clusters in data and to estimate other statistics visually.
Using parallel coordinates points are represented as connected line segments.
Each vertical line represents one attribute. One set of connected line segments
represents one data point. Points that tend to cluster will appear closer together.

[``python
from pandas.plotting import parallel_coordinates

data = pd.read_csv("data/iris.data")

plt.figure();

@savefig parallel_coordinates.png
parallel_coordinates(data, "Name");
```
```python
:suppress:

plt.close("all")
```


### Lag plot
Lag plots are used to check if a data set or time series is random. Random
data should not exhibit any structure in the lag plot. Non-random structure
implies that the underlying data are not random. The ``lag`` argument may
be passed, and when ``lag=1`` the plot is essentially ``data[:-1]`` vs.
``data[1:]``.

```python
:suppress:

np.random.seed(123456)
```
```python
from pandas.plotting import lag_plot

plt.figure();

spacing = np.linspace(-99 * np.pi, 99 * np.pi, num=1000)
data = pd.Series(0.1 * np.random.rand(1000) + 0.9 * np.sin(spacing))

@savefig lag_plot.png
lag_plot(data);
```
```python
:suppress:

plt.close("all")
```


### Autocorrelation plot
Autocorrelation plots are often used for checking randomness in time series.
This is done by computing autocorrelations for data values at varying time lags.
If time series is random, such autocorrelations should be near zero for any and
all time-lag separations. If time series is non-random then one or more of the
autocorrelations will be significantly non-zero. The horizontal lines displayed
in the plot correspond to 95% and 99% confidence bands. The dashed line is 99%
confidence band. See the
`Wikipedia entry](https://en.wikipedia.org/wiki/Correlogram)_ for more about
autocorrelation plots.

[``python
:suppress:

np.random.seed(123456)
```
```python
from pandas.plotting import autocorrelation_plot

plt.figure();

spacing = np.linspace(-9 * np.pi, 9 * np.pi, num=1000)
data = pd.Series(0.7 * np.random.rand(1000) + 0.3 * np.sin(spacing))

@savefig autocorrelation_plot.png
autocorrelation_plot(data);
```
```python
:suppress:

plt.close("all")
```


### Bootstrap plot
Bootstrap plots are used to visually assess the uncertainty of a statistic, such
as mean, median, midrange, etc. A random subset of a specified size is selected
from a data set, the statistic in question is computed for this subset and the
process is repeated a specified number of times. Resulting plots and histograms
are what constitutes the bootstrap plot.

```python
:suppress:

np.random.seed(123456)
```
```python
from pandas.plotting import bootstrap_plot

data = pd.Series(np.random.rand(1000))

@savefig bootstrap_plot.png
bootstrap_plot(data, size=50, samples=500, color="grey");
```
```python
:suppress:

 plt.close("all")
```


### RadViz
RadViz is a way of visualizing multi-variate data. It is based on a simple
spring tension minimization algorithm. Basically you set up a bunch of points in
a plane. In our case they are equally spaced on a unit circle. Each point
represents a single attribute. You then pretend that each sample in the data set
is attached to each of these points by a spring, the stiffness of which is
proportional to the numerical value of that attribute (they are normalized to
unit interval). The point in the plane, where our sample settles to (where the
forces acting on our sample are at an equilibrium) is where a dot representing
our sample will be drawn. Depending on which class that sample belongs it will
be colored differently.
See the R package `Radviz](https://cran.r-project.org/web/packages/Radviz/index.html)_
for more information.

**Note**: The "Iris" dataset is available [here](https://raw.githubusercontent.com/pandas-dev/pandas/main/pandas/tests/io/data/csv/iris.csv)_.

```python
from pandas.plotting import radviz

data = pd.read_csv("data/iris.data")

plt.figure();

@savefig radviz.png
radviz(data, "Name");
```
```python
:suppress:

plt.close("all")
```


## Plot formatting
### Setting the plot style
From version 1.5 and up, matplotlib offers a range of pre-configured plotting styles. Setting the
style can be used to easily give plots the general look that you want.
Setting the style is as easy as calling ``matplotlib.style.use(my_plot_style)`` before
creating your plot. For example you could write ``matplotlib.style.use('ggplot')`` for ggplot-style
plots.

You can see the various available style names at ``matplotlib.style.available`` and it's very
easy to try them out.

### General plot style arguments
Most plotting methods have a set of keyword arguments that control the
layout and formatting of the returned plot:

```python
plt.figure();
@savefig series_plot_basic2.png
ts.plot(style="k--", label="Series");
```
```python
:suppress:

plt.close("all")
```
For each kind of plot (e.g. ``line``, ``bar``, ``scatter``) any additional arguments
keywords are passed along to the corresponding matplotlib function
(`ax.plot() <matplotlib.axes.Axes.plot>`,
`ax.bar() <matplotlib.axes.Axes.bar>`,
`ax.scatter() <matplotlib.axes.Axes.scatter>`). These can be used
to control additional styling, beyond what pandas provides.

### Controlling the legend
You may set the ``legend`` argument to ``False`` to hide the legend, which is
shown by default.

```python
:suppress:

np.random.seed(123456)
```
```python
df = pd.DataFrame(np.random.randn(1000, 4), index=ts.index, columns=list("ABCD"))
df = df.cumsum()

@savefig frame_plot_basic_noleg.png
df.plot(legend=False);
```
```python
:suppress:

plt.close("all")
```
### Controlling the labels
You may set the ``xlabel`` and ``ylabel`` arguments to give the plot custom labels
for x and y axis. By default, pandas will pick up index name as xlabel, while leaving
it empty for ylabel.

```python
df.plot();

@savefig plot_xlabel_ylabel.png
df.plot(xlabel="new x", ylabel="new y");
```
```python
:suppress:

plt.close("all")
```
### Scales
You may pass ``logy`` to get a log-scale Y axis.

```python
:suppress:

plt.figure()
np.random.seed(123456)
```
```python
ts = pd.Series(np.random.randn(1000), index=pd.date_range("1/1/2000", periods=1000))
ts = np.exp(ts.cumsum())

@savefig series_plot_logy.png
ts.plot(logy=True);
```
```python
:suppress:

plt.close("all")
```
See also the ``logx`` and ``loglog`` keyword arguments.

### Plotting on a secondary y-axis
To plot data on a secondary y-axis, use the ``secondary_y`` keyword:

```python
:suppress:

plt.figure()
```
```python
df["A"].plot();

@savefig series_plot_secondary_y.png
df["B"].plot(secondary_y=True, style="g");
```
```python
:suppress:

plt.close("all")
```
To plot some columns in a ``DataFrame``, give the column names to the ``secondary_y``
keyword:

```python
plt.figure();
ax = df.plot(secondary_y=["A", "B"])
ax.set_ylabel("CD scale");
@savefig frame_plot_secondary_y.png
ax.right_ax.set_ylabel("AB scale");
```
```python
:suppress:

plt.close("all")
```
Note that the columns plotted on the secondary y-axis is automatically marked
with "(right)" in the legend. To turn off the automatic marking, use the
``mark_right=False`` keyword:

```python
plt.figure();

@savefig frame_plot_secondary_y_no_right.png
df.plot(secondary_y=["A", "B"], mark_right=False);
```
```python
:suppress:

plt.close("all")
```


### Custom formatters for timeseries plots
pandas provides custom formatters for timeseries plots. These change the
formatting of the axis labels for dates and times. By default,
the custom formatters are applied only to plots created by pandas with
`DataFrame.plot` or `Series.plot`. To have them apply to all
plots, including those made by matplotlib, set the option
``pd.options.plotting.matplotlib.register_converters = True`` or use
`pandas.plotting.register_matplotlib_converters`.

### Suppressing tick resolution adjustment
pandas includes automatic tick resolution adjustment for regular frequency
time-series data. For limited cases where pandas cannot infer the frequency
information (e.g., in an externally created ``twinx``), you can choose to
suppress this behavior for alignment purposes.

Here is the default behavior, notice how the x-axis tick labeling is performed:

```python
plt.figure();

@savefig ser_plot_suppress.png
df["A"].plot();
```
```python
:suppress:

plt.close("all")
```
Using the ``x_compat`` parameter, you can suppress this behavior:

```python
plt.figure();

@savefig ser_plot_suppress_parm.png
df["A"].plot(x_compat=True);
```
```python
:suppress:

plt.close("all")
```
If you have more than one plot that needs to be suppressed, the ``use`` method
in ``pandas.plotting.plot_params`` can be used in a ``with`` statement:

```python
plt.figure();

@savefig ser_plot_suppress_context.png
with pd.plotting.plot_params.use("x_compat", True):
    df["A"].plot(color="r")
    df["B"].plot(color="g")
    df["C"].plot(color="b")
```
```python
:suppress:

plt.close("all")
```
### Automatic date tick adjustment
``TimedeltaIndex`` now uses the native matplotlib
tick locator methods, it is useful to call the automatic
date tick adjustment from matplotlib for figures whose ticklabels overlap.

See the `autofmt_xdate <matplotlib.figure.autofmt_xdate>[ method and the
`matplotlib documentation](https://matplotlib.org/2.0.2/users/recipes.html#fixing-common-date-annoyances)_ for more.

### Subplots
Each [`Series`` in a ``DataFrame`` can be plotted on a different axis
with the ``subplots`` keyword:

```python
@savefig frame_plot_subplots.png
df.plot(subplots=True, figsize=(6, 6));
```
```python
:suppress:

plt.close("all")
```
### Using layout and targeting multiple axes
The layout of subplots can be specified by the ``layout`` keyword. It can accept
``(rows, columns)``. The ``layout`` keyword can be used in
``hist`` and ``boxplot`` also. If the input is invalid, a ``ValueError`` will be raised.

The number of axes which can be contained by rows x columns specified by ``layout`` must be
larger than the number of required subplots. If layout can contain more axes than required,
blank axes are not drawn. Similar to a NumPy array's ``reshape`` method, you
can use ``-1`` for one dimension to automatically calculate the number of rows
or columns needed, given the other.

```python
@savefig frame_plot_subplots_layout.png
df.plot(subplots=True, layout=(2, 3), figsize=(6, 6), sharex=False);
```
```python
:suppress:

plt.close("all")
```
The above example is identical to using:

```python
df.plot(subplots=True, layout=(2, -1), figsize=(6, 6), sharex=False);
```
```python
:suppress:

plt.close("all")
```
The required number of columns (3) is inferred from the number of series to plot
and the given number of rows (2).

You can pass multiple axes created beforehand as list-like via ``ax`` keyword.
This allows more complicated layouts.
The passed axes must be the same number as the subplots being drawn.

When multiple axes are passed via the ``ax`` keyword, ``layout``, ``sharex`` and ``sharey`` keywords
don't affect to the output. You should explicitly pass ``sharex=False`` and ``sharey=False``,
otherwise you will see a warning.

```python
fig, axes = plt.subplots(4, 4, figsize=(9, 9))
plt.subplots_adjust(wspace=0.5, hspace=0.5)
target1 = [axes[0][0], axes[1][1], axes[2][2], axes[3][3]]
target2 = [axes[3][0], axes[2][1], axes[1][2], axes[0][3]]

df.plot(subplots=True, ax=target1, legend=False, sharex=False, sharey=False);
@savefig frame_plot_subplots_multi_ax.png
(-df).plot(subplots=True, ax=target2, legend=False, sharex=False, sharey=False);
```
```python
:suppress:

plt.close("all")
```
Another option is passing an ``ax`` argument to `Series.plot` to plot on a particular axis:

```python
np.random.seed(123456)
ts = pd.Series(np.random.randn(1000), index=pd.date_range("1/1/2000", periods=1000))
ts = ts.cumsum()

df = pd.DataFrame(np.random.randn(1000, 4), index=ts.index, columns=list("ABCD"))
df = df.cumsum()
```
```python
:suppress:

plt.close("all")
```
```python
fig, axes = plt.subplots(nrows=2, ncols=2)
plt.subplots_adjust(wspace=0.2, hspace=0.5)
df["A"].plot(ax=axes[0, 0]);
axes[0, 0].set_title("A");
df["B"].plot(ax=axes[0, 1]);
axes[0, 1].set_title("B");
df["C"].plot(ax=axes[1, 0]);
axes[1, 0].set_title("C");
df["D"].plot(ax=axes[1, 1]);
@savefig series_plot_multi.png
axes[1, 1].set_title("D");
```
```python
:suppress:

 plt.close("all")
```


### Plotting with error bars
Plotting with error bars is supported in `DataFrame.plot` and `Series.plot`.

Horizontal and vertical error bars can be supplied to the ``xerr`` and ``yerr`` keyword arguments to `~DataFrame.plot`. The error values can be specified using a variety of formats:

* As a `DataFrame` or ``dict`` of errors with column names matching the ``columns`` attribute of the plotting `DataFrame` or matching the ``name`` attribute of the `Series`.
* As a ``str`` indicating which of the columns of plotting `DataFrame` contain the error values.
* As raw values (``list``, ``tuple``, or ``np.ndarray``). Must be the same length as the plotting `DataFrame`/`Series`.

Here is an example of one way to easily plot group means with standard deviations from the raw data.

```python
# Generate the data
ix3 = pd.MultiIndex.from_arrays(
    [
        ["a", "a", "a", "a", "a", "b", "b", "b", "b", "b"],
        ["foo", "foo", "foo", "bar", "bar", "foo", "foo", "bar", "bar", "bar"],
    ],
    names=["letter", "word"],
)

df3 = pd.DataFrame(
    {
        "data1": [9, 3, 2, 4, 3, 2, 4, 6, 3, 2],
        "data2": [9, 6, 5, 7, 5, 4, 5, 6, 5, 1],
    },
    index=ix3,
)

# Group by index labels and take the means and standard deviations
# for each group
gp3 = df3.groupby(level=("letter", "word"))
means = gp3.mean()
errors = gp3.std()
means
errors

# Plot
fig, ax = plt.subplots()
@savefig errorbar_example.png
means.plot.bar(yerr=errors, ax=ax, capsize=4, rot=0);
```
```python
:suppress:

plt.close("all")
```
Asymmetrical error bars are also supported, however raw error values must be provided in this case. For a ``N`` length `Series`, a ``2xN`` array should be provided indicating lower and upper (or left and right) errors. For a ``MxN`` `DataFrame`, asymmetrical errors should be in a ``Mx2xN`` array.

Here is an example of one way to plot the min/max range using asymmetrical error bars.

```python
mins = gp3.min()
maxs = gp3.max()

# errors should be positive, and defined in the order of lower, upper
errors = [[means[c] - mins[c], maxs[c] - means[c]] for c in df3.columns]

# Plot
fig, ax = plt.subplots()
@savefig errorbar_asymmetrical_example.png
means.plot.bar(yerr=errors, ax=ax, capsize=4, rot=0);
```
```python
:suppress:

plt.close("all")
```


### Plotting tables
Plotting with matplotlib table is now supported in  `DataFrame.plot` and `Series.plot` with a ``table`` keyword. The ``table`` keyword can accept ``bool``, `DataFrame` or `Series`. The simple way to draw a table is to specify ``table=True``. Data will be transposed to meet matplotlib's default layout.

```python
np.random.seed(123456)
fig, ax = plt.subplots(1, 1, figsize=(7, 6.5))
df = pd.DataFrame(np.random.rand(5, 3), columns=["a", "b", "c"])
ax.xaxis.tick_top()  # Display x-axis ticks on top.

@savefig line_plot_table_true.png
df.plot(table=True, ax=ax);
```
```python
:suppress:

plt.close("all")
```
Also, you can pass a different `DataFrame` or `Series` to the
``table`` keyword. The data will be drawn as displayed in print method
(not transposed automatically). If required, it should be transposed manually
as seen in the example below.

```python
fig, ax = plt.subplots(1, 1, figsize=(7, 6.75))
ax.xaxis.tick_top()  # Display x-axis ticks on top.

@savefig line_plot_table_data.png
df.plot(table=np.round(df.T, 2), ax=ax);
```
```python
:suppress:

plt.close("all")
```
There also exists a helper function ``pandas.plotting.table``, which creates a
table from `DataFrame` or `Series`, and adds it to an
``matplotlib.Axes`` instance. This function can accept keywords which the
matplotlib `table](https://matplotlib.org/stable/api/_as_gen/matplotlib.axes.Axes.table.html)_ has.

[``python
from pandas.plotting import table

fig, ax = plt.subplots(1, 1)

table(ax, np.round(df.describe(), 2), loc="upper right", colWidths=[0.2, 0.2, 0.2]);

@savefig line_plot_table_describe.png
df.plot(ax=ax, ylim=(0, 2), legend=None);
```
```python
:suppress:

plt.close("all")
```
**Note**: You can get table instances on the axes using ``axes.tables`` property for further decorations. See the `matplotlib table documentation](https://matplotlib.org/api/axes_api.html#matplotlib.axes.Axes.table)_ for more.



### Colormaps
A potential issue when plotting a large number of columns is that it can be
difficult to distinguish some series due to repetition in the default colors. To
remedy this, [`DataFrame`` plotting supports the use of the ``colormap`` argument,
which accepts either a Matplotlib `colormap](https://matplotlib.org/api/cm_api.html)_
or a string that is a name of a colormap registered with Matplotlib. A
visualization of the default matplotlib colormaps is available `here
<https://matplotlib.org/stable/gallery/color/colormap_reference.html>[__.

As matplotlib does not directly support colormaps for line-based plots, the
colors are selected based on an even spacing determined by the number of columns
in the ``DataFrame``. There is no consideration made for background color, so some
colormaps will produce lines that are not easily visible.

To use the cubehelix colormap, we can pass ``colormap='cubehelix'``.

```python
np.random.seed(123456)
df = pd.DataFrame(np.random.randn(1000, 10), index=ts.index)
df = df.cumsum()

plt.figure();

@savefig cubehelix.png
df.plot(colormap="cubehelix");
```
```python
:suppress:

plt.close("all")
```
Alternatively, we can pass the colormap itself:

```python
from matplotlib import cm

plt.figure();

@savefig cubehelix_cm.png
df.plot(colormap=cm.cubehelix);
```
```python
:suppress:

plt.close("all")
```
Colormaps can also be used other plot types, like bar charts:

```python
np.random.seed(123456)
dd = pd.DataFrame(np.random.randn(10, 10)).map(abs)
dd = dd.cumsum()

plt.figure();

@savefig greens.png
dd.plot.bar(colormap="Greens");
```
```python
:suppress:

plt.close("all")
```
Parallel coordinates charts:

```python
plt.figure();

@savefig parallel_gist_rainbow.png
parallel_coordinates(data, "Name", colormap="gist_rainbow");
```
```python
:suppress:

plt.close("all")
```
Andrews curves charts:

```python
plt.figure();

@savefig andrews_curve_winter.png
andrews_curves(data, "Name", colormap="winter");
```
```python
:suppress:

plt.close("all")
```
## Plotting directly with Matplotlib
In some situations it may still be preferable or necessary to prepare plots
directly with matplotlib, for instance when a certain type of plot or
customization is not (yet) supported by pandas. ``Series`` and ``DataFrame``
objects behave like arrays and can therefore be passed directly to
matplotlib functions without explicit casts.

pandas also automatically registers formatters and locators that recognize date
indices, thereby extending date and time support to practically all plot types
available in matplotlib. Although this formatting does not provide the same
level of refinement you would get when plotting via pandas, it can be faster
when plotting a large number of points.

```python
np.random.seed(123456)
price = pd.Series(
    np.random.randn(150).cumsum(),
    index=pd.date_range("2000-1-1", periods=150, freq="B"),
)
ma = price.rolling(20).mean()
mstd = price.rolling(20).std()

plt.figure();

plt.plot(price.index, price, "k");
plt.plot(ma.index, ma, "b");
@savefig bollinger.png
plt.fill_between(mstd.index, ma - 2 * mstd, ma + 2 * mstd, color="b", alpha=0.2);
```
```python
:suppress:

 plt.close("all")
```
## Plotting backends
pandas can be extended with third-party plotting backends. The
main idea is letting users select a plotting backend different than the provided
one based on Matplotlib.

This can be done by passing 'backend.module' as the argument ``backend`` in ``plot``
function. For example:

```python
>>> Series([1, 2, 3]).plot(backend="backend.module")
```
Alternatively, you can also set this option globally, do you don't need to specify
the keyword in each ``plot`` call. For example:

```python
>>> pd.set_option("plotting.backend", "backend.module")
>>> pd.Series([1, 2, 3]).plot()
```
Or:

```python
>>> pd.options.plotting.backend = "backend.module"
>>> pd.Series([1, 2, 3]).plot()
```
This would be more or less equivalent to:

```python
>>> import backend.module
>>> backend.module.plot(pd.Series([1, 2, 3]))
```
The backend module can then use other visualization tools (Bokeh, Altair, hvplot,...)
to generate the plots. Some libraries implementing a backend for pandas are listed
on `the ecosystem page](https://pandas.pydata.org/community/ecosystem.html).

Developers guide can be found at
https://pandas.pydata.org/docs/dev/development/extending.html#plotting-backends

---

# User-Defined Functions (UDFs)
In pandas, User-Defined Functions (UDFs) provide a way to extend the library’s
functionality by allowing users to apply custom computations to their data. While
pandas comes with a set of built-in functions for data manipulation, UDFs offer
flexibility when built-in methods are not sufficient. These functions can be
applied at different levels: element-wise, row-wise, column-wise, or group-wise,
and behave differently, depending on the method used.

Here’s a simple example to illustrate a UDF applied to a Series:

```python
s = pd.Series([1, 2, 3])

# Simple UDF that adds 1 to a value
def add_one(x):
    return x + 1

# Apply the function element-wise using .map
s.map(add_one)
```
## Why Not To Use User-Defined Functions
While UDFs provide flexibility, they come with significant drawbacks, primarily
related to performance and behavior. When using UDFs, pandas must perform inference
on the result, and that inference could be incorrect. Furthermore, unlike vectorized operations,
UDFs are slower because pandas can't optimize their computations, leading to
inefficient processing.

> **note.capitalize():**
    In general, most tasks can and should be accomplished using pandas’ built-in methods or vectorized operations.

Despite their drawbacks, UDFs can be helpful when:

* **Custom Computations Are Needed**: Implementing complex logic or domain-specific calculations that pandas'
  built-in methods cannot handle.
* **Extending pandas' Functionality**: Applying external libraries or specialized algorithms unavailable in pandas.
* **Handling Complex Grouped Operations**: Performing operations on grouped data that standard methods do not support.

For example:

```python
from sklearn.linear_model import LinearRegression

# Sample data
df = pd.DataFrame({
    'group': ['A', 'A', 'A', 'B', 'B', 'B'],
    'x': [1, 2, 3, 1, 2, 3],
    'y': [2, 4, 6, 1, 2, 1.5]
})

# Function to fit a model to each group
def fit_model(group):
    model = LinearRegression()
    model.fit(group[['x']], group['y'])
    group['y_pred'] = model.predict(group[['x']])
    return group

result = df.groupby('group').apply(fit_model)
```
## Methods that support User-Defined Functions
User-Defined Functions can be applied across various pandas methods:

+-------------------------------+------------------------+--------------------------+----------------------------------------------------------------------------------------------------------------------------------------------+
| Method                        | Function Input         | Function Output          | Description                                                                                                                                  |
+===============================+========================+==========================+==============================================================================================================================================+
| `udf.map`                | Scalar                 | Scalar                   | Apply a function to each element                                                                                                             |
+-------------------------------+------------------------+--------------------------+----------------------------------------------------------------------------------------------------------------------------------------------+
| `udf.apply` (axis=0)     | Column (Series)        | Column (Series)          | Apply a function to each column                                                                                                              |
+-------------------------------+------------------------+--------------------------+----------------------------------------------------------------------------------------------------------------------------------------------+
| `udf.apply` (axis=1)     | Row (Series)           | Row (Series)             | Apply a function to each row                                                                                                                 |
+-------------------------------+------------------------+--------------------------+----------------------------------------------------------------------------------------------------------------------------------------------+
| `udf.pipe`               | Series or DataFrame    | Series or DataFrame      | Chain functions together to apply to Series or Dataframe                                                                                     |
+-------------------------------+------------------------+--------------------------+----------------------------------------------------------------------------------------------------------------------------------------------+
| `udf.filter`             | Series or DataFrame    | Boolean                  | Only accepts UDFs in group by. Function is called for each group, and the group is removed from the result if the function returns ``False`` |
+-------------------------------+------------------------+--------------------------+----------------------------------------------------------------------------------------------------------------------------------------------+
| `udf.agg`                | Series or DataFrame    | Scalar or Series         | Aggregate and summarizes values, e.g., sum or custom reducer                                                                                 |
+-------------------------------+------------------------+--------------------------+----------------------------------------------------------------------------------------------------------------------------------------------+
| `udf.transform` (axis=0) | Column (Series)        | Column (Series)          | Same as `apply` with (axis=0), but it raises an exception if the function changes the shape of the data                                |
+-------------------------------+------------------------+--------------------------+----------------------------------------------------------------------------------------------------------------------------------------------+
| `udf.transform` (axis=1) | Row (Series)           | Row (Series)             | Same as `apply` with (axis=1), but it raises an exception if the function changes the shape of the data                                |
+-------------------------------+------------------------+--------------------------+----------------------------------------------------------------------------------------------------------------------------------------------+

When applying UDFs in pandas, it is essential to select the appropriate method based
on your specific task. Each method has its strengths and is designed for different use
cases. Understanding the purpose and behavior of each method will help you make informed
decisions, ensuring more efficient and maintainable code.

> **note.capitalize():**
    Some of these methods are can also be applied to groupby, resample, and various window objects.
    See `groupby`, `resample()<timeseries>`, `rolling()<window>`, `expanding()<window>`,
    and `ewm()<window>[ for details.




### `Series.map` and `DataFrame.map`
The `map` method is used specifically to apply element-wise UDFs. This means the function
will be called for each element in the ``Series`` or ``DataFrame``, with the individual value or
the cell as the function argument.

```python
temperature_celsius = pd.DataFrame({
    "NYC": [14, 21, 23],
    "Los Angeles": [22, 28, 31],
})

def to_fahrenheit(value):
    return value * (9 / 5) + 32

temperature_celsius.map(to_fahrenheit)
```
In this example, the function ``to_fahrenheit`` will be called 6 times, once for each value
in the ``DataFrame``. And the result of each call will be returned in the corresponding cell
of the resulting ``DataFrame``.

In general, ``map`` will be slow, as it will not make use of vectorization. Instead, a Python
function call for each value will be required, which will slow down things significantly if
working with medium or large data.

When to use: Use `map` for applying element-wise UDFs to DataFrames or Series.



### `Series.apply` and `DataFrame.apply`
The `apply` method allows you to apply UDFs for a whole column or row. This is different
from `map` in that the function will be called for each column (or row), not for each individual value.

```python
temperature_celsius = pd.DataFrame({
    "NYC": [14, 21, 23],
    "Los Angeles": [22, 28, 31],
})

def to_fahrenheit(column):
    return column * (9 / 5) + 32

temperature_celsius.apply(to_fahrenheit)
```
In the example, ``to_fahrenheit`` will be called only twice, as opposed to the 6 times with `map`.
This will be faster than using `map`, since the operations for each column are vectorized, and the
overhead of iterating over data in Python and calling Python functions is significantly reduced.

In some cases, the function may require all the data to be able to compute the result. So `apply`
is needed, since with `map` the function can only access one element at a time.

```python
temperature = pd.DataFrame({
    "NYC": [14, 21, 23],
    "Los Angeles": [22, 28, 31],
})

def normalize(column):
    return column / column.mean()

temperature.apply(normalize)
```
In the example, the ``normalize`` function needs to compute the mean of the whole column in order
to divide each element by it. So, we cannot call the function for each element, but we need the
function to receive the whole column.

`apply` can also execute function by row, by specifying ``axis=1``.

```python
temperature = pd.DataFrame({
    "NYC": [14, 21, 23],
    "Los Angeles": [22, 28, 31],
})

def hotter(row):
    return row["Los Angeles"] - row["NYC"]

temperature.apply(hotter, axis=1)
```
In the example, the function ``hotter`` will be called 3 times, once for each row. And each
call will receive the whole row as the argument, allowing computations that require more than
one value in the row.

``apply`` is also available for `SeriesGroupBy.apply`, `DataFrameGroupBy.apply`,
`Rolling.apply`, `Expanding.apply` and `Resampler.apply`. You can read more
about ``apply`` in groupby operations `groupby.apply`.

When to use: `apply` is suitable when no alternative vectorized method or UDF method is available,
but consider optimizing performance with vectorized operations wherever possible.



### `Series.pipe` and `DataFrame.pipe`
The ``pipe`` method is similar to ``map`` and ``apply``, but the function receives the whole ``Series``
or ``DataFrame`` it is called on.

```python
temperature = pd.DataFrame({
    "NYC": [14, 21, 23],
    "Los Angeles": [22, 28, 31],
})

def normalize(df):
    return df / df.mean().mean()

temperature.pipe(normalize)
```
This is equivalent to calling the ``normalize`` function with the ``DataFrame`` as the parameter.

```python
normalize(temperature)
```
The main advantage of using ``pipe`` is readability. It allows method chaining and clearer code when
calling multiple functions.

```python
temperature_celsius = pd.DataFrame({
    "NYC": [14, 21, 23],
    "Los Angeles": [22, 28, 31],
})

def multiply_by_9(value):
    return value * 9

def divide_by_5(value):
    return value / 5

def add_32(value):
    return value + 32

# Without `pipe`:
fahrenheit = add_32(divide_by_5(multiply_by_9(temperature_celsius)))

# With `pipe`:
fahrenheit = (temperature_celsius.pipe(multiply_by_9)
                                 .pipe(divide_by_5)
                                 .pipe(add_32))
```
``pipe`` is also available for `SeriesGroupBy.pipe`, `DataFrameGroupBy.pipe` and
`Resampler.pipe`. You can read more about ``pipe`` in groupby operations in `groupby.pipe`.

When to use: Use `pipe` when you need to create a pipeline of operations and want to keep the code readable and maintainable.



### `Series.filter` and `DataFrame.filter`
The ``filter`` method is used to select a subset of rows that match certain criteria.
`Series.filter` and `DataFrame.filter` do not support user defined functions,
but `SeriesGroupBy.filter` and `DataFrameGroupBy.filter` do. You can read more
about ``filter`` in groupby operations in `groupby.filter`.



### `Series.agg` and `DataFrame.agg`
The ``agg`` method is used to aggregate a set of data points into a single one.
The most common aggregation functions such as ``min``, ``max``, ``mean``, ``sum``, etc.
are already implemented in pandas. ``agg`` allows to implement other custom aggregate
functions.

```python
temperature = pd.DataFrame({
    "NYC": [14, 21, 23],
    "Los Angeles": [22, 28, 31],
})

def highest_jump(column):
    return column.pct_change().max()

temperature.agg(highest_jump)
```
When to use: Use `agg` for performing custom aggregations, where the operation returns
a scalar value on each input.



### `Series.transform` and `DataFrame.transform`
The ``transform``` method is similar to an aggregation, with the difference that the result is broadcasted
to the original data.

```python
temperature = pd.DataFrame({
    "NYC": [14, 21, 23],
    "Los Angeles": [22, 28, 31]},
    index=pd.date_range("2000-01-01", "2000-01-03"))

def warm_up_all_days(column):
    return pd.Series(column.max(), index=column.index)

temperature.transform(warm_up_all_days)
```
In the example, the ``warm_up_all_days`` function computes the ``max`` like an aggregation, but instead
of returning just the maximum value, it returns a ``DataFrame`` with the same shape as the original one
with the values of each day replaced by the maximum temperature of the city.

``transform`` is also available for `SeriesGroupBy.transform`, `DataFrameGroupBy.transform` and
`Resampler.transform`, where it's more common. You can read more about ``transform`` in groupby
operations in `groupby.transform`.

When to use: When you need to perform an aggregation that will be returned in the original structure of
the DataFrame.


## Performance
While UDFs provide flexibility, their use is generally discouraged as they can introduce
performance issues, especially when written in pure Python. To improve efficiency,
consider using built-in ``NumPy`` or ``pandas`` functions instead of UDFs
for common operations.

> **note.capitalize():**
    If performance is critical, explore **vectorized operations** before resorting
    to UDFs.

### Vectorized Operations
Below is a comparison of using UDFs versus using Vectorized Operations:

```python
# User-defined function
def calc_ratio(row):
    return 100 * (row["one"] / row["two"])

df["new_col"] = df.apply(calc_ratio, axis=1)

# Vectorized Operation
df["new_col2"] = 100 * (df["one"] / df["two"])
```
Measuring how long each operation takes:



    User-defined function:  5.6435 secs
    Vectorized:             0.0043 secs

Vectorized operations in pandas are significantly faster than using `DataFrame.apply`
with UDFs because they leverage highly optimized C functions
via ``NumPy`` to process entire arrays at once. This approach avoids the overhead of looping
through rows in Python and making separate function calls for each row, which is slow and
inefficient. Additionally, ``NumPy`` arrays benefit from memory efficiency and CPU-level
optimizations, making vectorized operations the preferred choice whenever possible.


### Improving Performance with UDFs
In scenarios where UDFs are necessary, there are still ways to mitigate their performance drawbacks.
One approach is to use **Numba**, a Just-In-Time (JIT) compiler that can significantly speed up numerical
Python code by compiling Python functions to optimized machine code at runtime.

By annotating your UDFs with ``@numba.jit``, you can achieve performance closer to vectorized operations,
especially for computationally heavy tasks.

> **note.capitalize():**
    You may also refer to the user guide on `Enhancing performance](https://pandas.pydata.org/pandas-docs/dev/user_guide/enhancingperf.html#numba-jit-compilation)
    for a more detailed guide to using **Numba**.

### Using `DataFrame.pipe` for Composable Logic
Another useful pattern for improving readability and composability, especially when mixing
vectorized logic with UDFs, is to use the `DataFrame.pipe` method.

`DataFrame.pipe` doesn't improve performance directly, but it enables cleaner
method chaining by passing the entire object into a function. This is especially helpful
when chaining custom transformations:

```python
def add_ratio_column(df):
    df["ratio"] = 100 * (df["one"] / df["two"])
    return df

df = (
    df
    .query("one > 0")
    .pipe(add_ratio_column)
    .dropna()
)
```
This is functionally equivalent to calling ``add_ratio_column(df)``, but keeps your code
clean and composable. The function you pass to `DataFrame.pipe` can use vectorized operations,
row-wise UDFs, or any other logic; `DataFrame.pipe` is agnostic.

> **note.capitalize():**
    While `DataFrame.pipe` does not improve performance on its own,
    it promotes clean, modular design and allows both vectorized and UDF-based logic
    to be composed in method chains.

---

# Group by: split-apply-combine
By "group by" we are referring to a process involving one or more of the following
steps:

* **Splitting** the data into groups based on some criteria.
* **Applying** a function to each group independently.
* **Combining** the results into a data structure.

Out of these, the split step is the most straightforward. In the apply step, we
might wish to do one of the following:

* **Aggregation**: compute a summary statistic (or statistics) for each
  group. Some examples:

    * Compute group sums or means.
    * Compute group sizes / counts.

* **Transformation**: perform some group-specific computations and return a
  like-indexed object. Some examples:

    * Standardize data (zscore) within a group.
    * Filling NAs within groups with a value derived from each group.

* **Filtration**: discard some groups, according to a group-wise computation
  that evaluates to True or False. Some examples:

    * Discard data that belong to groups with only a few members.
    * Filter out data based on the group sum or mean.

Many of these operations are defined on GroupBy objects. These operations are similar
to those of the `aggregating API <basics.aggregate>`,
`window API <window.overview>`, and `resample API <timeseries.aggregate>`.

It is possible that a given operation does not fall into one of these categories or
is some combination of them. In such a case, it may be possible to compute the
operation using GroupBy's ``apply`` method. This method will examine the results of the
apply step and try to sensibly combine them into a single result if it doesn't fit into either
of the above three categories.

> **note.capitalize():**
   An operation that is split into multiple steps using built-in GroupBy operations
   will be more efficient than using the ``apply`` method with a user-defined Python
   function.


The name GroupBy should be quite familiar to those who have used
a SQL-based tool (or ``itertools``), in which you can write code like:



   SELECT Column1, Column2, mean(Column3), sum(Column4)
   FROM SomeTable
   GROUP BY Column1, Column2

We aim to make operations like this natural and easy to express using
pandas. We'll address each area of GroupBy functionality, then provide some
non-trivial examples / use cases.

See the `cookbook<cookbook.grouping>` for some advanced strategies.



## Splitting an object into groups
The abstract definition of grouping is to provide a mapping of labels to
group names. To create a GroupBy object (more on what the GroupBy object is
later), you may do the following:

```python
speeds = pd.DataFrame(
    [
        ("bird", "Falconiformes", 389.0),
        ("bird", "Psittaciformes", 24.0),
        ("mammal", "Carnivora", 80.2),
        ("mammal", "Primates", np.nan),
        ("mammal", "Carnivora", 58),
    ],
    index=["falcon", "parrot", "lion", "monkey", "leopard"],
    columns=("class", "order", "max_speed"),
)
speeds

grouped = speeds.groupby("class")
grouped = speeds.groupby(["class", "order"])
```
The mapping can be specified many different ways:

* A Python function, to be called on each of the index labels.
* A list or NumPy array of the same length as the index.
* A dict or ``Series``, providing a ``label -> group name`` mapping.
* For ``DataFrame`` objects, a string indicating either a column name or
  an index level name to be used to group.
* A list of any of the above things.

Collectively we refer to the grouping objects as the **keys**. For example,
consider the following ``DataFrame``:

> **note.capitalize():**
   A string passed to ``groupby`` may refer to either a column or an index level.
   If a string matches both a column name and an index level name, a
   ``ValueError`` will be raised.

```python
df = pd.DataFrame(
    {
        "A": ["foo", "bar", "foo", "bar", "foo", "bar", "foo", "foo"],
        "B": ["one", "one", "two", "three", "two", "two", "one", "three"],
        "C": np.random.randn(8),
        "D": np.random.randn(8),
    }
)
df
```
On a DataFrame, we obtain a GroupBy object by calling `~DataFrame.groupby`.
This method returns a ``pandas.api.typing.DataFrameGroupBy`` instance.
We could naturally group by either the ``A`` or ``B`` columns, or both:

```python
grouped = df.groupby("A")
grouped = df.groupby("B")
grouped = df.groupby(["A", "B"])
```
> **note.capitalize():**
   ``df.groupby('A')`` is just syntactic sugar for ``df.groupby(df['A'])``.

The above GroupBy will split the DataFrame on its index (rows). To split by columns, first do
a transpose:



    In [4]: def get_letter_type(letter):
       ...:     if letter.lower() in 'aeiou':
       ...:         return 'vowel'
       ...:     else:
       ...:         return 'consonant'
       ...:

    In [5]: grouped = df.T.groupby(get_letter_type)

pandas `~pandas.Index` objects support duplicate values. If a
non-unique index is used as the group key in a groupby operation, all values
for the same index value will be considered to be in one group and thus the
output of aggregation functions will only contain unique index values:

```python
index = [1, 2, 3, 1, 2, 3]

s = pd.Series([1, 2, 3, 10, 20, 30], index=index)

s

grouped = s.groupby(level=0)

grouped.first()

grouped.last()

grouped.sum()
```
Note that **no splitting occurs** until it's needed. Creating the GroupBy object
only verifies that you've passed a valid mapping.

> **note.capitalize():**
   Many kinds of complicated data manipulations can be expressed in terms of
   GroupBy operations (though it can't be guaranteed to be the most efficient implementation).
   You can get quite creative with the label mapping functions.



### GroupBy sorting
By default the group keys are sorted during the ``groupby`` operation. You may however pass ``sort=False`` for potential speedups. With ``sort=False`` the order among group-keys follows the order of appearance of the keys in the original dataframe:

```python
df2 = pd.DataFrame({"X": ["B", "B", "A", "A"], "Y": [1, 2, 3, 4]})
df2.groupby(["X"]).sum()
df2.groupby(["X"], sort=False).sum()
```
Note that ``groupby`` will preserve the order in which *observations* are sorted *within* each group.
For example, the groups created by ``groupby()`` below are in the order they appeared in the original ``DataFrame``:

```python
df3 = pd.DataFrame({"X": ["A", "B", "A", "B"], "Y": [1, 4, 3, 2]})
df3.groupby("X").get_group("A")

df3.groupby(["X"]).get_group(("B",))
```


#### GroupBy dropna
By default ``NA`` values are excluded from group keys during the ``groupby`` operation. However,
in case you want to include ``NA`` values in group keys, you could pass ``dropna=False`` to achieve it.

```python
df_list = [[1, 2, 3], [1, None, 4], [2, 1, 3], [1, 2, 2]]
df_dropna = pd.DataFrame(df_list, columns=["a", "b", "c"])

df_dropna
```
```python
# Default ``dropna`` is set to True, which will exclude NaNs in keys
df_dropna.groupby(by=["b"], dropna=True).sum()

# In order to allow NaN in keys, set ``dropna`` to False
df_dropna.groupby(by=["b"], dropna=False).sum()
```
The default setting of ``dropna`` argument is ``True`` which means ``NA`` are not included in group keys.




### GroupBy object attributes
The ``groups`` attribute is a dictionary whose keys are the computed unique groups
and corresponding values are the index labels belonging to each group. In the
above example we have:

```python
df.groupby("A").groups
df.T.groupby(get_letter_type).groups
```
Calling the standard Python ``len`` function on the GroupBy object returns
the number of groups, which is the same as the length of the ``groups`` dictionary:

```python
grouped = df.groupby(["A", "B"])
grouped.groups
len(grouped)
```


``GroupBy`` will tab complete column names, GroupBy operations, and other attributes:

```python
n = 10
weight = np.random.normal(166, 20, size=n)
height = np.random.normal(60, 10, size=n)
time = pd.date_range("1/1/2000", periods=n)
gender = np.random.choice(["male", "female"], size=n)
df = pd.DataFrame(
    {"height": height, "weight": weight, "gender": gender}, index=time
)
df
gb = df.groupby("gender")
```


   @verbatim
   In [1]: gb.<TAB>  # noqa: E225, E999
   gb.agg        gb.boxplot    gb.cummin     gb.describe   gb.filter     gb.get_group  gb.height     gb.last       gb.median     gb.ngroups    gb.plot       gb.rank       gb.std        gb.transform
   gb.aggregate  gb.count      gb.cumprod    gb.dtype      gb.first      gb.groups     gb.hist       gb.max        gb.min        gb.nth        gb.prod       gb.resample   gb.sum        gb.var
   gb.apply      gb.cummax     gb.cumsum     gb.gender     gb.head       gb.indices    gb.mean       gb.name       gb.ohlc       gb.quantile   gb.size       gb.tail       gb.weight



### GroupBy with MultiIndex
With `hierarchically-indexed data <advanced.hierarchical>`, it's quite
natural to group by one of the levels of the hierarchy.

Let's create a Series with a two-level ``MultiIndex``.

```python
arrays = [
    ["bar", "bar", "baz", "baz", "foo", "foo", "qux", "qux"],
    ["one", "two", "one", "two", "one", "two", "one", "two"],
]
index = pd.MultiIndex.from_arrays(arrays, names=["first", "second"])
s = pd.Series(np.random.randn(8), index=index)
s
```
We can then group by one of the levels in ``s``.

```python
grouped = s.groupby(level=0)
grouped.sum()
```
If the MultiIndex has names specified, these can be passed instead of the level
number:

```python
s.groupby(level="second").sum()
```
Grouping with multiple levels is supported.

```python
arrays = [
    ["bar", "bar", "baz", "baz", "foo", "foo", "qux", "qux"],
    ["doo", "doo", "bee", "bee", "bop", "bop", "bop", "bop"],
    ["one", "two", "one", "two", "one", "two", "one", "two"],
]
index = pd.MultiIndex.from_arrays(arrays, names=["first", "second", "third"])
s = pd.Series(np.random.randn(8), index=index)
s
s.groupby(level=["first", "second"]).sum()
```
Index level names may be supplied as keys.

```python
s.groupby(["first", "second"]).sum()
```
More on the ``sum`` function and aggregation later.

### Grouping DataFrame with Index levels and columns
A DataFrame may be grouped by a combination of columns and index levels. You
can specify both column and index names, or use a `Grouper`.

Let's first create a DataFrame with a MultiIndex:

```python
arrays = [
    ["bar", "bar", "baz", "baz", "foo", "foo", "qux", "qux"],
    ["one", "two", "one", "two", "one", "two", "one", "two"],
]

index = pd.MultiIndex.from_arrays(arrays, names=["first", "second"])

df = pd.DataFrame({"A": [1, 1, 1, 1, 2, 2, 3, 3], "B": np.arange(8)}, index=index)

df
```
Then we group ``df`` by the ``second`` index level and the ``A`` column.

```python
df.groupby([pd.Grouper(level=1), "A"]).sum()
```
Index levels may also be specified by name.

```python
df.groupby([pd.Grouper(level="second"), "A"]).sum()
```
Index level names may be specified as keys directly to ``groupby``.

```python
df.groupby(["second", "A"]).sum()
```
### DataFrame column selection in GroupBy
Once you have created the GroupBy object from a DataFrame, you might want to do
something different for each of the columns. Thus, by using ``[]`` on the GroupBy
object in a similar way as the one used to get a column from a DataFrame, you can do:

```python
df = pd.DataFrame(
    {
        "A": ["foo", "bar", "foo", "bar", "foo", "bar", "foo", "foo"],
        "B": ["one", "one", "two", "three", "two", "two", "one", "three"],
        "C": np.random.randn(8),
        "D": np.random.randn(8),
    }
)

df

grouped = df.groupby(["A"])
grouped_C = grouped["C"]
grouped_D = grouped["D"]
```
This is mainly syntactic sugar for the alternative, which is much more verbose:

```python
df["C"].groupby(df["A"])
```
Additionally, this method avoids recomputing the internal grouping information
derived from the passed key.

You can also include the grouping columns if you want to operate on them.

```python
grouped[["A", "B"]].sum()
```
> **note.capitalize():**
   The ``groupby`` operation in pandas drops the ``name`` field of the columns Index object
   after the operation. This change ensures consistency in syntax between different
   column selection methods within groupby operations.



## Iterating through groups
With the GroupBy object in hand, iterating through the grouped data is very
natural and functions similarly to :py`itertools.groupby`:



   In [4]: grouped = df.groupby('A')

   In [5]: for name, group in grouped:
      ...:     print(name)
      ...:     print(group)
      ...:

In the case of grouping by multiple keys, the group name will be a tuple:



   In [5]: for name, group in df.groupby(['A', 'B']):
      ...:     print(name)
      ...:     print(group)
      ...:

See `timeseries.iterating-label`.

## Selecting a group
A single group can be selected using
`.DataFrameGroupBy.get_group`:

```python
grouped.get_group("bar")
```
Or for an object grouped on multiple columns:

```python
df.groupby(["A", "B"]).get_group(("bar", "one"))
```


## Aggregation
An aggregation is a GroupBy operation that reduces the dimension of the grouping
object. The result of an aggregation is, or at least is treated as,
a scalar value for each column in a group. For example, producing the sum of each
column in a group of values.

```python
animals = pd.DataFrame(
    {
        "kind": ["cat", "dog", "cat", "dog"],
        "height": [9.1, 6.0, 9.5, 34.0],
        "weight": [7.9, 7.5, 9.9, 198.0],
    }
)
animals
animals.groupby("kind").sum()
```
In the result, the keys of the groups appear in the index by default. They can be
instead included in the columns by passing ``as_index=False``.

```python
animals.groupby("kind", as_index=False).sum()
```


### Built-in aggregation methods
Many common aggregations are built-in to GroupBy objects as methods. Of the methods
listed below, those with a ``*`` do *not* have an efficient, GroupBy-specific, implementation.


    :header: "Method", "Description"
    :widths: 20, 80

        `~.DataFrameGroupBy.any`,Compute whether any of the values in the groups are truthy
        `~.DataFrameGroupBy.all`,Compute whether all of the values in the groups are truthy
        `~.DataFrameGroupBy.count`,Compute the number of non-NA values in the groups
        `~.DataFrameGroupBy.cov` * ,Compute the covariance of the groups
        `~.DataFrameGroupBy.first`,Compute the first occurring value in each group
        `~.DataFrameGroupBy.idxmax`,Compute the index of the maximum value in each group
        `~.DataFrameGroupBy.idxmin`,Compute the index of the minimum value in each group
        `~.DataFrameGroupBy.last`,Compute the last occurring value in each group
        `~.DataFrameGroupBy.max`,Compute the maximum value in each group
        `~.DataFrameGroupBy.mean`,Compute the mean of each group
        `~.DataFrameGroupBy.median`,Compute the median of each group
        `~.DataFrameGroupBy.min`,Compute the minimum value in each group
        `~.DataFrameGroupBy.nunique`,Compute the number of unique values in each group
        `~.DataFrameGroupBy.prod`,Compute the product of the values in each group
        `~.DataFrameGroupBy.quantile`,Compute a given quantile of the values in each group
        `~.DataFrameGroupBy.sem`,Compute the standard error of the mean of the values in each group
        `~.DataFrameGroupBy.size`,Compute the number of values in each group
        `~.DataFrameGroupBy.skew` * ,Compute the skew of the values in each group
        `~.DataFrameGroupBy.std`,Compute the standard deviation of the values in each group
        `~.DataFrameGroupBy.sum`,Compute the sum of the values in each group
        `~.DataFrameGroupBy.var`,Compute the variance of the values in each group

Some examples:

```python
df.groupby("A")[["C", "D"]].max()
df.groupby(["A", "B"]).mean()
```
Another aggregation example is to compute the size of each group.
This is included in GroupBy as the ``size`` method. It returns a Series whose
index consists of the group names and the values are the sizes of each group.

```python
grouped = df.groupby(["A", "B"])
grouped.size()
```
While the `.DataFrameGroupBy.describe` method is not itself a reducer, it
can be used to conveniently produce a collection of summary statistics about each of
the groups.

```python
grouped.describe()
```
Another aggregation example is to compute the number of unique values of each group.
This is similar to the `.DataFrameGroupBy.value_counts` function, except that it only counts the
number of unique values.

```python
ll = [['foo', 1], ['foo', 2], ['foo', 2], ['bar', 1], ['bar', 1]]
df4 = pd.DataFrame(ll, columns=["A", "B"])
df4
df4.groupby("A")["B"].nunique()
```
> **note.capitalize():**
   Aggregation functions **will not** return the groups that you are aggregating over
   as named *columns* when ``as_index=True``, the default. The grouped columns will
   be the **indices** of the returned object.

   Passing ``as_index=False`` **will** return the groups that you are aggregating over as
   named columns, regardless if they are named **indices** or *columns* in the inputs.




### The `~.DataFrameGroupBy.aggregate` method
> **note.capitalize():**
    The `~.DataFrameGroupBy.aggregate` method can accept many different types of
    inputs. This section details using string aliases for various GroupBy methods; other
    inputs are detailed in the sections below.

Any reduction method that pandas implements can be passed as a string to
`~.DataFrameGroupBy.aggregate`. Users are encouraged to use the shorthand,
``agg``. It will operate as if the corresponding method was called.

```python
grouped = df.groupby("A")
grouped[["C", "D"]].aggregate("sum")

grouped = df.groupby(["A", "B"])
grouped.agg("sum")
```
The result of the aggregation will have the group names as the
new index. In the case of multiple keys, the result is a
`MultiIndex <advanced.hierarchical>` by default. As mentioned above, this can be
changed by using the ``as_index`` option:

```python
grouped = df.groupby(["A", "B"], as_index=False)
grouped.agg("sum")

df.groupby("A", as_index=False)[["C", "D"]].agg("sum")
```
Note that you could use the `DataFrame.reset_index` DataFrame function to achieve
the same result as the column names are stored in the resulting ``MultiIndex``, although
this will make an extra copy.

```python
df.groupby(["A", "B"]).agg("sum").reset_index()
```


### Aggregation with user-defined functions
Users can also provide their own User-Defined Functions (UDFs) for custom aggregations.

> **warning.capitalize():**
    When aggregating with a UDF, the UDF should not mutate the
    provided ``Series``. See `gotchas.udf-mutation` for more information.

> **note.capitalize():**
    Aggregating with a UDF is often less performant than using
    the pandas built-in methods on GroupBy. Consider breaking up a complex operation
    into a chain of operations that utilize the built-in methods.

```python
animals
animals.groupby("kind")[["height"]].agg(lambda x: set(x))
```
The resulting dtype will reflect that of the aggregating function. If the results from different groups have
different dtypes, then a common dtype will be determined in the same way as ``DataFrame`` construction.

```python
animals.groupby("kind")[["height"]].agg(lambda x: x.astype(int).sum())
```


### Applying multiple functions at once
On a grouped ``Series``, you can pass a list or dict of functions to
`SeriesGroupBy.agg`, outputting a DataFrame:

```python
grouped = df.groupby("A")
grouped["C"].agg(["sum", "mean", "std"])
```
On a grouped ``DataFrame``, you can pass a list of functions to
`DataFrameGroupBy.agg` to aggregate each
column, which produces an aggregated result with a hierarchical column index:

```python
grouped[["C", "D"]].agg(["sum", "mean", "std"])
```
The resulting aggregations are named after the functions themselves.

For a ``Series``, if you need to rename, you can add in a chained operation like this:

```python
(
    grouped["C"]
    .agg(["sum", "mean", "std"])
    .rename(columns={"sum": "foo", "mean": "bar", "std": "baz"})
)
```
Or, you can simply pass a list of tuples each with the name of the new column and the aggregate function:

```python
(
   grouped["C"]
   .agg([("foo", "sum"), ("bar", "mean"), ("baz", "std")])
)
```
For a grouped ``DataFrame``, you can rename in a similar manner:

By chaining ``rename`` operation,

```python
(
    grouped[["C", "D"]].agg(["sum", "mean", "std"]).rename(
        columns={"sum": "foo", "mean": "bar", "std": "baz"}
    )
)
```
Or, passing a list of tuples,

```python
(
   grouped[["C", "D"]].agg(
      [("foo", "sum"), ("bar", "mean"), ("baz", "std")]
   )
)
```
> **note.capitalize():**
   In general, the output column names should be unique, but pandas will allow
   you apply to the same function (or two functions with the same name) to the same
   column.

   ```python
grouped["C"].agg(["sum", "sum"])


pandas also allows you to provide multiple lambdas. In this case, pandas
will mangle the name of the (nameless) lambda functions, appending ``_<i>``
to each subsequent lambda.



   grouped["C"].agg([lambda x: x.max() - x.min(), lambda x: x.median() - x.mean()])
```


### Named aggregation
To support column-specific aggregation *with control over the output column names*, pandas
accepts the special syntax in `.DataFrameGroupBy.agg` and `.SeriesGroupBy.agg`, known as "named aggregation", where

- The keywords are the *output* column names
- The values are tuples whose first element is the column to select
  and the second element is the aggregation to apply to that column. pandas
  provides the `NamedAgg` namedtuple with the fields ``['column', 'aggfunc']``
  to make it clearer what the arguments are. As usual, the aggregation can
  be a callable or a string alias.

```python
animals

animals.groupby("kind").agg(
    min_height=pd.NamedAgg(column="height", aggfunc="min"),
    max_height=pd.NamedAgg(column="height", aggfunc="max"),
    average_weight=pd.NamedAgg(column="weight", aggfunc="mean"),
)
```
`NamedAgg` is just a ``namedtuple``. Plain tuples are allowed as well.

```python
animals.groupby("kind").agg(
    min_height=("height", "min"),
    max_height=("height", "max"),
    average_weight=("weight", "mean"),
)
```
If the column names you want are not valid Python keywords, construct a dictionary
and unpack the keyword arguments

```python
animals.groupby("kind").agg(
    **{
        "total weight": pd.NamedAgg(column="weight", aggfunc="sum")
    }
)
```
When using named aggregation, additional keyword arguments are not passed through
to the aggregation functions; only pairs
of ``(column, aggfunc)`` should be passed as ``**kwargs``. If your aggregation functions
require additional arguments, apply them partially with `functools.partial`.

Named aggregation is also valid for Series groupby aggregations. In this case there's
no column selection, so the values are just the functions.

```python
animals.groupby("kind").height.agg(
    min_height="min",
    max_height="max",
)
```
### Applying different functions to DataFrame columns
By passing a dict to ``aggregate`` you can apply a different aggregation to the
columns of a DataFrame:

```python
grouped.agg({"C": "sum", "D": lambda x: np.std(x, ddof=1)})
```
The function names can also be strings. In order for a string to be valid it
must be implemented on GroupBy:

```python
grouped.agg({"C": "sum", "D": "std"})
```


## Transformation
A transformation is a GroupBy operation whose result is indexed the same
as the one being grouped. Common examples include `~.DataFrameGroupBy.cumsum` and
`~.DataFrameGroupBy.diff`.

```python
speeds
grouped = speeds.groupby("class")["max_speed"]
grouped.cumsum()
grouped.diff()
```
Unlike aggregations, the groupings that are used to split
the original object are not included in the result.

> **note.capitalize():**
    Since transformations do not include the groupings that are used to split the result,
    the arguments ``as_index`` and ``sort`` in `DataFrame.groupby` and
    `Series.groupby` have no effect.

A common use of a transformation is to add the result back into the original DataFrame.

```python
result = speeds.copy()
result["cumsum"] = grouped.cumsum()
result["diff"] = grouped.diff()
result
```
### Built-in transformation methods
The following methods on GroupBy act as transformations.


    :header: "Method", "Description"
    :widths: 20, 80

        `~.DataFrameGroupBy.bfill`,Back fill NA values within each group
        `~.DataFrameGroupBy.cumcount`,Compute the cumulative count within each group
        `~.DataFrameGroupBy.cummax`,Compute the cumulative max within each group
        `~.DataFrameGroupBy.cummin`,Compute the cumulative min within each group
        `~.DataFrameGroupBy.cumprod`,Compute the cumulative product within each group
        `~.DataFrameGroupBy.cumsum`,Compute the cumulative sum within each group
        `~.DataFrameGroupBy.diff`,Compute the difference between adjacent values within each group
        `~.DataFrameGroupBy.ffill`,Forward fill NA values within each group
        `~.DataFrameGroupBy.pct_change`,Compute the percent change between adjacent values within each group
        `~.DataFrameGroupBy.rank`,Compute the rank of each value within each group
        `~.DataFrameGroupBy.shift`,Shift values up or down within each group

In addition, passing any built-in aggregation method as a string to
`~.DataFrameGroupBy.transform` (see the next section) will broadcast the result
across the group, producing a transformed result. If the aggregation method has an efficient
implementation, this will be performant as well.



### The `~.DataFrameGroupBy.transform` method
Similar to the `aggregation method <groupby.aggregate.agg>`, the
`~.DataFrameGroupBy.transform` method can accept string aliases to the built-in
transformation methods in the previous section. It can *also* accept string aliases to
the built-in aggregation methods. When an aggregation method is provided, the result
will be broadcast across the group.

```python
speeds
grouped = speeds.groupby("class")[["max_speed"]]
grouped.transform("cumsum")
grouped.transform("sum")
```
In addition to string aliases, the `~.DataFrameGroupBy.transform` method can
also accept User-Defined Functions (UDFs). The UDF must:

* Return a result that is either the same size as the group chunk or
  broadcastable to the size of the group chunk (e.g., a scalar,
  ``grouped.transform(lambda x: x.iloc[-1])``).
* Operate column-by-column on the group chunk.  The transform is applied to
  the first group chunk using chunk.apply.
* Not perform in-place operations on the group chunk. Group chunks should
  be treated as immutable, and changes to a group chunk may produce unexpected
  results. See `gotchas.udf-mutation` for more information.
* (Optionally) operates on all columns of the entire group chunk at once. If this is
  supported, a fast path is used starting from the *second* chunk.

> **note.capitalize():**
    Transforming by supplying ``transform`` with a UDF is
    often less performant than using the built-in methods on GroupBy.
    Consider breaking up a complex operation into a chain of operations that utilize
    the built-in methods.

    All of the examples in this section can be made more performant by calling
    built-in methods instead of using UDFs.
    See `below for examples <groupby_efficient_transforms>`.



    When using ``.transform`` on a grouped DataFrame and the transformation function
    returns a DataFrame, pandas now aligns the result's index
    with the input's index. You can call ``.to_numpy()`` within the transformation
    function to avoid alignment.

Similar to `groupby.aggregate.agg`, the resulting dtype will reflect that of the
transformation function. If the results from different groups have different dtypes, then
a common dtype will be determined in the same way as ``DataFrame`` construction.

Suppose we wish to standardize the data within each group:

```python
index = pd.date_range("10/1/1999", periods=1100)
ts = pd.Series(np.random.normal(0.5, 2, 1100), index)
ts = ts.rolling(window=100, min_periods=100).mean().dropna()

ts.head()
ts.tail()

transformed = ts.groupby(lambda x: x.year).transform(
    lambda x: (x - x.mean()) / x.std()
)
```
We would expect the result to now have mean 0 and standard deviation 1 within
each group (up to floating-point error), which we can easily check:

```python
# Original Data
grouped = ts.groupby(lambda x: x.year)
grouped.mean()
grouped.std()

# Transformed Data
grouped_trans = transformed.groupby(lambda x: x.year)
grouped_trans.mean()
grouped_trans.std()
```
We can also visually compare the original and transformed data sets.

```python
compare = pd.DataFrame({"Original": ts, "Transformed": transformed})

@savefig groupby_transform_plot.png
compare.plot()
```
Transformation functions that have lower dimension outputs are broadcast to
match the shape of the input array.

```python
ts.groupby(lambda x: x.year).transform(lambda x: x.max() - x.min())
```
Another common data transform is to replace missing data with the group mean.

```python
cols = ["A", "B", "C"]
values = np.random.randn(1000, 3)
values[np.random.randint(0, 1000, 100), 0] = np.nan
values[np.random.randint(0, 1000, 50), 1] = np.nan
values[np.random.randint(0, 1000, 200), 2] = np.nan
data_df = pd.DataFrame(values, columns=cols)
data_df

countries = np.array(["US", "UK", "GR", "JP"])
key = countries[np.random.randint(0, 4, 1000)]

grouped = data_df.groupby(key)

# Non-NA count in each group
grouped.count()

transformed = grouped.transform(lambda x: x.fillna(x.mean()))
```
We can verify that the group means have not changed in the transformed data,
and that the transformed data contains no NAs.

```python
grouped_trans = transformed.groupby(key)

grouped.mean()  # original group means
grouped_trans.mean()  # transformation did not change group means

grouped.count()  # original has some missing data points
grouped_trans.count()  # counts after transformation
grouped_trans.size()  # Verify non-NA count equals group size
```


As mentioned in the note above, each of the examples in this section can be computed
more efficiently using built-in methods. In the code below, the inefficient way
using a UDF is commented out and the faster alternative appears below.

```python
# result = ts.groupby(lambda x: x.year).transform(
#     lambda x: (x - x.mean()) / x.std()
# )
grouped = ts.groupby(lambda x: x.year)
result = (ts - grouped.transform("mean")) / grouped.transform("std")

# result = ts.groupby(lambda x: x.year).transform(lambda x: x.max() - x.min())
grouped = ts.groupby(lambda x: x.year)
result = grouped.transform("max") - grouped.transform("min")

# grouped = data_df.groupby(key)
# result = grouped.transform(lambda x: x.fillna(x.mean()))
grouped = data_df.groupby(key)
result = data_df.fillna(grouped.transform("mean"))
```


### Window and resample operations
It is possible to use ``resample()``, ``expanding()`` and
``rolling()`` as methods on groupbys.

The example below will apply the ``rolling()`` method on the samples of
the column B, based on the groups of column A.

```python
df_re = pd.DataFrame({"A": [1] * 10 + [5] * 10, "B": np.arange(20)})
df_re

df_re.groupby("A").rolling(4).B.mean()
```
The ``expanding()`` method will accumulate a given operation
(``sum()`` in the example) for all the members of each particular
group.

```python
df_re.groupby("A").expanding().sum()
```
Suppose you want to use the ``resample()`` method to get a daily
frequency in each group of your dataframe, and wish to complete the
missing values with the ``ffill()`` method.

```python
df_re = pd.DataFrame(
    {
        "date": pd.date_range(start="2016-01-01", periods=4, freq="W"),
        "group": [1, 1, 2, 2],
        "val": [5, 6, 7, 8],
    }
).set_index("date")
df_re

df_re.groupby("group").resample("1D").ffill()
```


## Filtration
A filtration is a GroupBy operation that subsets the original grouping object. It
may either filter out entire groups, part of groups, or both. Filtrations return
a filtered version of the calling object, including the grouping columns when provided.
In the following example, ``class`` is included in the result.

```python
speeds
speeds.groupby("class").nth(1)
```
> **note.capitalize():**
    Unlike aggregations, filtrations do not add the group keys to the index of the
    result. Because of this, passing ``as_index=False`` or ``sort=True`` will not
    affect these methods.

Filtrations will respect subsetting the columns of the GroupBy object.

```python
speeds.groupby("class")[["order", "max_speed"]].nth(1)
```
### Built-in filtrations
The following methods on GroupBy act as filtrations. All these methods have an
efficient, GroupBy-specific, implementation.


    :header: "Method", "Description"
    :widths: 20, 80

        `~.DataFrameGroupBy.head`,Select the top row(s) of each group
        `~.DataFrameGroupBy.nth`,Select the nth row(s) of each group
        `~.DataFrameGroupBy.tail`,Select the bottom row(s) of each group

Users can also use transformations along with Boolean indexing to construct complex
filtrations within groups. For example, suppose we are given groups of products and
their volumes, and we wish to subset the data to only the largest products capturing no
more than 90% of the total volume within each group.

```python
product_volumes = pd.DataFrame(
    {
        "group": list("xxxxyyy"),
        "product": list("abcdefg"),
        "volume": [10, 30, 20, 15, 40, 10, 20],
    }
)
product_volumes

# Sort by volume to select the largest products first
product_volumes = product_volumes.sort_values("volume", ascending=False)
grouped = product_volumes.groupby("group")["volume"]
cumpct = grouped.cumsum() / grouped.transform("sum")
cumpct
significant_products = product_volumes[cumpct <= 0.9]
significant_products.sort_values(["group", "product"])
[``
### The `~DataFrameGroupBy.filter` method
> **note.capitalize():**
    Filtering by supplying ``filter`` with a User-Defined Function (UDF) is
    often less performant than using the built-in methods on GroupBy.
    Consider breaking up a complex operation into a chain of operations that utilize
    the built-in methods.

The ``filter`` method takes a User-Defined Function (UDF) that, when applied to
an entire group, returns either ``True`` or ``False``. The result of the ``filter``
method is then the subset of groups for which the UDF returned ``True``.

Suppose we want to take only elements that belong to groups with a group sum greater
than 2.

```python
sf = pd.Series([1, 1, 2, 3, 3, 3])
sf.groupby(sf).filter(lambda x: x.sum() > 2)
```
Another useful operation is filtering out elements that belong to groups
with only a couple members.

```python
dff = pd.DataFrame({"A": np.arange(8), "B": list("aabbbbcc")})
dff.groupby("B").filter(lambda x: len(x) > 2)
```
Alternatively, instead of dropping the offending groups, we can return a
like-indexed objects where the groups that do not pass the filter are filled
with NaNs.

```python
dff.groupby("B").filter(lambda x: len(x) > 2, dropna=False)
```
For DataFrames with multiple columns, filters should explicitly specify a column as the filter criterion.

```python
dff["C"] = np.arange(8)
dff.groupby("B").filter(lambda x: len(x["C"]) > 2)
```


## Flexible ``apply``
Some operations on the grouped data might not fit into the aggregation,
transformation, or filtration categories. For these, you can use the ``apply``
function.

> **warning.capitalize():**
   ``apply`` has to try to infer from the result whether it should act as a reducer,
   transformer, *or* filter, depending on exactly what is passed to it. Thus the
   grouped column(s) may be included in the output or not. While
   it tries to intelligently guess how to behave, it can sometimes guess wrong.

> **note.capitalize():**
   All of the examples in this section can be more reliably, and more efficiently,
   computed using other pandas functionality.

```python
df
grouped = df.groupby("A")

# could also just call .describe()
grouped["C"].apply(lambda x: x.describe())
```
The dimension of the returned result can also change:

```python
grouped = df.groupby('A')['C']

def f(group):
    return pd.DataFrame({'original': group,
                         'demeaned': group - group.mean()})

grouped.apply(f)
```
``apply`` on a Series can operate on a returned value from the applied function
that is itself a series, and possibly upcast the result to a DataFrame:

```python
def f(x):
    return pd.Series([x, x ** 2], index=["x", "x^2"])


s = pd.Series(np.random.rand(5))
s
s.apply(f)
```
Similar to `groupby.aggregate.agg`, the resulting dtype will reflect that of the
apply function. If the results from different groups have different dtypes, then
a common dtype will be determined in the same way as ``DataFrame`` construction.

### Control grouped column(s) placement with ``group_keys``
To control whether the grouped column(s) are included in the indices, you can use
the argument ``group_keys`` which defaults to ``True``. Compare

```python
df.groupby("A", group_keys=True).apply(lambda x: x)
```
with

```python
df.groupby("A", group_keys=False).apply(lambda x: x)
```
## Numba accelerated routines


If `Numba](https://numba.pydata.org/)_ is installed as an optional dependency, the ``transform`` and
``aggregate`` methods support ``engine='numba'`` and ``engine_kwargs`` arguments.
See `enhancing performance with Numba <enhancingperf.numba>` for general usage of the arguments
and performance considerations.

The function signature must start with ``values, index`` **exactly** as the data belonging to each group
will be passed into ``values``, and the group index will be passed into ``index``.

> **warning.capitalize():**
   When using ``engine='numba'``, there will be no "fall back" behavior internally. The group
   data and group index will be passed as NumPy arrays to the JITed user defined function, and no
   alternative execution attempts will be tried.

## Other useful features
### Exclusion of non-numeric columns
Again consider the example DataFrame we've been looking at:

```python
df
```
Suppose we wish to compute the standard deviation grouped by the ``A``
column. There is a slight problem, namely that we don't care about the data in
column ``B`` because it is not numeric. You can avoid non-numeric columns by
specifying ``numeric_only=True``:

```python
df.groupby("A").std(numeric_only=True)
```
Note that ``df.groupby('A').colname.std().`` is more efficient than
``df.groupby('A').std().colname``. So if the result of an aggregation function
is only needed over one column (here ``colname``), it may be filtered
*before* applying the aggregation function.

```python
from decimal import Decimal

df_dec = pd.DataFrame(
    {
        "id": [1, 2, 1, 2],
        "int_column": [1, 2, 3, 4],
        "dec_column": [
            Decimal("0.50"),
            Decimal("0.15"),
            Decimal("0.25"),
            Decimal("0.40"),
        ],
    }
)
df_dec.groupby(["id"])[["dec_column"]].sum()
```


### Handling of (un)observed Categorical values
When using a ``Categorical`` grouper (as a single grouper, or as part of multiple groupers), the ``observed`` keyword
controls whether to return a cartesian product of all possible groupers values (``observed=False``) or only those
that are observed groupers (``observed=True``).

Show all values:

```python
pd.Series([1, 1, 1]).groupby(
    pd.Categorical(["a", "a", "a"], categories=["a", "b"]), observed=False
).count()
```
Show only the observed values:

```python
pd.Series([1, 1, 1]).groupby(
    pd.Categorical(["a", "a", "a"], categories=["a", "b"]), observed=True
).count()
```
The returned dtype of the grouped will *always* include *all* of the categories that were grouped.

```python
s = (
    pd.Series([1, 1, 1])
    .groupby(pd.Categorical(["a", "a", "a"], categories=["a", "b"]), observed=True)
    .count()
)
s.index.dtype
```


### NA group handling
By ``NA``, we are referring to any ``NA`` values, including
`NA`, ``NaN``, ``NaT``, and ``None``. If there are any ``NA`` values in the
grouping key, by default these will be excluded. In other words, any
"``NA`` group" will be dropped. You can include NA groups by specifying ``dropna=False``.

```python
df = pd.DataFrame({"key": [1.0, 1.0, np.nan, 2.0, np.nan], "A": [1, 2, 3, 4, 5]})
df

df.groupby("key", dropna=True).sum()

df.groupby("key", dropna=False).sum()
```
### Grouping with ordered factors
Categorical variables represented as instances of pandas's ``Categorical`` class
can be used as group keys. If so, the order of the levels will be preserved. When
``observed=False`` and ``sort=False``, any unobserved categories will be at the
end of the result in order.

```python
days = pd.Categorical(
    values=["Wed", "Mon", "Thu", "Mon", "Wed", "Sat"],
    categories=["Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"],
)
data = pd.DataFrame(
   {
       "day": days,
       "workers": [3, 4, 1, 4, 2, 2],
   }
)
data

data.groupby("day", observed=False, sort=True).sum()

data.groupby("day", observed=False, sort=False).sum()
```


### Grouping with a grouper specification
You may need to specify a bit more data to properly group. You can
use the ``pd.Grouper`` to provide this local control.

```python
import datetime

df = pd.DataFrame(
    {
        "Branch": "A A A A A A A B".split(),
        "Buyer": "Carl Mark Carl Carl Joe Joe Joe Carl".split(),
        "Quantity": [1, 3, 5, 1, 8, 1, 9, 3],
        "Date": [
            datetime.datetime(2013, 1, 1, 13, 0),
            datetime.datetime(2013, 1, 1, 13, 5),
            datetime.datetime(2013, 10, 1, 20, 0),
            datetime.datetime(2013, 10, 2, 10, 0),
            datetime.datetime(2013, 10, 1, 20, 0),
            datetime.datetime(2013, 10, 2, 10, 0),
            datetime.datetime(2013, 12, 2, 12, 0),
            datetime.datetime(2013, 12, 2, 14, 0),
        ],
    }
)

df
```
Groupby a specific column with the desired frequency. This is like resampling.

```python
df.groupby([pd.Grouper(freq="1ME", key="Date"), "Buyer"])[["Quantity"]].sum()
```
When ``freq`` is specified, the object returned by ``pd.Grouper`` will be an
instance of ``pandas.api.typing.TimeGrouper``. When there is a column and index
with the same name, you can use ``key`` to group by the column and ``level``
to group by the index.

```python
df = df.set_index("Date")
df["Date"] = df.index + pd.offsets.MonthEnd(2)
df.groupby([pd.Grouper(freq="6ME", key="Date"), "Buyer"])[["Quantity"]].sum()

df.groupby([pd.Grouper(freq="6ME", level="Date"), "Buyer"])[["Quantity"]].sum()
```
### Taking the first rows of each group
Just like for a DataFrame or Series you can call head and tail on a groupby:

```python
df = pd.DataFrame([[1, 2], [1, 4], [5, 6]], columns=["A", "B"])
df

g = df.groupby("A")
g.head(1)

g.tail(1)
```
This shows the first or last n rows from each group.



### Taking the nth row of each group
To select the nth item from each group, use `.DataFrameGroupBy.nth` or
`.SeriesGroupBy.nth`. Arguments supplied can be any integer, lists of integers,
slices, or lists of slices; see below for examples. When the nth element of a group
does not exist an error is *not* raised; instead no corresponding rows are returned.

In general this operation acts as a filtration. In certain cases it will also return
one row per group, making it also a reduction. However because in general it can
return zero or multiple rows per group, pandas treats it as a filtration in all cases.

```python
df = pd.DataFrame([[1, np.nan], [1, 4], [5, 6]], columns=["A", "B"])
g = df.groupby("A")

g.nth(0)
g.nth(-1)
g.nth(1)
```
If the nth element of a group does not exist, then no corresponding row is included
in the result. In particular, if the specified ``n`` is larger than any group, the
result will be an empty DataFrame.

```python
g.nth(5)
```
If you want to select the nth not-null item, use the ``dropna`` kwarg. For a DataFrame this should be either ``'any'`` or ``'all'`` just like you would pass to dropna:

```python
# nth(0) is the same as g.first()
g.nth(0, dropna="any")
g.first()

# nth(-1) is the same as g.last()
g.nth(-1, dropna="any")
g.last()

g.B.nth(0, dropna="all")
```
You can also select multiple rows from each group by specifying multiple nth values as a list of ints.

```python
business_dates = pd.date_range(start="4/1/2014", end="6/30/2014", freq="B")
df = pd.DataFrame(1, index=business_dates, columns=["a", "b"])
# get the first, 4th, and last date index for each month
df.groupby([df.index.year, df.index.month]).nth([0, 3, -1])
```
You may also use slices or lists of slices.

```python
df.groupby([df.index.year, df.index.month]).nth[1:]
df.groupby([df.index.year, df.index.month]).nth[1:, :-1]
```
### Enumerate group items
To see the order in which each row appears within its group, use the
``cumcount`` method:

```python
dfg = pd.DataFrame(list("aaabba"), columns=["A"])
dfg

dfg.groupby("A").cumcount()

dfg.groupby("A").cumcount(ascending=False)
```


### Enumerate groups
To see the ordering of the groups (as opposed to the order of rows
within a group given by ``cumcount``) you can use
`.DataFrameGroupBy.ngroup`.



Note that the numbers given to the groups match the order in which the
groups would be seen when iterating over the groupby object, not the
order they are first observed.

```python
dfg = pd.DataFrame(list("aaabba"), columns=["A"])
dfg

dfg.groupby("A").ngroup()

dfg.groupby("A").ngroup(ascending=False)
```
### Plotting
Groupby also works with some plotting methods.  In this case, suppose we
suspect that the values in column 1 are 3 times higher on average in group "B".


```python
np.random.seed(1234)
df = pd.DataFrame(np.random.randn(50, 2))
df["g"] = np.random.choice(["A", "B"], size=50)
df.loc[df["g"] == "B", 1] += 3
```
We can easily visualize this with a boxplot:

```python
:okwarning:

@savefig groupby_boxplot.png
df.groupby("g").boxplot()
```
The result of calling ``boxplot`` is a dictionary whose keys are the values
of our grouping column ``g`` ("A" and "B"). The values of the resulting dictionary
can be controlled by the ``return_type`` keyword of ``boxplot``.
See the `visualization documentation<visualization.box>` for more.

> **warning.capitalize():**
  For historical reasons, ``df.groupby("g").boxplot()`` is not equivalent
  to ``df.boxplot(by="g")``. See `here<visualization.box.return>` for
  an explanation.



### Piping function calls
Similar to the functionality provided by ``DataFrame`` and ``Series``, functions
that take ``GroupBy`` objects can be chained together using a ``pipe`` method to
allow for a cleaner, more readable syntax. To read about ``.pipe`` in general terms,
see `here <basics.pipe>`.

Combining ``.groupby`` and ``.pipe`` is often useful when you need to reuse
GroupBy objects.

As an example, imagine having a DataFrame with columns for stores, products,
revenue and quantity sold. We'd like to do a groupwise calculation of *prices*
(i.e. revenue/quantity) per store and per product. We could do this in a
multi-step operation, but expressing it in terms of piping can make the
code more readable. First we set the data:

```python
n = 1000
df = pd.DataFrame(
    {
        "Store": np.random.choice(["Store_1", "Store_2"], n),
        "Product": np.random.choice(["Product_1", "Product_2"], n),
        "Revenue": (np.random.random(n) * 50 + 10).round(2),
        "Quantity": np.random.randint(1, 10, size=n),
    }
)
df.head(2)
```
We now find the prices per store/product.

```python
(
    df.groupby(["Store", "Product"])
    .pipe(lambda grp: grp.Revenue.sum() / grp.Quantity.sum())
    .unstack()
    .round(2)
)
```
Piping can also be expressive when you want to deliver a grouped object to some
arbitrary function, for example:

```python
def mean(groupby):
    return groupby.mean()


df.groupby(["Store", "Product"]).pipe(mean)
```
Here ``mean`` takes a GroupBy object and finds the mean of the Revenue and Quantity
columns respectively for each Store-Product combination. The ``mean`` function can
be any function that takes in a GroupBy object; the ``.pipe`` will pass the GroupBy
object as a parameter into the function you specify.

## Examples


### Multi-column factorization
By using `.DataFrameGroupBy.ngroup`, we can extract
information about the groups in a way similar to `factorize` (as described
further in the `reshaping API <reshaping.factorize>`) but which applies
naturally to multiple columns of mixed type and different
sources. This can be useful as an intermediate categorical-like step
in processing, when the relationships between the group rows are more
important than their content, or as input to an algorithm which only
accepts the integer encoding. (For more information about support in
pandas for full categorical data, see the Categorical
introduction  and the
`API documentation <api.arrays.categorical>`.)

```python
dfg = pd.DataFrame({"A": [1, 1, 2, 3, 2], "B": list("aaaba")})

dfg

dfg.groupby(["A", "B"]).ngroup()

dfg.groupby(["A", [0, 0, 0, 1, 1]]).ngroup()
```
### GroupBy by indexer to 'resample' data
Resampling produces new hypothetical samples (resamples) from already existing observed data or from a model that generates data. These new samples are similar to the pre-existing samples.

In order for resample to work on indices that are non-datetimelike, the following procedure can be utilized.

In the following examples, **df.index // 5** returns an integer array which is used to determine what gets selected for the groupby operation.

> **note.capitalize():**
   The example below shows how we can downsample by consolidation of samples into fewer ones.
   Here by using **df.index // 5**, we are aggregating the samples in bins. By applying **std()**
   function, we aggregate the information contained in many samples into a small subset of values
   which is their standard deviation thereby reducing the number of samples.

```python
df = pd.DataFrame(np.random.randn(10, 2))
df
df.index // 5
df.groupby(df.index // 5).std()
```
### Returning a Series to propagate names
Group DataFrame columns, compute a set of metrics and return a named Series.
The Series name is used as the name for the column index. This is especially
useful in conjunction with reshaping operations such as stacking, in which the
column index name will be used as the name of the inserted column:

```python
df = pd.DataFrame(
    {
        "a": [0, 0, 0, 0, 1, 1, 1, 1, 2, 2, 2, 2],
        "b": [0, 0, 1, 1, 0, 0, 1, 1, 0, 0, 1, 1],
        "c": [1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1, 0],
        "d": [0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 0, 1],
    }
)

def compute_metrics(x):
    result = {"b_sum": x["b"].sum(), "c_mean": x["c"].mean()}
    return pd.Series(result, name="metrics")

result = df.groupby("a").apply(compute_metrics)

result

result.stack()
```

---

# Windowing operations
pandas contains a compact set of APIs for performing windowing operations - an operation that performs
an aggregation over a sliding partition of values. The API functions similarly to the [`groupby`` API
in that `Series` and `DataFrame` call the windowing method with
necessary parameters and then subsequently call the aggregation function.

```python
s = pd.Series(range(5))
s.rolling(window=2).sum()
```
The windows are comprised by looking back the length of the window from the current observation.
The result above can be derived by taking the sum of the following windowed partitions of data:

```python
for window in s.rolling(window=2):
    print(window)
```


## Overview
pandas supports 4 types of windowing operations:

#. Rolling window: Generic fixed or variable sliding window over the values.
#. Weighted window: Weighted, non-rectangular window supplied by the ``scipy.signal`` library.
#. Expanding window: Accumulating window over the values.
#. Exponentially Weighted window: Accumulating and exponentially weighted window over the values.

=============================   =================  =============================================   ===========================  ========================  ===================================  ===========================
Concept                         Method             Returned Object                                 Supports time-based windows  Supports chained groupby  Supports table method                Supports online operations
=============================   =================  =============================================   ===========================  ========================  ===================================  ===========================
Rolling window                  ``rolling``        ``pandas.typing.api.Rolling``                   Yes                          Yes                       Yes (as of version 1.3)              No
Weighted window                 ``rolling``        ``pandas.typing.api.Window``                    No                           No                        No                                   No
Expanding window                ``expanding``      ``pandas.typing.api.Expanding``                 No                           Yes                       Yes (as of version 1.3)              No
Exponentially Weighted window   ``ewm``            ``pandas.typing.api.ExponentialMovingWindow``   No                           Yes (as of version 1.2)   No                                   Yes (as of version 1.3)
=============================   =================  =============================================   ===========================  ========================  ===================================  ===========================

As noted above, some operations support specifying a window based on a time offset:

```python
s = pd.Series(range(5), index=pd.date_range('2020-01-01', periods=5, freq='1D'))
s.rolling(window='2D').sum()
```
Additionally, some methods support chaining a ``groupby`` operation with a windowing operation
which will first group the data by the specified keys and then perform a windowing operation per group.

```python
df = pd.DataFrame({'A': ['a', 'b', 'a', 'b', 'a'], 'B': range(5)})
df.groupby('A').expanding().sum()
```
> **note.capitalize():**
   Windowing operations currently only support numeric data (integer and float)
   and will always return ``float64`` values.

> **warning.capitalize():**
    Some windowing aggregation, ``mean``, ``sum``, ``var`` and ``std`` methods may suffer from numerical
    imprecision due to the underlying windowing algorithms accumulating sums. When values differ
    with magnitude ``1/np.finfo(np.double).eps`` (approximately `4.5 \times 10^{15}`),
    this results in truncation. It must be
    noted, that large values may have an impact on windows, which do not include these values. `Kahan summation
   ](https://en.wikipedia.org/wiki/Kahan_summation_algorithm)_ is used
    to compute the rolling sums to preserve accuracy as much as possible.




Some windowing operations also support the [`method='table'`` option in the constructor which
performs the windowing operation over an entire `DataFrame` instead of a single column at a time.
This can provide a useful performance benefit for a `DataFrame` with many columns
or the ability to utilize other columns during the windowing
operation. The ``method='table'`` option can only be used if ``engine='numba'`` is specified
in the corresponding method call.

For example, a `weighted mean](https://en.wikipedia.org/wiki/Weighted_arithmetic_mean)_ calculation can
be calculated with [~Rolling.apply` by specifying a separate column of weights.

```python
:okwarning:

def weighted_mean(x):
    arr = np.ones((1, x.shape[1]))
    arr[:, :2] = (x[:, :2] * x[:, 2]).sum(axis=0) / x[:, 2].sum()
    return arr

df = pd.DataFrame([[1, 2, 0.6], [2, 3, 0.4], [3, 4, 0.2], [4, 5, 0.7]])
df.rolling(2, method="table", min_periods=0).apply(weighted_mean, raw=True, engine="numba")  # noqa: E501
```


Some windowing operations also support an ``online`` method after constructing a windowing object
which returns a new object that supports passing in new `DataFrame` or `Series` objects
to continue the windowing calculation with the new values (i.e. online calculations).

The methods on this new windowing objects must call the aggregation method first to "prime" the initial
state of the online calculation. Then, new `DataFrame` or `Series` objects can be passed in
the ``update`` argument to continue the windowing calculation.

```python
df = pd.DataFrame([[1, 2, 0.6], [2, 3, 0.4], [3, 4, 0.2], [4, 5, 0.7]])
df.ewm(0.5).mean()
```
```python
:okwarning:

online_ewm = df.head(2).ewm(0.5).online()
online_ewm.mean()
online_ewm.mean(update=df.tail(1))
```
All windowing operations support a ``min_periods`` argument that dictates the minimum amount of
non-``np.nan`` values a window must have; otherwise, the resulting value is ``np.nan``.
``min_periods`` defaults to 1 for time-based windows and ``window`` for fixed windows

```python
s = pd.Series([np.nan, 1, 2, np.nan, np.nan, 3])
s.rolling(window=3, min_periods=1).sum()
s.rolling(window=3, min_periods=2).sum()
# Equivalent to min_periods=3
s.rolling(window=3, min_periods=None).sum()
```
Additionally, all windowing operations supports the ``aggregate`` method for returning a result
of multiple aggregations applied to a window.

```python
df = pd.DataFrame({"A": range(5), "B": range(10, 15)})
df.expanding().agg(["sum", "mean", "std"])
```


## Rolling window
Generic rolling windows support specifying windows as a fixed number of observations or variable
number of observations based on an offset. If a time based offset is provided, the corresponding
time based index must be monotonic.

```python
times = ['2020-01-01', '2020-01-03', '2020-01-04', '2020-01-05', '2020-01-29']
s = pd.Series(range(5), index=pd.DatetimeIndex(times))
s
# Window with 2 observations
s.rolling(window=2).sum()
# Window with 2 days worth of observations
s.rolling(window='2D').sum()
```
For all supported aggregation functions, see `api.functions_rolling`.



### Centering windows
By default the labels are set to the right edge of the window, but a
``center`` keyword is available so the labels can be set at the center.

```python
s = pd.Series(range(10))
s.rolling(window=5).mean()
s.rolling(window=5, center=True).mean()
```
This can also be applied to datetime-like indices.



```python
df = pd.DataFrame(
    {"A": [0, 1, 2, 3, 4]}, index=pd.date_range("2020", periods=5, freq="1D")
)
df
df.rolling("2D", center=False).mean()
df.rolling("2D", center=True).mean()
```


### Rolling window endpoints
The inclusion of the interval endpoints in rolling window calculations can be specified with the ``closed``
parameter:

=============  ====================
Value          Behavior
=============  ====================
``'right'``     close right endpoint
``'left'``     close left endpoint
``'both'``     close both endpoints
``'neither'``  open endpoints
=============  ====================

For example, having the right endpoint open is useful in many problems that require that there is no contamination
from present information back to past information. This allows the rolling window to compute statistics
"up to that point in time", but not including that point in time.

```python
df = pd.DataFrame(
    {"x": 1},
    index=[
        pd.Timestamp("20130101 09:00:01"),
        pd.Timestamp("20130101 09:00:02"),
        pd.Timestamp("20130101 09:00:03"),
        pd.Timestamp("20130101 09:00:04"),
        pd.Timestamp("20130101 09:00:06"),
    ],
)

df["right"] = df.rolling("2s", closed="right").x.sum()  # default
df["both"] = df.rolling("2s", closed="both").x.sum()
df["left"] = df.rolling("2s", closed="left").x.sum()
df["neither"] = df.rolling("2s", closed="neither").x.sum()

df
```


### Custom window rolling
In addition to accepting an integer or offset as a ``window`` argument, ``rolling`` also accepts
a ``BaseIndexer`` subclass that allows a user to define a custom method for calculating window bounds.
The ``BaseIndexer`` subclass will need to define a ``get_window_bounds`` method that returns
a tuple of two arrays, the first being the starting indices of the windows and second being the
ending indices of the windows. Additionally, ``num_values``, ``min_periods``, ``center``, ``closed``
and ``step`` will automatically be passed to ``get_window_bounds`` and the defined method must
always accept these arguments.

For example, if we have the following `DataFrame`

```python
use_expanding = [True, False, True, False, True]
use_expanding
df = pd.DataFrame({"values": range(5)})
df
```
and we want to use an expanding window where ``use_expanding`` is ``True`` otherwise a window of size
1, we can create the following ``BaseIndexer`` subclass:

```python
from pandas.api.indexers import BaseIndexer

class CustomIndexer(BaseIndexer):
     def get_window_bounds(self, num_values, min_periods, center, closed, step):
         start = np.empty(num_values, dtype=np.int64)
         end = np.empty(num_values, dtype=np.int64)
         for i in range(num_values):
             if self.use_expanding[i]:
                 start[i] = 0
                 end[i] = i + 1
             else:
                 start[i] = i
                 end[i] = i + self.window_size
         return start, end

indexer = CustomIndexer(window_size=1, use_expanding=use_expanding)

df.rolling(indexer).sum()
```
You can view other examples of ``BaseIndexer`` subclasses `here](https://github.com/pandas-dev/pandas/blob/main/pandas/core/indexers/objects.py)_

One subclass of note within those examples is the ``VariableOffsetWindowIndexer`` that allows
rolling operations over a non-fixed offset like a ``BusinessDay``.

```python
from pandas.api.indexers import VariableOffsetWindowIndexer

df = pd.DataFrame(range(10), index=pd.date_range("2020", periods=10))
offset = pd.offsets.BDay(1)
indexer = VariableOffsetWindowIndexer(index=df.index, offset=offset)
df
df.rolling(indexer).sum()
```
For some problems knowledge of the future is available for analysis. For example, this occurs when
each data point is a full time series read from an experiment, and the task is to extract underlying
conditions. In these cases it can be useful to perform forward-looking rolling window computations.
`FixedForwardWindowIndexer <pandas.api.indexers.FixedForwardWindowIndexer>` class is available for this purpose.
This `BaseIndexer <pandas.api.indexers.BaseIndexer>[ subclass implements a closed fixed-width
forward-looking rolling window, and we can use it as follows:

```python
from pandas.api.indexers import FixedForwardWindowIndexer
indexer = FixedForwardWindowIndexer(window_size=2)
df.rolling(indexer, min_periods=1).sum()
```
We can also achieve this by using slicing, applying rolling aggregation, and then flipping the result as shown in example below:

```python
df = pd.DataFrame(
    data=[
        [pd.Timestamp("2018-01-01 00:00:00"), 100],
        [pd.Timestamp("2018-01-01 00:00:01"), 101],
        [pd.Timestamp("2018-01-01 00:00:03"), 103],
        [pd.Timestamp("2018-01-01 00:00:04"), 111],
    ],
    columns=["time", "value"],
).set_index("time")
df

reversed_df = df[::-1].rolling("2s").sum()[::-1]
reversed_df
```


### Rolling apply
The `~Rolling.apply` function takes an extra ``func`` argument and performs
generic rolling computations. The ``func`` argument should be a single function
that produces a single value from an ndarray input. ``raw`` specifies whether
the windows are cast as `Series` objects (``raw=False``) or ndarray objects (``raw=True``).

```python
def mad(x):
    return np.fabs(x - x.mean()).mean()

s = pd.Series(range(10))
s.rolling(window=4).apply(mad, raw=True)
```


### Numba engine
Additionally, `~Rolling.apply` can leverage `Numba](https://numba.pydata.org/)_
if installed as an optional dependency. The apply aggregation can be executed using Numba by specifying
``engine='numba'`` and ``engine_kwargs`` arguments (``raw`` must also be set to ``True``).
See `enhancing performance with Numba <enhancingperf.numba>[ for general usage of the arguments and performance considerations.

Numba will be applied in potentially two routines:

#. If ``func`` is a standard Python function, the engine will `JIT](https://numba.readthedocs.io/en/stable/user/overview.html)_ the passed function. [`func`` can also be a JITed function in which case the engine will not JIT the function again.
#. The engine will JIT the for loop where the apply function is applied to each window.

The ``engine_kwargs`` argument is a dictionary of keyword arguments that will be passed into the
`numba.jit decorator](https://numba.readthedocs.io/en/stable/user/jit.html)_.
These keyword arguments will be applied to *both* the passed function (if a standard Python function)
and the apply for loop over each window.



[`mean``, ``median``, ``max``, ``min``, and ``sum`` also support the ``engine`` and ``engine_kwargs`` arguments.



### Binary window functions
`~Rolling.cov` and `~Rolling.corr` can compute moving window statistics about
two `Series` or any combination of `DataFrame`/`Series` or
`DataFrame`/`DataFrame`. Here is the behavior in each case:

* two `Series`: compute the statistic for the pairing.
* `DataFrame`/`Series`: compute the statistics for each column of the DataFrame
  with the passed Series, thus returning a DataFrame.
* `DataFrame`/`DataFrame`: by default compute the statistic for matching column
  names, returning a DataFrame. If the keyword argument ``pairwise=True`` is
  passed then computes the statistic for each pair of columns, returning a `DataFrame` with a
  `MultiIndex` whose values are the dates in question (see the next section
  ).

For example:

```python
df = pd.DataFrame(
    np.random.randn(10, 4),
    index=pd.date_range("2020-01-01", periods=10),
    columns=["A", "B", "C", "D"],
)
df = df.cumsum()

df2 = df[:4]
df2.rolling(window=2).corr(df2["B"])
```


### Computing rolling pairwise covariances and correlations
In financial data analysis and other fields it's common to compute covariance
and correlation matrices for a collection of time series. Often one is also
interested in moving-window covariance and correlation matrices. This can be
done by passing the ``pairwise`` keyword argument, which in the case of
`DataFrame` inputs will yield a MultiIndexed `DataFrame` whose ``index`` are the dates in
question. In the case of a single DataFrame argument the ``pairwise`` argument
can even be omitted:

> **note.capitalize():**
    Missing values are ignored and each entry is computed using the pairwise
    complete observations.

    Assuming the missing data are missing at random this results in an estimate
    for the covariance matrix which is unbiased. However, for many applications
    this estimate may not be acceptable because the estimated covariance matrix
    is not guaranteed to be positive semi-definite. This could lead to
    estimated correlations having absolute values which are greater than one,
    and/or a non-invertible covariance matrix. See `Estimation of covariance
    matrices](https://en.wikipedia.org/w/index.php?title=Estimation_of_covariance_matrices)
    for more details.

```python
covs = (
    df[["B", "C", "D"]]
    .rolling(window=4)
    .cov(df[["A", "B", "C"]], pairwise=True)
)
covs
```


## Weighted window
The ``win_type`` argument in ``.rolling`` generates a weighted windows that are commonly used in filtering
and spectral estimation. ``win_type`` must be string that corresponds to a `scipy.signal window function
<https://docs.scipy.org/doc/scipy/reference/signal.windows.html#module-scipy.signal.windows>`__.
Scipy must be installed in order to use these windows, and supplementary arguments
that the Scipy window methods take must be specified in the aggregation function.


```python
s = pd.Series(range(10))
s.rolling(window=5).mean()
s.rolling(window=5, win_type="triang").mean()
# Supplementary Scipy arguments passed in the aggregation function
s.rolling(window=5, win_type="gaussian").mean(std=0.1)
```
For all supported aggregation functions, see `api.functions_window`.



## Expanding window
An expanding window yields the value of an aggregation statistic with all the data available up to that
point in time. Since these calculations are a special case of rolling statistics,
they are implemented in pandas such that the following two calls are equivalent:

```python
df = pd.DataFrame(range(5))
df.rolling(window=len(df), min_periods=1).mean()
df.expanding(min_periods=1).mean()
```
For all supported aggregation functions, see `api.functions_expanding`.




## Exponentially weighted window
An exponentially weighted window is similar to an expanding window but with each prior point
being exponentially weighted down relative to the current point.

In general, a weighted moving average is calculated as



    y_t = \frac{\sum_{i=0}^t w_i x_{t-i}}{\sum_{i=0}^t w_i},

where `x_t` is the input, `y_t` is the result and the `w_i`
are the weights.

For all supported aggregation functions, see `api.functions_ewm`.

The EW functions support two variants of exponential weights.
The default, ``adjust=True``, uses the weights `w_i = (1 - \alpha)^i`
which gives



    y_t = \frac{x_t + (1 - \alpha)x_{t-1} + (1 - \alpha)^2 x_{t-2} + ...
    + (1 - \alpha)^t x_{0}}{1 + (1 - \alpha) + (1 - \alpha)^2 + ...
    + (1 - \alpha)^t}

When ``adjust=False`` is specified, moving averages are calculated as



    y_0 &= x_0 \\
    y_t &= (1 - \alpha) y_{t-1} + \alpha x_t,

which is equivalent to using weights



    w_i = \begin{cases}
        \alpha (1 - \alpha)^i & \text{if } i < t \\
        (1 - \alpha)^i        & \text{if } i = t.
    \end{cases}

> **note.capitalize():**
   These equations are sometimes written in terms of `\alpha' = 1 - \alpha`, e.g.

   .. math::

      y_t = \alpha' y_{t-1} + (1 - \alpha') x_t.

The difference between the above two variants arises because we are
dealing with series which have finite history. Consider a series of infinite
history, with ``adjust=True``:



    y_t = \frac{x_t + (1 - \alpha)x_{t-1} + (1 - \alpha)^2 x_{t-2} + ...}
    {1 + (1 - \alpha) + (1 - \alpha)^2 + ...}

Noting that the denominator is a geometric series with initial term equal to 1
and a ratio of `1 - \alpha` we have



    y_t &= \frac{x_t + (1 - \alpha)x_{t-1} + (1 - \alpha)^2 x_{t-2} + ...}
    {\frac{1}{1 - (1 - \alpha)}}\\
    &= [x_t + (1 - \alpha)x_{t-1} + (1 - \alpha)^2 x_{t-2} + ...] \alpha \\
    &= \alpha x_t + [(1-\alpha)x_{t-1} + (1 - \alpha)^2 x_{t-2} + ...]\alpha \\
    &= \alpha x_t + (1 - \alpha)[x_{t-1} + (1 - \alpha) x_{t-2} + ...]\alpha\\
    &= \alpha x_t + (1 - \alpha) y_{t-1}

which is the same expression as ``adjust=False`` above and therefore
shows the equivalence of the two variants for infinite series.
When ``adjust=False``, we have `y_0 = x_0` and
`y_t = \alpha x_t + (1 - \alpha) y_{t-1}`.
Therefore, there is an assumption that `x_0` is not an ordinary value
but rather an exponentially weighted moment of the infinite series up to that
point.

One must have `0 < \alpha \leq 1[, and while it is possible to pass
`\alpha` directly, it's often easier to think about either the
**span**, **center  of mass (com)** or **half-life** of an EW moment:



   \alpha =
    \begin{cases}
        \frac{2}{s + 1},            & \text{for span}\ s \geq 1\\
        \frac{1}{1 + c},            & \text{for center of mass}\ c \geq 0\\
        1 - e^{\frac{\log 0.5}{h}}, & \text{for half-life}\ h > 0
    \end{cases}

One must specify precisely one of **span**, **center of mass**, **half-life**
and **alpha** to the EW functions:

* **Span** corresponds to what is commonly called an "N-day EW moving average".
* **Center of mass** has a more physical interpretation and can be thought of
  in terms of span: `c = (s - 1) / 2`.
* **Half-life** is the period of time for the exponential weight to reduce to
  one half.
* **Alpha** specifies the smoothing factor directly.

You can also specify ``halflife`` in terms of a timedelta convertible unit to specify the amount of
time it takes for an observation to decay to half its value when also specifying a sequence
of ``times``.

```python
df = pd.DataFrame({"B": [0, 1, 2, np.nan, 4]})
df
times = ["2020-01-01", "2020-01-03", "2020-01-10", "2020-01-15", "2020-01-17"]
df.ewm(halflife="4 days", times=pd.DatetimeIndex(times)).mean()
```
The following formula is used to compute exponentially weighted mean with an input vector of times:



    y_t = \frac{\sum_{i=0}^t 0.5^\frac{t_{t} - t_{i}}{\lambda} x_{t-i}}{\sum_{i=0}^t 0.5^\frac{t_{t} - t_{i}}{\lambda}},


ExponentialMovingWindow also has an ``ignore_na`` argument, which determines how
intermediate null values affect the calculation of the weights.
When ``ignore_na=False`` (the default), weights are calculated based on absolute
positions, so that intermediate null values affect the result.
When ``ignore_na=True``,
weights are calculated by ignoring intermediate null values.
For example, assuming ``adjust=True``, if ``ignore_na=False``, the weighted
average of ``3, NaN, 5`` would be calculated as



        \frac{(1-\alpha)^2 \cdot 3 + 1 \cdot 5}{(1-\alpha)^2 + 1}.

Whereas if ``ignore_na=True``, the weighted average would be calculated as



        \frac{(1-\alpha) \cdot 3 + 1 \cdot 5}{(1-\alpha) + 1}.

The `~Ewm.var`, `~Ewm.std`, and `~Ewm.cov` functions have a ``bias`` argument,
specifying whether the result should contain biased or unbiased statistics.
For example, if ``bias=True``, ``ewmvar(x)`` is calculated as
``ewmvar(x) = ewma(x**2) - ewma(x)**2``;
whereas if ``bias=False`` (the default), the biased variance statistics
are scaled by debiasing factors



    \frac{\left(\sum_{i=0}^t w_i\right)^2}{\left(\sum_{i=0}^t w_i\right)^2 - \sum_{i=0}^t w_i^2}.

(For `w_i = 1`, this reduces to the usual `N / (N - 1)` factor,
with `N = t + 1`.)
See `Weighted Sample Variance](https://en.wikipedia.org/wiki/Weighted_arithmetic_mean#Weighted_sample_variance)_
on Wikipedia for further details.

---

# Time series / date functionality
pandas contains extensive capabilities and features for working with time series data for all domains.
Using the NumPy ``datetime64`` and ``timedelta64`` dtypes, pandas has consolidated a large number of
features from other Python libraries like ``scikits.timeseries`` as well as created
a tremendous amount of new functionality for manipulating time series data.

For example, pandas supports:

Parsing time series information from various sources and formats

```python
import datetime

dti = pd.to_datetime(
    ["1/1/2018", np.datetime64("2018-01-01"), datetime.datetime(2018, 1, 1)]
)
dti
```
Generate sequences of fixed-frequency dates and time spans

```python
dti = pd.date_range("2018-01-01", periods=3, freq="h")
dti
```
Manipulating and converting date times with timezone information

```python
dti = dti.tz_localize("UTC")
dti
dti.tz_convert("US/Pacific")
```
Resampling or converting a time series to a particular frequency

```python
idx = pd.date_range("2018-01-01", periods=5, freq="h")
ts = pd.Series(range(len(idx)), index=idx)
ts
ts.resample("2h").mean()
```
Performing date and time arithmetic with absolute or relative time increments

```python
friday = pd.Timestamp("2018-01-05")
friday.day_name()
# Add 1 day
saturday = friday + pd.Timedelta("1 day")
saturday.day_name()
# Add 1 business day (Friday --> Monday)
monday = friday + pd.offsets.BDay()
monday.day_name()
```
pandas provides a relatively compact and self-contained set of tools for
performing the above tasks and more.




## Overview
pandas captures 4 general time related concepts:

#. Date times: A specific date and time with timezone support. Similar to ``datetime.datetime`` from the standard library.
#. Time deltas: An absolute time duration. Similar to ``datetime.timedelta`` from the standard library.
#. Time spans: A span of time defined by a point in time and its associated frequency.
#. Date offsets: A relative time duration that respects calendar arithmetic. Similar to ``dateutil.relativedelta.relativedelta`` from the ``dateutil`` package.

=====================   =================  ===================   ============================================  ========================================
Concept                 Scalar Class       Array Class           pandas Data Type                              Primary Creation Method
=====================   =================  ===================   ============================================  ========================================
Date times              ``Timestamp``      ``DatetimeIndex``     ``datetime64[ns]`` or ``datetime64[ns, tz]``  ``to_datetime`` or ``date_range``
Time deltas             ``Timedelta``      ``TimedeltaIndex``    ``timedelta64[ns]``                           ``to_timedelta`` or ``timedelta_range``
Time spans              ``Period``         ``PeriodIndex``       ``period[freq]``                              ``Period`` or ``period_range``
Date offsets            ``DateOffset``     ``None``              ``None``                                      ``DateOffset``
=====================   =================  ===================   ============================================  ========================================

For time series data, it's conventional to represent the time component in the index of a `Series` or `DataFrame`
so manipulations can be performed with respect to the time element.

```python
pd.Series(range(3), index=pd.date_range("2000", freq="D", periods=3))
```
However, `Series` and `DataFrame` can directly also support the time component as data itself.

```python
pd.Series(pd.date_range("2000", freq="D", periods=3))
```
`Series` and `DataFrame` have extended data type support and functionality for ``datetime``, ``timedelta``
and ``Period`` data when passed into those constructors. ``DateOffset``
data however will be stored as ``object`` data.

```python
pd.Series(pd.period_range("1/1/2011", freq="M", periods=3))
pd.Series([pd.DateOffset(1), pd.DateOffset(2)])
pd.Series(pd.date_range("1/1/2011", freq="ME", periods=3))
```
Lastly, pandas represents null date times, time deltas, and time spans as ``NaT`` which
is useful for representing missing or null date like values and behaves similar
as ``np.nan`` does for float data.

```python
pd.Timestamp(pd.NaT)
pd.Timedelta(pd.NaT)
pd.Period(pd.NaT)
# Equality acts as np.nan would
pd.NaT == pd.NaT
```


## Timestamps vs. time spans
Timestamped data is the most basic type of time series data that associates
values with points in time. For pandas objects it means using the points in
time.

```python
import datetime

pd.Timestamp(datetime.datetime(2012, 5, 1))
pd.Timestamp("2012-05-01")
pd.Timestamp(2012, 5, 1)
```
However, in many cases it is more natural to associate things like change
variables with a time span instead. The span represented by ``Period`` can be
specified explicitly, or inferred from datetime string format.

For example:

```python
pd.Period("2011-01")

pd.Period("2012-05", freq="D")
```
`Timestamp` and `Period` can serve as an index. Lists of
``Timestamp`` and ``Period`` are automatically coerced to `DatetimeIndex`
and `PeriodIndex` respectively.

```python
dates = [
    pd.Timestamp("2012-05-01"),
    pd.Timestamp("2012-05-02"),
    pd.Timestamp("2012-05-03"),
]
ts = pd.Series(np.random.randn(3), dates)

type(ts.index)
ts.index

ts

periods = [pd.Period("2012-01"), pd.Period("2012-02"), pd.Period("2012-03")]

ts = pd.Series(np.random.randn(3), periods)

type(ts.index)
ts.index

ts
```
pandas allows you to capture both representations and
convert between them. Under the hood, pandas represents timestamps using
instances of ``Timestamp`` and sequences of timestamps using instances of
``DatetimeIndex``. For regular time spans, pandas uses ``Period`` objects for
scalar values and ``PeriodIndex`` for sequences of spans. Better support for
irregular intervals with arbitrary start and end points are forth-coming in
future releases.




## Converting to timestamps
To convert a `Series` or list-like object of date-like objects e.g. strings,
epochs, or a mixture, you can use the ``to_datetime`` function. When passed
a ``Series``, this returns a ``Series`` (with the same index), while a list-like
is converted to a ``DatetimeIndex``:

```python
pd.to_datetime(pd.Series(["Jul 31, 2009", "Jan 10, 2010", None]))

pd.to_datetime(["2005/11/23", "2010/12/31"])
```
If you use dates which start with the day first (i.e. European style),
you can pass the ``dayfirst`` flag:

```python
:okwarning:

pd.to_datetime(["04-01-2012 10:00"], dayfirst=True)

pd.to_datetime(["04-14-2012 10:00"], dayfirst=True)
```
> **warning.capitalize():**
   You see in the above example that ``dayfirst`` isn't strict. If a date
   can't be parsed with the day being first it will be parsed as if
   ``dayfirst`` were ``False`` and a warning will also be raised.

If you pass a single string to ``to_datetime``, it returns a single ``Timestamp``.
``Timestamp`` can also accept string input, but it doesn't accept string parsing
options like ``dayfirst`` or ``format``, so use ``to_datetime`` if these are required.

```python
pd.to_datetime("2010/11/12")

pd.Timestamp("2010/11/12")
```
You can also use the ``DatetimeIndex`` constructor directly:

```python
pd.DatetimeIndex(["2018-01-01", "2018-01-03", "2018-01-05"])
```
The string 'infer' can be passed in order to set the frequency of the index as the
inferred frequency upon creation:

```python
pd.DatetimeIndex(["2018-01-01", "2018-01-03", "2018-01-05"], freq="infer")
```


### Providing a format argument
In addition to the required datetime string, a ``format`` argument can be passed to ensure specific parsing.
This could also potentially speed up the conversion considerably.

```python
pd.to_datetime("2010/11/12", format="%Y/%m/%d")

pd.to_datetime("12-11-2010 00:00", format="%d-%m-%Y %H:%M")
```
For more information on the choices available when specifying the ``format``
option, see the Python `datetime documentation`_.

.. _datetime documentation: https://docs.python.org/3/library/datetime.html#strftime-and-strptime-behavior

### Assembling datetime from multiple DataFrame columns
You can also pass a ``DataFrame`` of integer or string columns to assemble into a ``Series`` of ``Timestamps``.

```python
df = pd.DataFrame(
    {"year": [2015, 2016], "month": [2, 3], "day": [4, 5], "hour": [2, 3]}
)
pd.to_datetime(df)
```
You can pass only the columns that you need to assemble.

```python
pd.to_datetime(df[["year", "month", "day"]])
```
``pd.to_datetime`` looks for standard designations of the datetime component in the column names, including:

* required: ``year``, ``month``, ``day``
* optional: ``hour``, ``minute``, ``second``, ``millisecond``, ``microsecond``, ``nanosecond``

### Invalid data
The default behavior, ``errors='raise'``, is to raise when unparsable:

```python
:okexcept:

pd.to_datetime(['2009/07/31', 'asd'], errors='raise')
```
Pass ``errors='coerce'`` to convert unparsable data to ``NaT`` (not a time):

```python
pd.to_datetime(["2009/07/31", "asd"], errors="coerce")
```


### Epoch timestamps
pandas supports converting integer or float epoch times to ``Timestamp`` and
``DatetimeIndex``. The default unit is nanoseconds, since that is how ``Timestamp``
objects are stored internally. However, epochs are often stored in another ``unit``
which can be specified. These are computed from the starting point specified by the
``origin`` parameter.

```python
pd.to_datetime(
    [1349720105, 1349806505, 1349892905, 1349979305, 1350065705], unit="s"
)

pd.to_datetime(
    [1349720105100, 1349720105200, 1349720105300, 1349720105400, 1349720105500],
    unit="ms",
)
```
> **note.capitalize():**
   The ``unit`` parameter does not use the same strings as the ``format`` parameter
   that was discussed `above<timeseries.converting.format>`. The
   available units are listed on the documentation for `pandas.to_datetime`.

Constructing a `Timestamp` or `DatetimeIndex` with an epoch timestamp
with the ``tz`` argument specified will raise a ValueError. If you have
epochs in wall time in another timezone, you can read the epochs
as timezone-naive timestamps and then localize to the appropriate timezone:

```python
pd.Timestamp(1262347200000000000).tz_localize("US/Pacific")
pd.DatetimeIndex([1262347200000000000]).tz_localize("US/Pacific")
```
> **note.capitalize():**
   Epoch times will be rounded to the nearest nanosecond.

> **warning.capitalize():**
   Conversion of float epoch times can lead to inaccurate and unexpected results.
   `Python floats <python:tut-fp-issues>` have about 15 digits precision in
   decimal. Rounding during conversion from float to high precision ``Timestamp`` is
   unavoidable. The only way to achieve exact precision is to use a fixed-width
   types (e.g. an int64).

   ```python
pd.to_datetime([1490195805.433, 1490195805.433502912], unit="s")
pd.to_datetime(1490195805433502912, unit="ns")
```
> **seealso.capitalize():**
   `timeseries.origin`



### From timestamps to epoch
To invert the operation from above, namely, to convert from a ``Timestamp`` to a 'unix' epoch:

```python
stamps = pd.date_range("2012-10-08 18:15:05", periods=4, freq="D")
stamps
```
We subtract the epoch (midnight at January 1, 1970 UTC) and then floor divide by the
"unit" (1 second).

```python
(stamps - pd.Timestamp("1970-01-01")) // pd.Timedelta("1s")
```


### Using the ``origin`` parameter
Using the ``origin`` parameter, one can specify an alternative starting point for creation
of a ``DatetimeIndex``. For example, to use 1960-01-01 as the starting date:

```python
pd.to_datetime([1, 2, 3], unit="D", origin=pd.Timestamp("1960-01-01"))
```
The default is set at ``origin='unix'``, which defaults to ``1970-01-01 00:00:00``.
Commonly called 'unix epoch' or POSIX time.

```python
pd.to_datetime([1, 2, 3], unit="D")
```


## Generating ranges of timestamps
To generate an index with timestamps, you can use either the ``DatetimeIndex`` or
``Index`` constructor and pass in a list of datetime objects:

```python
dates = [
    datetime.datetime(2012, 5, 1),
    datetime.datetime(2012, 5, 2),
    datetime.datetime(2012, 5, 3),
]

# Note the frequency information
index = pd.DatetimeIndex(dates)
index

# Automatically converted to DatetimeIndex
index = pd.Index(dates)
index
```
In practice this becomes very cumbersome because we often need a very long
index with a large number of timestamps. If we need timestamps on a regular
frequency, we can use the `date_range` and `bdate_range` functions
to create a ``DatetimeIndex``. The default frequency for ``date_range`` is a
**calendar day** while the default for ``bdate_range`` is a **business day**:

```python
start = datetime.datetime(2011, 1, 1)
end = datetime.datetime(2012, 1, 1)

index = pd.date_range(start, end)
index

index = pd.bdate_range(start, end)
index
```
Convenience functions like ``date_range`` and ``bdate_range`` can utilize a
variety of `frequency aliases <timeseries.offset_aliases>`:

```python
pd.date_range(start, periods=1000, freq="ME")

pd.bdate_range(start, periods=250, freq="BQS")
```
``date_range`` and ``bdate_range`` make it easy to generate a range of dates
using various combinations of parameters like ``start``, ``end``, ``periods``,
and ``freq``. The start and end dates are strictly inclusive, so dates outside
of those specified will not be generated:

```python
pd.date_range(start, end, freq="BME")

pd.date_range(start, end, freq="W")

pd.bdate_range(end=end, periods=20)

pd.bdate_range(start=start, periods=20)
```
Specifying ``start``, ``end``, and ``periods`` will generate a range of evenly spaced
dates from ``start`` to ``end`` inclusively, with ``periods`` number of elements in the
resulting ``DatetimeIndex``:

```python
pd.date_range("2018-01-01", "2018-01-05", periods=5)

pd.date_range("2018-01-01", "2018-01-05", periods=10)
```


### Custom frequency ranges
``bdate_range`` can also generate a range of custom frequency dates by using
the ``weekmask`` and ``holidays`` parameters.  These parameters will only be
used if a custom frequency string is passed.

```python
weekmask = "Mon Wed Fri"

holidays = [datetime.datetime(2011, 1, 5), datetime.datetime(2011, 3, 14)]

pd.bdate_range(start, end, freq="C", weekmask=weekmask, holidays=holidays)

pd.bdate_range(start, end, freq="CBMS", weekmask=weekmask)
```
> **seealso.capitalize():**
   `timeseries.custombusinessdays`



## Timestamp limitations
The limits of timestamp representation depend on the chosen resolution. For
nanosecond resolution, the time span that
can be represented using a 64-bit integer is limited to approximately 584 years:

```python
pd.Timestamp.min
pd.Timestamp.max
```
When choosing second-resolution, the available range grows to  ``+/- 2.9e11 years``.
Different resolutions can be converted to each other through ``as_unit``.

> **seealso.capitalize():**
   `timeseries.oob`



## Indexing
One of the main uses for ``DatetimeIndex`` is as an index for pandas objects.
The ``DatetimeIndex`` class contains many time series related optimizations:

* A large range of dates for various offsets are pre-computed and cached
  under the hood in order to make generating subsequent date ranges very fast
  (just have to grab a slice).
* Fast shifting using the ``shift`` method on pandas objects.
* Unioning of overlapping ``DatetimeIndex`` objects with the same frequency is
  very fast (important for fast data alignment).
* Quick access to date fields via properties such as ``year``, ``month``, etc.
* Regularization functions like ``snap`` and very fast ``asof`` logic.

``DatetimeIndex`` objects have all the basic functionality of regular ``Index``
objects, and a smorgasbord of advanced time series specific methods for easy
frequency processing.

> **seealso.capitalize():**
    `Reindexing methods <basics.reindexing>`

> **note.capitalize():**
    While pandas does not force you to have a sorted date index, some of these
    methods may have unexpected or incorrect behavior if the dates are unsorted.

``DatetimeIndex`` can be used like a regular index and offers all of its
intelligent functionality like selection, slicing, etc.

```python
rng = pd.date_range(start, end, freq="BME")
ts = pd.Series(np.random.randn(len(rng)), index=rng)
ts.index
ts[:5].index
ts[::2].index
```


### Partial string indexing
Dates and strings that parse to timestamps can be passed as indexing parameters:

```python
ts["1/31/2011"]

ts[datetime.datetime(2011, 12, 25):]

ts["10/31/2011":"12/31/2011"]
```
To provide convenience for accessing longer time series, you can also pass in
the year or year and month as strings:

```python
ts["2011"]

ts["2011-6"]
```
This type of slicing will work on a ``DataFrame`` with a ``DatetimeIndex`` as well. Since the
partial string selection is a form of label slicing, the endpoints **will be** included. This
would include matching times on an included date:

> **warning.capitalize():**
   Indexing ``DataFrame`` rows with a *single* string with getitem (e.g. ``frame[dtstring]``)
   is deprecated starting with pandas 1.2.0 (given the ambiguity whether it is indexing
   the rows or selecting a column) and will be removed in a future version. The equivalent
   with ``.loc`` (e.g. ``frame.loc[dtstring]``) is still supported.

```python
dft = pd.DataFrame(
    np.random.randn(100000, 1),
    columns=["A"],
    index=pd.date_range("20130101", periods=100000, freq="min"),
)
dft
dft.loc["2013"]
```
This starts on the very first time in the month, and includes the last date and
time for the month:

```python
dft["2013-1":"2013-2"]
```
This specifies a stop time **that includes all of the times on the last day**:

```python
dft["2013-1":"2013-2-28"]
```
This specifies an **exact** stop time (and is not the same as the above):

```python
dft["2013-1":"2013-2-28 00:00:00"]
```
We are stopping on the included end-point as it is part of the index:

```python
dft["2013-1-15":"2013-1-15 12:30:00"]
```
``DatetimeIndex`` partial string indexing also works on a ``DataFrame`` with a ``MultiIndex``:

```python
dft2 = pd.DataFrame(
    np.random.randn(20, 1),
    columns=["A"],
    index=pd.MultiIndex.from_product(
        [pd.date_range("20130101", periods=10, freq="12h"), ["a", "b"]]
    ),
)
dft2
dft2.loc["2013-01-05"]
idx = pd.IndexSlice
dft2 = dft2.swaplevel(0, 1).sort_index()
dft2.loc[idx[:, "2013-01-05"], :]
```
Slicing with string indexing also honors UTC offset.

```python
df = pd.DataFrame([0], index=pd.DatetimeIndex(["2019-01-01"], tz="US/Pacific"))
df
df["2019-01-01 12:00:00+04:00":"2019-01-01 13:00:00+04:00"]
```


### Slice vs. exact match
The same string used as an indexing parameter can be treated either as a slice or as an exact match depending on the resolution of the index. If the string is less accurate than the index, it will be treated as a slice, otherwise as an exact match.

Consider a ``Series`` object with a minute resolution index:

```python
series_minute = pd.Series(
    [1, 2, 3],
    pd.DatetimeIndex(
        ["2011-12-31 23:59:00", "2012-01-01 00:00:00", "2012-01-01 00:02:00"]
    ),
)
series_minute.index.resolution
```
A timestamp string less accurate than a minute gives a ``Series`` object.

```python
series_minute["2011-12-31 23"]
```
A timestamp string with minute resolution (or more accurate), gives a scalar instead, i.e. it is not casted to a slice.

```python
series_minute["2011-12-31 23:59"]
series_minute["2011-12-31 23:59:00"]
```
If index resolution is second, then the minute-accurate timestamp gives a
``Series``.

```python
series_second = pd.Series(
    [1, 2, 3],
    pd.DatetimeIndex(
        ["2011-12-31 23:59:59", "2012-01-01 00:00:00", "2012-01-01 00:00:01"]
    ),
)
series_second.index.resolution
series_second["2011-12-31 23:59"]
```
If the timestamp string is treated as a slice, it can be used to index ``DataFrame`` with ``.loc[]`` as well.

```python
dft_minute = pd.DataFrame(
    {"a": [1, 2, 3], "b": [4, 5, 6]}, index=series_minute.index
)
dft_minute.loc["2011-12-31 23"]
```
> **warning.capitalize():**
   However, if the string is treated as an exact match, the selection in ``DataFrame``'s ``[]`` will be column-wise and not row-wise, see `Indexing Basics <indexing.basics>`. For example ``dft_minute['2011-12-31 23:59']`` will raise ``KeyError`` as ``'2012-12-31 23:59'`` has the same resolution as the index and there is no column with such name:

   To *always* have unambiguous selection, whether the row is treated as a slice or a single selection, use ``.loc``.

   ```python
dft_minute.loc["2011-12-31 23:59"]
```
Note also that ``DatetimeIndex`` resolution cannot be less precise than day.

```python
series_monthly = pd.Series(
    [1, 2, 3], pd.DatetimeIndex(["2011-12", "2012-01", "2012-02"])
)
series_monthly.index.resolution
series_monthly["2011-12"]  # returns Series
```
### Exact indexing
As discussed in previous section, indexing a ``DatetimeIndex`` with a partial string depends on the "accuracy" of the period, in other words how specific the interval is in relation to the resolution of the index. In contrast, indexing with ``Timestamp`` or ``datetime`` objects is exact, because the objects have exact meaning. These also follow the semantics of *including both endpoints*.

These ``Timestamp`` and ``datetime`` objects have exact ``hours, minutes,`` and ``seconds``, even though they were not explicitly specified (they are ``0``).

```python
dft[datetime.datetime(2013, 1, 1): datetime.datetime(2013, 2, 28)]
```
With no defaults.

```python
dft[
    datetime.datetime(2013, 1, 1, 10, 12, 0): datetime.datetime(
        2013, 2, 28, 10, 12, 0
    )
]
```
### Truncating & fancy indexing
A `~DataFrame.truncate` convenience function is provided that is similar
to slicing. Note that ``truncate`` assumes a 0 value for any unspecified date
component in a ``DatetimeIndex`` in contrast to slicing which returns any
partially matching dates:

```python
rng2 = pd.date_range("2011-01-01", "2012-01-01", freq="W")
ts2 = pd.Series(np.random.randn(len(rng2)), index=rng2)

ts2.truncate(before="2011-11", after="2011-12")
ts2["2011-11":"2011-12"]
```
Even complicated fancy indexing that breaks the ``DatetimeIndex`` frequency
regularity will result in a ``DatetimeIndex``, although frequency is lost:

```python
ts2.iloc[[0, 2, 6]].index
```


## Time/date components
There are several time/date properties that one can access from ``Timestamp`` or a collection of timestamps like a ``DatetimeIndex``.


    :header: "Property", "Description"
    :widths: 15, 65

    year, "The year of the datetime"
    month,"The month of the datetime"
    day,"The days of the datetime"
    hour,"The hour of the datetime"
    minute,"The minutes of the datetime"
    second,"The seconds of the datetime"
    microsecond,"The microseconds of the datetime"
    nanosecond,"The nanoseconds of the datetime"
    date,"Returns datetime.date (does not contain timezone information)"
    time,"Returns datetime.time (does not contain timezone information)"
    timetz,"Returns datetime.time as local time with timezone information"
    dayofyear,"The ordinal day of year"
    day_of_year,"The ordinal day of year"
    dayofweek,"The number of the day of the week with Monday=0, Sunday=6"
    day_of_week,"The number of the day of the week with Monday=0, Sunday=6"
    weekday,"The number of the day of the week with Monday=0, Sunday=6"
    quarter,"Quarter of the date: Jan-Mar = 1, Apr-Jun = 2, etc."
    days_in_month,"The number of days in the month of the datetime"
    is_month_start,"Logical indicating if first day of month (defined by frequency)"
    is_month_end,"Logical indicating if last day of month (defined by frequency)"
    is_quarter_start,"Logical indicating if first day of quarter (defined by frequency)"
    is_quarter_end,"Logical indicating if last day of quarter (defined by frequency)"
    is_year_start,"Logical indicating if first day of year (defined by frequency)"
    is_year_end,"Logical indicating if last day of year (defined by frequency)"
    is_leap_year,"Logical indicating if the date belongs to a leap year"

> **note.capitalize():**
   You can use ``DatetimeIndex.isocalendar().week`` to access week of year date information.

Furthermore, if you have a ``Series`` with datetimelike values, then you can
access these properties via the ``.dt`` accessor, as detailed in the section
on `.dt accessors<basics.dt_accessors>`.

You may obtain the year, week and day components of the ISO year from the ISO 8601 standard:

```python
idx = pd.date_range(start="2019-12-29", freq="D", periods=4)
idx.isocalendar()
idx.to_series().dt.isocalendar()
```


## DateOffset objects
In the preceding examples, frequency strings (e.g. ``'D'``) were used to specify
a frequency that defined:

* how the date times in `DatetimeIndex` were spaced when using `date_range`
* the frequency of a `Period` or `PeriodIndex`

These frequency strings map to a `DateOffset` object and its subclasses. A `DateOffset`
is similar to a `Timedelta` that represents a duration of time but follows specific calendar duration rules.
For example, a `Timedelta` day will always increment ``datetimes`` by 24 hours, while a `DateOffset` day
will increment ``datetimes`` to the same time the next day whether a day represents 23, 24 or 25 hours due to daylight
savings time. However, all `DateOffset` subclasses that are an hour or smaller
(``Hour``, ``Minute``, ``Second``, ``Milli``, ``Micro``, ``Nano``) behave like
`Timedelta` and respect absolute time.

The basic `DateOffset` acts similar to ``dateutil.relativedelta`` (`relativedelta documentation`_)
that shifts a date time by the corresponding calendar duration specified. The
arithmetic operator (``+``) can be used to perform the shift.

```python
# This particular day contains a day light savings time transition
ts = pd.Timestamp("2016-10-30 00:00:00", tz="Europe/Helsinki")
# Respects absolute time
ts + pd.Timedelta(days=1)
# Respects calendar time
ts + pd.DateOffset(days=1)
friday = pd.Timestamp("2018-01-05")
friday.day_name()
# Add 2 business days (Friday --> Tuesday)
two_business_days = 2 * pd.offsets.BDay()
friday + two_business_days
(friday + two_business_days).day_name()
```
Most ``DateOffsets`` have associated frequencies strings, or offset aliases, that can be passed
into ``freq`` keyword arguments. The available date offsets and associated frequency strings can be found below:


    :header: "Date Offset", "Frequency String", "Description"
    :widths: 15, 15, 65

    `~pandas.tseries.offsets.DateOffset`, None, "Generic offset class, defaults to absolute 24 hours"
    `~pandas.tseries.offsets.BDay` or `~pandas.tseries.offsets.BusinessDay`, ``'B'``,"business day (weekday)"
    `~pandas.tseries.offsets.CDay` or `~pandas.tseries.offsets.CustomBusinessDay`, ``'C'``, "custom business day"
    `~pandas.tseries.offsets.Week`, ``'W'``, "one week, optionally anchored on a day of the week"
    `~pandas.tseries.offsets.WeekOfMonth`, ``'WOM'``, "the x-th day of the y-th week of each month"
    `~pandas.tseries.offsets.LastWeekOfMonth`, ``'LWOM'``, "the x-th day of the last week of each month"
    `~pandas.tseries.offsets.MonthEnd`, ``'ME'``, "calendar month end"
    `~pandas.tseries.offsets.MonthBegin`, ``'MS'``, "calendar month begin"
    `~pandas.tseries.offsets.BMonthEnd` or `~pandas.tseries.offsets.BusinessMonthEnd`, ``'BME'``, "business month end"
    `~pandas.tseries.offsets.BMonthBegin` or `~pandas.tseries.offsets.BusinessMonthBegin`, ``'BMS'``, "business month begin"
    `~pandas.tseries.offsets.CBMonthEnd` or `~pandas.tseries.offsets.CustomBusinessMonthEnd`, ``'CBME'``, "custom business month end"
    `~pandas.tseries.offsets.CBMonthBegin` or `~pandas.tseries.offsets.CustomBusinessMonthBegin`, ``'CBMS'``, "custom business month begin"
    `~pandas.tseries.offsets.SemiMonthEnd`, ``'SME'``, "15th (or other day_of_month) and calendar month end"
    `~pandas.tseries.offsets.SemiMonthBegin`, ``'SMS'``, "15th (or other day_of_month) and calendar month begin"
    `~pandas.tseries.offsets.QuarterEnd`, ``'QE'``, "calendar quarter end"
    `~pandas.tseries.offsets.QuarterBegin`, ``'QS'``, "calendar quarter begin"
    `~pandas.tseries.offsets.BQuarterEnd`, ``'BQE``, "business quarter end"
    `~pandas.tseries.offsets.BQuarterBegin`, ``'BQS'``, "business quarter begin"
    `~pandas.tseries.offsets.FY5253Quarter`, ``'REQ'``, "retail (aka 52-53 week) quarter"
    `~pandas.tseries.offsets.HalfYearEnd`, ``'HYE'``, "calendar half year end"
    `~pandas.tseries.offsets.HalfYearBegin`, ``'HYS'``, "calendar half year begin"
    `~pandas.tseries.offsets.BHalfYearEnd`, ``'BHYE``, "business half year end"
    `~pandas.tseries.offsets.BHalfYearBegin`, ``'BHYS'``, "business half year begin"
    `~pandas.tseries.offsets.YearEnd`, ``'YE'``, "calendar year end"
    `~pandas.tseries.offsets.YearBegin`, ``'YS'`` or ``'BYS'``,"calendar year begin"
    `~pandas.tseries.offsets.BYearEnd`, ``'BYE'``, "business year end"
    `~pandas.tseries.offsets.BYearBegin`, ``'BYS'``, "business year begin"
    `~pandas.tseries.offsets.FY5253`, ``'RE'``, "retail (aka 52-53 week) year"
    `~pandas.tseries.offsets.Easter`, None, "Easter holiday"
    `~pandas.tseries.offsets.BusinessHour`, ``'bh'``, "business hour"
    `~pandas.tseries.offsets.CustomBusinessHour`, ``'cbh'``, "custom business hour"
    `~pandas.tseries.offsets.Day`, ``'D'``, "one calendar day"
    `~pandas.tseries.offsets.Hour`, ``'h'``, "one hour"
    `~pandas.tseries.offsets.Minute`, ``'min'``,"one minute"
    `~pandas.tseries.offsets.Second`, ``'s'``, "one second"
    `~pandas.tseries.offsets.Milli`, ``'ms'``, "one millisecond"
    `~pandas.tseries.offsets.Micro`, ``'us'``, "one microsecond"
    `~pandas.tseries.offsets.Nano`, ``'ns'``, "one nanosecond"

``DateOffsets`` additionally have `rollforward` and `rollback`
methods for moving a date forward or backward respectively to a valid offset
date relative to the offset. For example, business offsets will roll dates
that land on the weekends (Saturday and Sunday) forward to Monday since
business offsets operate on the weekdays.

```python
ts = pd.Timestamp("2018-01-06 00:00:00")
ts.day_name()
# BusinessHour's valid offset dates are Monday through Friday
offset = pd.offsets.BusinessHour(start="09:00")
# Bring the date to the closest offset date (Monday)
offset.rollforward(ts)
# Date is brought to the closest offset date first and then the hour is added
ts + offset
```
These operations preserve time (hour, minute, etc) information by default.
To reset time to midnight, use `normalize` before or after applying
the operation (depending on whether you want the time information included
in the operation).

```python
ts = pd.Timestamp("2014-01-01 09:00")
day = pd.offsets.Day()
day + ts
(day + ts).normalize()

ts = pd.Timestamp("2014-01-01 22:00")
hour = pd.offsets.Hour()
hour + ts
(hour + ts).normalize()
(hour + pd.Timestamp("2014-01-01 23:30")).normalize()
```
.. _relativedelta documentation: https://dateutil.readthedocs.io/en/stable/relativedelta.html


### Parametric offsets
Some of the offsets can be "parameterized" when created to result in different
behaviors. For example, the ``Week`` offset for generating weekly data accepts a
``weekday`` parameter which results in the generated dates always lying on a
particular day of the week:

```python
d = datetime.datetime(2008, 8, 18, 9, 0)
d
d + pd.offsets.Week()
d + pd.offsets.Week(weekday=4)
(d + pd.offsets.Week(weekday=4)).weekday()

d - pd.offsets.Week()
```
The ``normalize`` option will be effective for addition and subtraction.

```python
d + pd.offsets.Week(normalize=True)
d - pd.offsets.Week(normalize=True)
```
Another example is parameterizing ``YearEnd`` with the specific ending month:

```python
d + pd.offsets.YearEnd()
d + pd.offsets.YearEnd(month=6)
```


### Using offsets with ``Series`` / ``DatetimeIndex``
Offsets can be used with either a ``Series`` or ``DatetimeIndex`` to
apply the offset to each element.

```python
rng = pd.date_range("2012-01-01", "2012-01-03")
s = pd.Series(rng)
rng
rng + pd.DateOffset(months=2)
s + pd.DateOffset(months=2)
s - pd.DateOffset(months=2)
```
If the offset class maps directly to a ``Timedelta`` (``Hour``,
``Minute``, ``Second``, ``Micro``, ``Milli``, ``Nano``) it can be
used exactly like a ``Timedelta`` - see the
`Timedelta section<timedeltas.operations>` for more examples.

```python
s - pd.offsets.Day(2)
td = s - pd.Series(pd.date_range("2011-12-29", "2011-12-31"))
td
td + pd.offsets.Minute(15)
```
Note that some offsets (such as ``BQuarterEnd``) do not have a
vectorized implementation.  They can still be used but may
calculate significantly slower and will show a ``PerformanceWarning``

```python
:okwarning:

rng + pd.offsets.BQuarterEnd()
```


### Custom business days
The ``CDay`` or ``CustomBusinessDay`` class provides a parametric
``BusinessDay`` class which can be used to create customized business day
calendars which account for local holidays and local weekend conventions.

As an interesting example, let's look at Egypt where a Friday-Saturday weekend is observed.

```python
weekmask_egypt = "Sun Mon Tue Wed Thu"

# They also observe International Workers' Day so let's
# add that for a couple of years

holidays = [
    "2012-05-01",
    datetime.datetime(2013, 5, 1),
    np.datetime64("2014-05-01"),
]
bday_egypt = pd.offsets.CustomBusinessDay(
    holidays=holidays,
    weekmask=weekmask_egypt,
)
dt = datetime.datetime(2013, 4, 30)
dt + 2 * bday_egypt
```
Let's map to the weekday names:

```python
dts = pd.date_range(dt, periods=5, freq=bday_egypt)

pd.Series(dts.weekday, dts).map(pd.Series("Mon Tue Wed Thu Fri Sat Sun".split()))
```
Holiday calendars can be used to provide the list of holidays.  See the
`holiday calendar<timeseries.holiday>` section for more information.

```python
from pandas.tseries.holiday import USFederalHolidayCalendar

bday_us = pd.offsets.CustomBusinessDay(calendar=USFederalHolidayCalendar())

# Friday before MLK Day
dt = datetime.datetime(2014, 1, 17)

# Tuesday after MLK Day (Monday is skipped because it's a holiday)
dt + bday_us
```
Monthly offsets that respect a certain holiday calendar can be defined
in the usual way.

```python
bmth_us = pd.offsets.CustomBusinessMonthBegin(calendar=USFederalHolidayCalendar())

# Skip new years
dt = datetime.datetime(2013, 12, 17)
dt + bmth_us

# Define date index with custom offset
pd.date_range(start="20100101", end="20120101", freq=bmth_us)
```
> **note.capitalize():**
    The frequency string 'C' is used to indicate that a CustomBusinessDay
    DateOffset is used, it is important to note that since CustomBusinessDay is
    a parameterised type, instances of CustomBusinessDay may differ and this is
    not detectable from the 'C' frequency string. The user therefore needs to
    ensure that the 'C' frequency string is used consistently within the user's
    application.



### Business hour
The ``BusinessHour`` class provides a business hour representation on ``BusinessDay``,
allowing to use specific start and end times.

By default, ``BusinessHour`` uses 9:00 - 17:00 as business hours.
Adding ``BusinessHour`` will increment ``Timestamp`` by hourly frequency.
If target ``Timestamp`` is out of business hours, move to the next business hour
then increment it. If the result exceeds the business hours end, the remaining
hours are added to the next business day.

```python
bh = pd.offsets.BusinessHour()
bh

# 2014-08-01 is Friday
pd.Timestamp("2014-08-01 10:00").weekday()
pd.Timestamp("2014-08-01 10:00") + bh

# Below example is the same as: pd.Timestamp('2014-08-01 09:00') + bh
pd.Timestamp("2014-08-01 08:00") + bh

# If the results is on the end time, move to the next business day
pd.Timestamp("2014-08-01 16:00") + bh

# Remainings are added to the next day
pd.Timestamp("2014-08-01 16:30") + bh

# Adding 2 business hours
pd.Timestamp("2014-08-01 10:00") + pd.offsets.BusinessHour(2)

# Subtracting 3 business hours
pd.Timestamp("2014-08-01 10:00") + pd.offsets.BusinessHour(-3)
```
You can also specify ``start`` and ``end`` time by keywords. The argument must
be a ``str`` with an ``hour:minute`` representation or a ``datetime.time``
instance. Specifying seconds, microseconds and nanoseconds as business hour
results in ``ValueError``.

```python
bh = pd.offsets.BusinessHour(start="11:00", end=datetime.time(20, 0))
bh

pd.Timestamp("2014-08-01 13:00") + bh
pd.Timestamp("2014-08-01 09:00") + bh
pd.Timestamp("2014-08-01 18:00") + bh
```
Passing ``start`` time later than ``end`` represents midnight business hour.
In this case, business hour exceeds midnight and overlap to the next day.
Valid business hours are distinguished by whether it started from valid ``BusinessDay``.

```python
bh = pd.offsets.BusinessHour(start="17:00", end="09:00")
bh

pd.Timestamp("2014-08-01 17:00") + bh
pd.Timestamp("2014-08-01 23:00") + bh

# Although 2014-08-02 is Saturday,
# it is valid because it starts from 08-01 (Friday).
pd.Timestamp("2014-08-02 04:00") + bh

# Although 2014-08-04 is Monday,
# it is out of business hours because it starts from 08-03 (Sunday).
pd.Timestamp("2014-08-04 04:00") + bh
```
Applying ``BusinessHour.rollforward`` and ``rollback`` to out of business hours results in
the next business hour start or previous day's end. Different from other offsets, ``BusinessHour.rollforward``
may output different results from ``apply`` by definition.

This is because one day's business hour end is equal to next day's business hour start. For example,
under the default business hours (9:00 - 17:00), there is no gap (0 minutes) between ``2014-08-01 17:00`` and
``2014-08-04 09:00``.

```python
# This adjusts a Timestamp to business hour edge
pd.offsets.BusinessHour().rollback(pd.Timestamp("2014-08-02 15:00"))
pd.offsets.BusinessHour().rollforward(pd.Timestamp("2014-08-02 15:00"))

# It is the same as BusinessHour() + pd.Timestamp('2014-08-01 17:00').
# And it is the same as BusinessHour() + pd.Timestamp('2014-08-04 09:00')
pd.offsets.BusinessHour() + pd.Timestamp("2014-08-02 15:00")

# BusinessDay results (for reference)
pd.offsets.BusinessHour().rollforward(pd.Timestamp("2014-08-02"))

# It is the same as BusinessDay() + pd.Timestamp('2014-08-01')
# The result is the same as rollworward because BusinessDay never overlap.
pd.offsets.BusinessHour() + pd.Timestamp("2014-08-02")
```
``BusinessHour`` regards Saturday and Sunday as holidays. To use arbitrary
holidays, you can use ``CustomBusinessHour`` offset, as explained in the
following subsection.



### Custom business hour
The ``CustomBusinessHour`` is a mixture of ``BusinessHour`` and ``CustomBusinessDay`` which
allows you to specify arbitrary holidays. ``CustomBusinessHour`` works as the same
as ``BusinessHour`` except that it skips specified custom holidays.

```python
from pandas.tseries.holiday import USFederalHolidayCalendar

bhour_us = pd.offsets.CustomBusinessHour(calendar=USFederalHolidayCalendar())
# Friday before MLK Day
dt = datetime.datetime(2014, 1, 17, 15)

dt + bhour_us

# Tuesday after MLK Day (Monday is skipped because it's a holiday)
dt + bhour_us * 2
```
You can use keyword arguments supported by either ``BusinessHour`` and ``CustomBusinessDay``.

```python
bhour_mon = pd.offsets.CustomBusinessHour(start="10:00", weekmask="Tue Wed Thu Fri")

# Monday is skipped because it's a holiday, business hour starts from 10:00
dt + bhour_mon * 2
```


### Offset aliases
A number of string aliases are given to useful common time series
frequencies. We will refer to these aliases as *offset aliases*.


    :header: "Alias", "Description"
    :widths: 15, 100

    "B", "business day frequency"
    "C", "custom business day frequency"
    "D", "calendar day frequency"
    "W", "weekly frequency"
    "ME", "month end frequency"
    "SME", "semi-month end frequency (15th and end of month)"
    "BME", "business month end frequency"
    "CBME", "custom business month end frequency"
    "MS", "month start frequency"
    "SMS", "semi-month start frequency (1st and 15th)"
    "BMS", "business month start frequency"
    "CBMS", "custom business month start frequency"
    "QE", "quarter end frequency"
    "BQE", "business quarter end frequency"
    "QS", "quarter start frequency"
    "BQS", "business quarter start frequency"
    "YE", "year end frequency"
    "BYE", "business year end frequency"
    "YS", "year start frequency"
    "BYS", "business year start frequency"
    "h", "hourly frequency"
    "bh", "business hour frequency"
    "cbh", "custom business hour frequency"
    "min", "minutely frequency"
    "s", "secondly frequency"
    "ms", "milliseconds"
    "us", "microseconds"
    "ns", "nanoseconds"



   Aliases ``H``, ``BH``, ``CBH``, ``T``, ``S``, ``L``, ``U``, and ``N``
   are deprecated in favour of the aliases ``h``, ``bh``, ``cbh``,
   ``min``, ``s``, ``ms``, ``us``, and ``ns``.

   Aliases ``Y``, ``M``, and ``Q`` are deprecated in favour of the aliases
   ``YE``, ``ME``, ``QE``.


> **note.capitalize():**
    When using the offset aliases above, it should be noted that functions
    such as `date_range`, `bdate_range`, will only return
    timestamps that are in the interval defined by ``start_date`` and
    ``end_date``. If the ``start_date`` does not correspond to the frequency,
    the returned timestamps will start at the next valid timestamp, same for
    ``end_date``, the returned timestamps will stop at the previous valid
    timestamp.

   For example, for the offset ``MS``, if the ``start_date`` is not the first
   of the month, the returned timestamps will start with the first day of the
   next month. If ``end_date`` is not the first day of a month, the last
   returned timestamp will be the first day of the corresponding month.

   ```python
dates_lst_1 = pd.date_range("2020-01-06", "2020-04-03", freq="MS")
    dates_lst_1

    dates_lst_2 = pd.date_range("2020-01-01", "2020-04-01", freq="MS")
    dates_lst_2

We can see in the above example `date_range` and
`bdate_range` will only return the valid timestamps between the
``start_date`` and ``end_date``. If these are not valid timestamps for the
given frequency it will roll to the next value for ``start_date``
(respectively previous for the ``end_date``)
```


### Period aliases
A number of string aliases are given to useful common time series
frequencies. We will refer to these aliases as *period aliases*.


    :header: "Alias", "Description"
    :widths: 15, 100

    "B", "business day frequency"
    "D", "calendar day frequency"
    "W", "weekly frequency"
    "M", "monthly frequency"
    "Q", "quarterly frequency"
    "Y", "yearly frequency"
    "h", "hourly frequency"
    "min", "minutely frequency"
    "s", "secondly frequency"
    "ms", "milliseconds"
    "us", "microseconds"
    "ns", "nanoseconds"



   Aliases ``H``, ``T``, ``S``, ``L``, ``U``, and ``N`` are deprecated in favour of the aliases
   ``h``, ``min``, ``s``, ``ms``, ``us``, and ``ns``.


### Combining aliases
As we have seen previously, the alias and the offset instance are fungible in
most functions:

```python
pd.date_range(start, periods=5, freq="B")

pd.date_range(start, periods=5, freq=pd.offsets.BDay())
```
You can combine together day and intraday offsets:

```python
pd.date_range(start, periods=10, freq="2h20min")

pd.date_range(start, periods=10, freq="1D10us")
```
### Anchored offsets
For some frequencies you can specify an anchoring suffix:


    :header: "Alias", "Description"
    :widths: 15, 100

    "W\-SUN", "weekly frequency (Sundays). Same as 'W'"
    "W\-MON", "weekly frequency (Mondays)"
    "W\-TUE", "weekly frequency (Tuesdays)"
    "W\-WED", "weekly frequency (Wednesdays)"
    "W\-THU", "weekly frequency (Thursdays)"
    "W\-FRI", "weekly frequency (Fridays)"
    "W\-SAT", "weekly frequency (Saturdays)"
    "(B)Q(E)(S)\-DEC", "quarterly frequency, year ends in December. Same as 'QE'"
    "(B)Q(E)(S)\-JAN", "quarterly frequency, year ends in January"
    "(B)Q(E)(S)\-FEB", "quarterly frequency, year ends in February"
    "(B)Q(E)(S)\-MAR", "quarterly frequency, year ends in March"
    "(B)Q(E)(S)\-APR", "quarterly frequency, year ends in April"
    "(B)Q(E)(S)\-MAY", "quarterly frequency, year ends in May"
    "(B)Q(E)(S)\-JUN", "quarterly frequency, year ends in June"
    "(B)Q(E)(S)\-JUL", "quarterly frequency, year ends in July"
    "(B)Q(E)(S)\-AUG", "quarterly frequency, year ends in August"
    "(B)Q(E)(S)\-SEP", "quarterly frequency, year ends in September"
    "(B)Q(E)(S)\-OCT", "quarterly frequency, year ends in October"
    "(B)Q(E)(S)\-NOV", "quarterly frequency, year ends in November"
    "(B)Y(E)(S)\-DEC", "annual frequency, anchored end of December. Same as 'YE'"
    "(B)Y(E)(S)\-JAN", "annual frequency, anchored end of January"
    "(B)Y(E)(S)\-FEB", "annual frequency, anchored end of February"
    "(B)Y(E)(S)\-MAR", "annual frequency, anchored end of March"
    "(B)Y(E)(S)\-APR", "annual frequency, anchored end of April"
    "(B)Y(E)(S)\-MAY", "annual frequency, anchored end of May"
    "(B)Y(E)(S)\-JUN", "annual frequency, anchored end of June"
    "(B)Y(E)(S)\-JUL", "annual frequency, anchored end of July"
    "(B)Y(E)(S)\-AUG", "annual frequency, anchored end of August"
    "(B)Y(E)(S)\-SEP", "annual frequency, anchored end of September"
    "(B)Y(E)(S)\-OCT", "annual frequency, anchored end of October"
    "(B)Y(E)(S)\-NOV", "annual frequency, anchored end of November"

These can be used as arguments to ``date_range``, ``bdate_range``, constructors
for ``DatetimeIndex``, as well as various other timeseries-related functions
in pandas.

### Anchored offset semantics
For those offsets that are anchored to the start or end of specific
frequency (``MonthEnd``, ``MonthBegin``, ``WeekEnd``, etc), the following
rules apply to rolling forward and backwards.

When ``n`` is not 0, if the given date is not on an anchor point, it snapped to the next(previous)
anchor point, and moved ``|n|-1`` additional steps forwards or backwards.

```python
pd.Timestamp("2014-01-02") + pd.offsets.MonthBegin(n=1)
pd.Timestamp("2014-01-02") + pd.offsets.MonthEnd(n=1)

pd.Timestamp("2014-01-02") - pd.offsets.MonthBegin(n=1)
pd.Timestamp("2014-01-02") - pd.offsets.MonthEnd(n=1)

pd.Timestamp("2014-01-02") + pd.offsets.MonthBegin(n=4)
pd.Timestamp("2014-01-02") - pd.offsets.MonthBegin(n=4)
```
If the given date *is* on an anchor point, it is moved ``|n|`` points forwards
or backwards.

```python
pd.Timestamp("2014-01-01") + pd.offsets.MonthBegin(n=1)
pd.Timestamp("2014-01-31") + pd.offsets.MonthEnd(n=1)

pd.Timestamp("2014-01-01") - pd.offsets.MonthBegin(n=1)
pd.Timestamp("2014-01-31") - pd.offsets.MonthEnd(n=1)

pd.Timestamp("2014-01-01") + pd.offsets.MonthBegin(n=4)
pd.Timestamp("2014-01-31") - pd.offsets.MonthBegin(n=4)
```
For the case when ``n=0``, the date is not moved if on an anchor point, otherwise
it is rolled forward to the next anchor point.

```python
pd.Timestamp("2014-01-02") + pd.offsets.MonthBegin(n=0)
pd.Timestamp("2014-01-02") + pd.offsets.MonthEnd(n=0)

pd.Timestamp("2014-01-01") + pd.offsets.MonthBegin(n=0)
pd.Timestamp("2014-01-31") + pd.offsets.MonthEnd(n=0)
```


### Holidays / holiday calendars
Holidays and calendars provide a simple way to define holiday rules to be used
with ``CustomBusinessDay`` or in other analysis that requires a predefined
set of holidays.  The ``AbstractHolidayCalendar`` class provides all the necessary
methods to return a list of holidays and only ``rules`` need to be defined
in a specific holiday calendar class. Furthermore, the ``start_date`` and ``end_date``
class attributes determine over what date range holidays are generated.  These
should be overwritten on the ``AbstractHolidayCalendar`` class to have the range
apply to all calendar subclasses.  ``USFederalHolidayCalendar`` is the
only calendar that exists and primarily serves as an example for developing
other calendars.

For holidays that occur on fixed dates (e.g., US Memorial Day or July 4th) an
observance rule determines when that holiday is observed if it falls on a weekend
or some other non-observed day.  Defined observance rules are:


    :header: "Rule", "Description"
    :widths: 15, 70

    "next_workday", "move Saturday and Sunday to Monday"
    "previous_workday", "move Saturday and Sunday to Friday"
    "nearest_workday", "move Saturday to Friday and Sunday to Monday"
    "before_nearest_workday", "apply ``nearest_workday`` and then move to previous workday before that day"
    "after_nearest_workday", "apply ``nearest_workday`` and then move to next workday after that day"
    "sunday_to_monday", "move Sunday to following Monday"
    "next_monday_or_tuesday", "move Saturday to Monday and Sunday/Monday to Tuesday"
    "previous_friday", "move Saturday and Sunday to previous Friday"
    "next_monday", "move Saturday and Sunday to following Monday"
    "weekend_to_monday", "same as ``next_monday``"

An example of how holidays and holiday calendars are defined:

```python
from pandas.tseries.holiday import (
    Holiday,
    USMemorialDay,
    AbstractHolidayCalendar,
    nearest_workday,
    MO,
)

class ExampleCalendar(AbstractHolidayCalendar):
    rules = [
        USMemorialDay,
        Holiday("July 4th", month=7, day=4, observance=nearest_workday),
        Holiday(
            "Columbus Day",
            month=10,
            day=1,
            offset=pd.DateOffset(weekday=MO(2)),
        ),
    ]

cal = ExampleCalendar()
cal.holidays(datetime.datetime(2012, 1, 1), datetime.datetime(2012, 12, 31))
```
:hint:
   **weekday=MO(2)** is same as **2 * Week(weekday=2)**

Using this calendar, creating an index or doing offset arithmetic skips weekends
and holidays (i.e., Memorial Day/July 4th).  For example, the below defines
a custom business day offset using the ``ExampleCalendar``.  Like any other offset,
it can be used to create a ``DatetimeIndex`` or added to ``datetime``
or ``Timestamp`` objects.

```python
pd.date_range(
    start="7/1/2012", end="7/10/2012", freq=pd.offsets.CDay(calendar=cal)
).to_pydatetime()
offset = pd.offsets.CustomBusinessDay(calendar=cal)
datetime.datetime(2012, 5, 25) + offset
datetime.datetime(2012, 7, 3) + offset
datetime.datetime(2012, 7, 3) + 2 * offset
datetime.datetime(2012, 7, 6) + offset
```
Ranges are defined by the ``start_date`` and ``end_date`` class attributes
of ``AbstractHolidayCalendar``.  The defaults are shown below.

```python
AbstractHolidayCalendar.start_date
AbstractHolidayCalendar.end_date
```
These dates can be overwritten by setting the attributes as
datetime/Timestamp/string.

```python
AbstractHolidayCalendar.start_date = datetime.datetime(2012, 1, 1)
AbstractHolidayCalendar.end_date = datetime.datetime(2012, 12, 31)
cal.holidays()
```
Every calendar class is accessible by name using the ``get_calendar`` function
which returns a holiday class instance.  Any imported calendar class will
automatically be available by this function.  Also, ``HolidayCalendarFactory``
provides an easy interface to create calendars that are combinations of calendars
or calendars with additional rules.

```python
from pandas.tseries.holiday import get_calendar, HolidayCalendarFactory, USLaborDay

cal = get_calendar("ExampleCalendar")
cal.rules
new_cal = HolidayCalendarFactory("NewExampleCalendar", cal, USLaborDay)
new_cal.rules
```


## Time Series-related instance methods
### Shifting / lagging
One may want to *shift* or *lag* the values in a time series back and forward in
time. The method for this is `~Series.shift`, which is available on all of
the pandas objects.

```python
ts = pd.Series(range(len(rng)), index=rng)
ts = ts[:5]
ts.shift(1)
```
The ``shift`` method accepts a ``freq`` argument which can accept a
``DateOffset`` class or other ``timedelta``-like object or also an
`offset alias <timeseries.offset_aliases>`.

When ``freq`` is specified, ``shift`` method changes all the dates in the index
rather than changing the alignment of the data and the index:

```python
ts.shift(5, freq="D")
ts.shift(5, freq=pd.offsets.BDay())
ts.shift(5, freq="BME")
```
Note that with when ``freq`` is specified, the leading entry is no longer NaN
because the data is not being realigned.

### Frequency conversion
The primary function for changing frequencies is the `~Series.asfreq`
method. For a ``DatetimeIndex``, this is basically just a thin, but convenient
wrapper around `~Series.reindex`  which generates a ``date_range`` and
calls ``reindex``.

```python
dr = pd.date_range("1/1/2010", periods=3, freq=3 * pd.offsets.BDay())
ts = pd.Series(np.random.randn(3), index=dr)
ts
ts.asfreq(pd.offsets.BDay())
```
``asfreq`` provides a further convenience so you can specify an interpolation
method for any gaps that may appear after the frequency conversion.

```python
ts.asfreq(pd.offsets.BDay(), method="pad")
```
### Filling forward / backward
Related to ``asfreq`` and ``reindex`` is `~Series.fillna`, which is
documented in the `missing data section <missing_data.fillna>`.

### Converting to Python datetimes
``DatetimeIndex`` can be converted to an array of Python native
:py`datetime.datetime` objects using the ``to_pydatetime`` method.



## Resampling
pandas has a simple, powerful, and efficient functionality for performing
resampling operations during frequency conversion (e.g., converting secondly
data into 5-minutely data). This is extremely common in, but not limited to,
financial applications.

`~Series.resample` is a time-based groupby, followed by a reduction method
on each of its groups. See some `cookbook examples <cookbook.resample>` for
some advanced strategies.

The ``resample()`` method can be used directly from ``DataFrameGroupBy`` objects,
see the `groupby docs <groupby.transform.window_resample>`.

### Basics
```python
rng = pd.date_range("1/1/2012", periods=100, freq="s")

ts = pd.Series(np.random.randint(0, 500, len(rng)), index=rng)

ts.resample("5Min").sum()
```
The ``resample`` function is very flexible and allows you to specify many
different parameters to control the frequency conversion and resampling
operation.

Any built-in method available via `GroupBy <api.groupby>` is available as
a method of the returned object, including ``sum``, ``mean``, ``std``, ``sem``,
``max``, ``min``, ``median``, ``first``, ``last``, ``ohlc``:

```python
ts.resample("5Min").mean()

ts.resample("5Min").ohlc()

ts.resample("5Min").max()
```
For downsampling, ``closed`` can be set to 'left' or 'right' to specify which
end of the interval is closed:

```python
ts.resample("5Min", closed="right").mean()

ts.resample("5Min", closed="left").mean()
```
Parameters like ``label`` are used to manipulate the resulting labels.
``label`` specifies whether the result is labeled with the beginning or
the end of the interval.

```python
ts.resample("5Min").mean()  # by default label='left'

ts.resample("5Min", label="left").mean()
```
> **warning.capitalize():**
    The default values for ``label`` and ``closed`` is '**left**' for all
    frequency offsets except for 'ME', 'YE', 'QE', 'BME', 'BYE', 'BQE', and 'W'
    which all have a default of 'right'.

    This might unintendedly lead to looking ahead, where the value for a later
    time is pulled back to a previous time as in the following example with
    the `~pandas.tseries.offsets.BusinessDay` frequency:

    ```python
s = pd.date_range("2000-01-01", "2000-01-05").to_series()
    s.iloc[2] = pd.NaT
    s.dt.day_name()

    # default: label='left', closed='left'
    s.resample("B").last().dt.day_name()

Notice how the value for Sunday got pulled back to the previous Friday.
To get the behavior where the value for Sunday is pushed to Monday, use
instead



    s.resample("B", label="right", closed="right").last().dt.day_name()
```
The ``axis`` parameter can be set to 0 or 1 and allows you to resample the
specified axis for a ``DataFrame``.

``kind`` can be set to 'timestamp' or 'period' to convert the resulting index
to/from timestamp and time span representations. By default ``resample``
retains the input representation.

``convention`` can be set to 'start' or 'end' when resampling period data
(detail below). It specifies how low frequency periods are converted to higher
frequency periods.


### Upsampling
For upsampling, you can specify a way to upsample and the ``limit`` parameter to interpolate over the gaps that are created:

```python
# from secondly to every 250 milliseconds

ts[:2].resample("250ms").asfreq()

ts[:2].resample("250ms").ffill()

ts[:2].resample("250ms").ffill(limit=2)
```
### Sparse resampling
Sparse timeseries are the ones where you have a lot fewer points relative
to the amount of time you are looking to resample. Naively upsampling a sparse
series can potentially generate lots of intermediate values. When you don't want
to use a method to fill these values, e.g. ``fill_method`` is ``None``, then
intermediate values will be filled with ``NaN``.

Since ``resample`` is a time-based groupby, the following is a method to efficiently
resample only the groups that are not all ``NaN``.

```python
rng = pd.date_range("2014-1-1", periods=100, freq="D") + pd.Timedelta("1s")
ts = pd.Series(range(100), index=rng)
```
If we want to resample to the full range of the series:

```python
ts.resample("3min").sum()
```
We can instead only resample those groups where we have points as follows:

```python
from functools import partial
from pandas.tseries.frequencies import to_offset

def round(t, freq):
    # round a Timestamp to a specified freq
    freq = to_offset(freq)
    td = pd.Timedelta(freq)
    return pd.Timestamp((t.value // td.value) * td.value)

ts.groupby(partial(round, freq="3min")).sum()
```


### Aggregation
The ``resample()`` method returns a ``pandas.api.typing.Resampler`` instance.  Similar to
the `aggregating API <basics.aggregate>`, `groupby API <groupby.aggregate>`,
and the `window API <window.overview>`, a ``Resampler`` can be selectively resampled.

Resampling a ``DataFrame``, the default will be to act on all columns with the same function.

```python
df = pd.DataFrame(
    np.random.randn(1000, 3),
    index=pd.date_range("1/1/2012", freq="s", periods=1000),
    columns=["A", "B", "C"],
)
r = df.resample("3min")
r.mean()
```
We can select a specific column or columns using standard getitem.

```python
r["A"].mean()

r[["A", "B"]].mean()
```
You can pass a list or dict of functions to do aggregation with, outputting a ``DataFrame``:

```python
r["A"].agg(["sum", "mean", "std"])
```
On a resampled ``DataFrame``, you can pass a list of functions to apply to each
column, which produces an aggregated result with a hierarchical index:

```python
r.agg(["sum", "mean"])
```
By passing a dict to ``aggregate`` you can apply a different aggregation to the
columns of a ``DataFrame``:

```python
:okexcept:

r.agg({"A": "sum", "B": lambda x: np.std(x, ddof=1)})
```
The function names can also be strings. In order for a string to be valid it
must be implemented on the resampled object:

```python
r.agg({"A": "sum", "B": "std"})
```
Furthermore, you can also specify multiple aggregation functions for each column separately.

```python
r.agg({"A": ["sum", "std"], "B": ["mean", "std"]})
```
If a ``DataFrame`` does not have a datetimelike index, but instead you want
to resample based on datetimelike column in the frame, it can passed to the
``on`` keyword.

```python
df = pd.DataFrame(
    {"date": pd.date_range("2015-01-01", freq="W", periods=5), "a": np.arange(5)},
    index=pd.MultiIndex.from_arrays(
        [[1, 2, 3, 4, 5], pd.date_range("2015-01-01", freq="W", periods=5)],
        names=["v", "d"],
    ),
)
df
df.resample("MS", on="date")[["a"]].sum()
```
Similarly, if you instead want to resample by a datetimelike
level of ``MultiIndex``, its name or location can be passed to the
``level`` keyword.

```python
df.resample("MS", level="d")[["a"]].sum()
```


### Iterating through groups
With the ``Resampler`` object in hand, iterating through the grouped data is very
natural and functions similarly to :py`itertools.groupby`:

```python
small = pd.Series(
    range(6),
    index=pd.to_datetime(
        [
            "2017-01-01T00:00:00",
            "2017-01-01T00:30:00",
            "2017-01-01T00:31:00",
            "2017-01-01T01:00:00",
            "2017-01-01T03:00:00",
            "2017-01-01T03:05:00",
        ]
    ),
)
resampled = small.resample("h")

for name, group in resampled:
    print("Group: ", name)
    print("-" * 27)
    print(group, end="\n\n")
```
See `groupby.iterating-label` or `Resampler.__iter__` for more.



### Use ``origin`` or ``offset`` to adjust the start of the bins
The bins of the grouping are adjusted based on the beginning of the day of the time series starting point. This works well with frequencies that are multiples of a day (like ``30D``) or that divide a day evenly (like ``90s`` or ``1min``). This can create inconsistencies with some frequencies that do not meet this criteria. To change this behavior you can specify a fixed Timestamp with the argument ``origin``.

For example:

```python
start, end = "2000-10-01 23:30:00", "2000-10-02 00:30:00"
middle = "2000-10-02 00:00:00"
rng = pd.date_range(start, end, freq="7min")
ts = pd.Series(np.arange(len(rng)) * 3, index=rng)
ts
```
Here we can see that, when using ``origin`` with its default value (``'start_day'``), the result after ``'2000-10-02 00:00:00'`` are not identical depending on the start of time series:

```python
ts.resample("17min", origin="start_day").sum()
ts[middle:end].resample("17min", origin="start_day").sum()
```
Here we can see that, when setting ``origin`` to ``'epoch'``, the result after ``'2000-10-02 00:00:00'`` are identical depending on the start of time series:

```python
ts.resample("17min", origin="epoch").sum()
ts[middle:end].resample("17min", origin="epoch").sum()
```
If needed you can use a custom timestamp for ``origin``:

```python
ts.resample("17min", origin="2001-01-01").sum()
ts[middle:end].resample("17min", origin=pd.Timestamp("2001-01-01")).sum()
```
If needed you can just adjust the bins with an ``offset`` Timedelta that would be added to the default ``origin``.
Those two examples are equivalent for this time series:

```python
ts.resample("17min", origin="start").sum()
ts.resample("17min", offset="23h30min").sum()
```
Note the use of ``'start'`` for ``origin`` on the last example. In that case, ``origin`` will be set to the first value of the timeseries.

### Backward resample


Instead of adjusting the beginning of bins, sometimes we need to fix the end of the bins to make a backward resample with a given ``freq``. The backward resample sets ``closed`` to ``'right'`` by default since the last value should be considered as the edge point for the last bin.

We can set ``origin`` to ``'end'``. The value for a specific ``Timestamp`` index stands for the resample result from the current ``Timestamp`` minus ``freq`` to the current ``Timestamp`` with a right close.

```python
ts.resample('17min', origin='end').sum()
```
Besides, in contrast with the ``'start_day'`` option, ``end_day`` is supported. This will set the origin as the ceiling midnight of the largest ``Timestamp``.

```python
ts.resample('17min', origin='end_day').sum()
```
The above result uses ``2000-10-02 00:29:00`` as the last bin's right edge since the following computation.

```python
ceil_mid = rng.max().ceil('D')
freq = pd.offsets.Minute(17)
bin_res = ceil_mid - freq * ((ceil_mid - rng.max()) // freq)
bin_res
```


## Time span representation
Regular intervals of time are represented by ``Period`` objects in pandas while
sequences of ``Period`` objects are collected in a ``PeriodIndex``, which can
be created with the convenience function ``period_range``.

### Period
A ``Period`` represents a span of time (e.g., a day, a month, a quarter, etc).
You can specify the span via ``freq`` keyword using a frequency alias like below.
Because ``freq`` represents a span of ``Period``, it cannot be negative like "-3D".

```python
pd.Period("2012", freq="Y-DEC")

pd.Period("2012-1-1", freq="D")

pd.Period("2012-1-1 19:00", freq="h")

pd.Period("2012-1-1 19:00", freq="5h")
```
Adding and subtracting integers from periods shifts the period by its own
frequency. Arithmetic is not allowed between ``Period`` with different ``freq`` (span).

```python
p = pd.Period("2012", freq="Y-DEC")
p + 1
p - 3
p = pd.Period("2012-01", freq="2M")
p + 2
p - 1
p == pd.Period("2012-01", freq="3M")
```
If ``Period`` freq is daily or higher (``D``, ``h``, ``min``, ``s``, ``ms``, ``us``, and ``ns``), ``offsets`` and ``timedelta``-like can be added if the result can have the same freq. Otherwise, ``ValueError`` will be raised.

```python
p = pd.Period("2014-07-01 09:00", freq="h")
p + pd.offsets.Hour(2)
p + datetime.timedelta(minutes=120)
p + np.timedelta64(7200, "s")
```
```python
:okexcept:

p + pd.offsets.Minute(5)
```
If ``Period`` has other frequencies, only the same ``offsets`` can be added. Otherwise, ``ValueError`` will be raised.

```python
p = pd.Period("2014-07", freq="M")
p + pd.offsets.MonthEnd(3)
```
```python
:okexcept:

p + pd.offsets.MonthBegin(3)
```
Taking the difference of ``Period`` instances with the same frequency will
return the number of frequency units between them:

```python
pd.Period("2012", freq="Y-DEC") - pd.Period("2002", freq="Y-DEC")
```
### PeriodIndex and period_range
Regular sequences of ``Period`` objects can be collected in a ``PeriodIndex``,
which can be constructed using the ``period_range`` convenience function:

```python
prng = pd.period_range("1/1/2011", "1/1/2012", freq="M")
prng
```
The ``PeriodIndex`` constructor can also be used directly:

```python
pd.PeriodIndex(["2011-1", "2011-2", "2011-3"], freq="M")
```
Passing multiplied frequency outputs a sequence of ``Period`` which
has multiplied span.

```python
pd.period_range(start="2014-01", freq="3M", periods=4)
```
If ``start`` or ``end`` are ``Period`` objects, they will be used as anchor
endpoints for a ``PeriodIndex`` with frequency matching that of the
``PeriodIndex`` constructor.

```python
pd.period_range(
    start=pd.Period("2017Q1", freq="Q"), end=pd.Period("2017Q2", freq="Q"), freq="M"
)
```
Just like ``DatetimeIndex``, a ``PeriodIndex`` can also be used to index pandas
objects:

```python
ps = pd.Series(np.random.randn(len(prng)), prng)
ps
```
``PeriodIndex`` supports addition and subtraction with the same rule as ``Period``.

```python
idx = pd.period_range("2014-07-01 09:00", periods=5, freq="h")
idx
idx + pd.offsets.Hour(2)

idx = pd.period_range("2014-07", periods=5, freq="M")
idx
idx + pd.offsets.MonthEnd(3)
```
``PeriodIndex`` has its own dtype named ``period``, refer to `Period Dtypes <timeseries.period_dtype>`.



### Period dtypes
``PeriodIndex`` has a custom ``period`` dtype. This is a pandas extension
dtype similar to the `timezone aware dtype <timeseries.timezone_series>` (``datetime64[ns, tz]``).

The ``period`` dtype holds the ``freq`` attribute and is represented with
``period[freq]`` like ``period[D]`` or ``period[M]``, using `frequency strings <timeseries.period_aliases>`.

```python
pi = pd.period_range("2016-01-01", periods=3, freq="M")
pi
pi.dtype
```
The ``period`` dtype can be used in ``.astype(...)``. It allows one to change the
``freq`` of a ``PeriodIndex`` like ``.asfreq()`` and convert a
``DatetimeIndex`` to ``PeriodIndex`` like ``to_period()``:

```python
# change monthly freq to daily freq
pi.astype("period[D]")

# convert to DatetimeIndex
pi.astype("datetime64[ns]")

# convert to PeriodIndex
dti = pd.date_range("2011-01-01", freq="ME", periods=3)
dti
dti.astype("period[M]")
```
### PeriodIndex partial string indexing
PeriodIndex now supports partial string slicing with non-monotonic indexes.

You can pass in dates and strings to ``Series`` and ``DataFrame`` with ``PeriodIndex``, in the same manner as ``DatetimeIndex``. For details, refer to `DatetimeIndex Partial String Indexing <timeseries.partialindexing>`.

```python
ps["2011-01"]

ps[datetime.datetime(2011, 12, 25):]

ps["10/31/2011":"12/31/2011"]
```
Passing a string representing a lower frequency than ``PeriodIndex`` returns partial sliced data.

```python
ps["2011"]

dfp = pd.DataFrame(
    np.random.randn(600, 1),
    columns=["A"],
    index=pd.period_range("2013-01-01 9:00", periods=600, freq="min"),
)
dfp
dfp.loc["2013-01-01 10h"]
```
As with ``DatetimeIndex``, the endpoints will be included in the result. The example below slices data starting from 10:00 to 11:59.

```python
dfp["2013-01-01 10h":"2013-01-01 11h"]
```
### Frequency conversion and resampling with PeriodIndex
The frequency of ``Period`` and ``PeriodIndex`` can be converted via the ``asfreq``
method. Let's start with the fiscal year 2011, ending in December:

```python
p = pd.Period("2011", freq="Y-DEC")
p
```
We can convert it to a monthly frequency. Using the ``how`` parameter, we can
specify whether to return the starting or ending month:

```python
p.asfreq("M", how="start")

p.asfreq("M", how="end")
```
The shorthands 's' and 'e' are provided for convenience:

```python
p.asfreq("M", "s")
p.asfreq("M", "e")
```
Converting to a "super-period" (e.g., annual frequency is a super-period of
quarterly frequency) automatically returns the super-period that includes the
input period:

```python
p = pd.Period("2011-12", freq="M")

p.asfreq("Y-NOV")
```
Note that since we converted to an annual frequency that ends the year in
November, the monthly period of December 2011 is actually in the 2012 Y-NOV
period.



Period conversions with anchored frequencies are particularly useful for
working with various quarterly data common to economics, business, and other
fields. Many organizations define quarters relative to the month in which their
fiscal year starts and ends. Thus, first quarter of 2011 could start in 2010 or
a few months into 2011. Via anchored frequencies, pandas works for all quarterly
frequencies ``Q-JAN`` through ``Q-DEC``.

``Q-DEC`` define regular calendar quarters:

```python
p = pd.Period("2012Q1", freq="Q-DEC")

p.asfreq("D", "s")

p.asfreq("D", "e")
```
``Q-MAR`` defines fiscal year end in March:

```python
p = pd.Period("2011Q4", freq="Q-MAR")

p.asfreq("D", "s")

p.asfreq("D", "e")
```


## Converting between representations
Timestamped data can be converted to PeriodIndex-ed data using ``to_period``
and vice-versa using ``to_timestamp``:

```python
rng = pd.date_range("1/1/2012", periods=5, freq="ME")

ts = pd.Series(np.random.randn(len(rng)), index=rng)

ts

ps = ts.to_period()

ps

ps.to_timestamp()
```
Remember that 's' and 'e' can be used to return the timestamps at the start or
end of the period:

```python
ps.to_timestamp("D", how="s")
```
Converting between period and timestamp enables some convenient arithmetic
functions to be used. In the following example, we convert a quarterly
frequency with year ending in November to 9am of the end of the month following
the quarter end:

```python
prng = pd.period_range("1990Q1", "2000Q4", freq="Q-NOV")

ts = pd.Series(np.random.randn(len(prng)), prng)

ts.index = (prng.asfreq("M", "e") + 1).asfreq("h", "s") + 9

ts.head()
```


## Representing out-of-bounds spans
If you have data that is outside of the ``Timestamp`` bounds, see `Timestamp limitations <timeseries.timestamp-limits>`,
then you can use a ``PeriodIndex`` and/or ``Series`` of ``Periods`` to do computations.

```python
span = pd.period_range("1215-01-01", "1381-01-01", freq="D")
span
```
To convert from an ``int64`` based YYYYMMDD representation.

```python
s = pd.Series([20121231, 20141130, 99991231])
s

def conv(x):
    return pd.Period(year=x // 10000, month=x // 100 % 100, day=x % 100, freq="D")

s.apply(conv)
s.apply(conv)[2]
```
These can easily be converted to a ``PeriodIndex``:

```python
span = pd.PeriodIndex(s.apply(conv))
span
```


## Time zone handling
pandas provides rich support for working with timestamps in different time
zones using the ``zoneinfo``, ``pytz`` and ``dateutil`` libraries or `datetime.timezone`
objects from the standard library.


### Working with time zones
By default, pandas objects are time zone unaware:

```python
rng = pd.date_range("3/6/2012 00:00", periods=15, freq="D")
rng.tz is None
```
To localize these dates to a time zone (assign a particular time zone to a naive date),
you can use the ``tz_localize`` method or the ``tz`` keyword argument in
`date_range`, `Timestamp`, or `DatetimeIndex`.
You can either pass ``zoneinfo``, ``pytz`` or ``dateutil`` time zone objects or Olson time zone database strings.
Olson time zone strings will return ``pytz`` time zone objects by default.
To return ``dateutil`` time zone objects, append ``dateutil/`` before the string.

* For ``zoneinfo``, a list of available timezones are available from :py`zoneinfo.available_timezones`.
* In ``pytz`` you can find a list of common (and less common) time zones using ``pytz.all_timezones``.
* ``dateutil`` uses the OS time zones so there isn't a fixed list available. For
  common zones, the names are the same as ``pytz`` and ``zoneinfo``.

```python
import dateutil

# pytz
rng_pytz = pd.date_range("3/6/2012 00:00", periods=3, freq="D", tz="Europe/London")
rng_pytz.tz

# dateutil
rng_dateutil = pd.date_range("3/6/2012 00:00", periods=3, freq="D")
rng_dateutil = rng_dateutil.tz_localize("dateutil/Europe/London")
rng_dateutil.tz

# dateutil - utc special case
rng_utc = pd.date_range(
    "3/6/2012 00:00",
    periods=3,
    freq="D",
    tz=dateutil.tz.tzutc(),
)
rng_utc.tz
```
```python
# datetime.timezone
rng_utc = pd.date_range(
    "3/6/2012 00:00",
    periods=3,
    freq="D",
    tz=datetime.timezone.utc,
)
rng_utc.tz
```
Note that the ``UTC`` time zone is a special case in ``dateutil`` and should be constructed explicitly
as an instance of ``dateutil.tz.tzutc``. You can also construct other time
zones objects explicitly first.

```python
import pytz

# pytz
tz_pytz = pytz.timezone("Europe/London")
rng_pytz = pd.date_range("3/6/2012 00:00", periods=3, freq="D")
rng_pytz = rng_pytz.tz_localize(tz_pytz)
rng_pytz.tz == tz_pytz

# dateutil
tz_dateutil = dateutil.tz.gettz("Europe/London")
rng_dateutil = pd.date_range("3/6/2012 00:00", periods=3, freq="D", tz=tz_dateutil)
rng_dateutil.tz == tz_dateutil
```
To convert a time zone aware pandas object from one time zone to another,
you can use the ``tz_convert`` method.

```python
rng_pytz.tz_convert("US/Eastern")
```
> **note.capitalize():**
    When using ``pytz`` time zones, `DatetimeIndex` will construct a different
    time zone object than a `Timestamp` for the same time zone input. A `DatetimeIndex`
    can hold a collection of `Timestamp` objects that may have different UTC offsets and cannot be
    succinctly represented by one ``pytz`` time zone instance while one `Timestamp`
    represents one point in time with a specific UTC offset.

    ```python
dti = pd.date_range("2019-01-01", periods=3, freq="D", tz="US/Pacific")
dti.tz
ts = pd.Timestamp("2019-01-01", tz="US/Pacific")
ts.tz
```
> **warning.capitalize():**
        Be wary of conversions between libraries. For some time zones, ``pytz`` and ``dateutil`` have different
        definitions of the zone. This is more of a problem for unusual time zones than for
        'standard' zones like ``US/Eastern``.

> **warning.capitalize():**
    Be aware that a time zone definition across versions of time zone libraries may not
    be considered equal.  This may cause problems when working with stored data that
    is localized using one version and operated on with a different version.
    See `here<io.hdf5-notes>[ for how to handle such a situation.

> **warning.capitalize():**
    For ``pytz`` time zones, it is incorrect to pass a time zone object directly into
    the ``datetime.datetime`` constructor
    (e.g., ``datetime.datetime(2011, 1, 1, tzinfo=pytz.timezone('US/Eastern'))``).
    Instead, the datetime needs to be localized using the ``localize`` method
    on the ``pytz`` time zone object.

> **warning.capitalize():**
    Be aware that for times in the future, correct conversion between time zones
    (and UTC) cannot be guaranteed by any time zone library because a timezone's
    offset from UTC may be changed by the respective government.

> **warning.capitalize():**
    If you are using dates beyond 2038-01-18 with ``pytz``, due to current deficiencies
    in the underlying libraries caused by the year 2038 problem, daylight saving time (DST) adjustments
    to timezone aware dates will not be applied. If and when the underlying libraries are fixed,
    the DST transitions will be applied.

    For example, for two dates that are in British Summer Time (and so would normally be GMT+1), both the following asserts evaluate as true:

    ```python
import pytz

 d_2037 = "2037-03-31T010101"
 d_2038 = "2038-03-31T010101"
 DST = pytz.timezone("Europe/London")
 assert pd.Timestamp(d_2037, tz=DST) != pd.Timestamp(d_2037, tz="GMT")
 assert pd.Timestamp(d_2038, tz=DST) == pd.Timestamp(d_2038, tz="GMT")
```
Under the hood, all timestamps are stored in UTC. Values from a time zone aware
`DatetimeIndex` or `Timestamp` will have their fields (day, hour, minute, etc.)
localized to the time zone. However, timestamps with the same UTC value are
still considered to be equal even if they are in different time zones:

```python
rng_eastern = rng_utc.tz_convert("US/Eastern")
rng_berlin = rng_utc.tz_convert("Europe/Berlin")

rng_eastern[2]
rng_berlin[2]
rng_eastern[2] == rng_berlin[2]
```
Operations between `Series` in different time zones will yield UTC
`Series`, aligning the data on the UTC timestamps:

```python
ts_utc = pd.Series(range(3), pd.date_range("20130101", periods=3, tz="UTC"))
eastern = ts_utc.tz_convert("US/Eastern")
berlin = ts_utc.tz_convert("Europe/Berlin")
result = eastern + berlin
result
result.index
```
To remove time zone information, use ``tz_localize(None)`` or ``tz_convert(None)``.
``tz_localize(None)`` will remove the time zone yielding the local time representation.
``tz_convert(None)`` will remove the time zone after converting to UTC time.

```python
didx = pd.date_range(start="2014-08-01 09:00", freq="h", periods=3, tz="US/Eastern")
didx
didx.tz_localize(None)
didx.tz_convert(None)

# tz_convert(None) is identical to tz_convert('UTC').tz_localize(None)
didx.tz_convert("UTC").tz_localize(None)
```


### Fold
For ambiguous times, pandas supports explicitly specifying the keyword-only fold argument.
Due to daylight saving time, one wall clock time can occur twice when shifting
from summer to winter time; fold describes whether the datetime-like corresponds
to the first (0) or the second time (1) the wall clock hits the ambiguous time.
Fold is supported only for constructing from naive ``datetime.datetime``
(see `datetime documentation](https://docs.python.org/3/library/datetime.html)_ for details) or from [Timestamp`
or for constructing from components (see below). Only ``dateutil`` timezones are supported
(see `dateutil documentation](https://dateutil.readthedocs.io/en/stable/tz.html#dateutil.tz.enfold)_
for [`dateutil`` methods that deal with ambiguous datetimes) as ``pytz``
timezones do not support fold (see `pytz documentation](https://pythonhosted.org/pytz/)_
for details on how ``pytz`` deals with ambiguous datetimes). To localize an ambiguous datetime
with ``pytz``, please use `Timestamp.tz_localize`. In general, we recommend to rely
on `Timestamp.tz_localize` when localizing ambiguous datetimes if you need direct
control over how they are handled.

```python
pd.Timestamp(
    datetime.datetime(2019, 10, 27, 1, 30, 0, 0),
    tz="dateutil/Europe/London",
    fold=0,
)
pd.Timestamp(
    year=2019,
    month=10,
    day=27,
    hour=1,
    minute=30,
    tz="dateutil/Europe/London",
    fold=1,
)
```


### Ambiguous times when localizing
``tz_localize`` may not be able to determine the UTC offset of a timestamp
because daylight savings time (DST) in a local time zone causes some times to occur
twice within one day ("clocks fall back"). The following options are available:

* ``'raise'``: Raises a ``ValueError`` (the default behavior)
* ``'infer'``: Attempt to determine the correct offset based on the monotonicity of the timestamps
* ``'NaT'``: Replaces ambiguous times with ``NaT``
* ``bool``: ``True`` represents a DST time, ``False`` represents non-DST time. An array-like of ``bool`` values is supported for a sequence of times.

```python
rng_hourly = pd.DatetimeIndex(
    ["11/06/2011 00:00", "11/06/2011 01:00", "11/06/2011 01:00", "11/06/2011 02:00"]
)
```
This will fail as there are ambiguous times (``'11/06/2011 01:00'``)

```python
:okexcept:

rng_hourly.tz_localize('US/Eastern')
```
Handle these ambiguous times by specifying the following.

```python
rng_hourly.tz_localize("US/Eastern", ambiguous="infer")
rng_hourly.tz_localize("US/Eastern", ambiguous="NaT")
rng_hourly.tz_localize("US/Eastern", ambiguous=[True, True, False, False])
```


### Nonexistent times when localizing
A DST transition may also shift the local time ahead by 1 hour creating nonexistent
local times ("clocks spring forward"). The behavior of localizing a timeseries with nonexistent times
can be controlled by the ``nonexistent`` argument. The following options are available:

* ``'raise'``: Raises a ``ValueError`` (the default behavior)
* ``'NaT'``: Replaces nonexistent times with ``NaT``
* ``'shift_forward'``: Shifts nonexistent times forward to the closest real time
* ``'shift_backward'``: Shifts nonexistent times backward to the closest real time
* timedelta object: Shifts nonexistent times by the timedelta duration

```python
dti = pd.date_range(start="2015-03-29 02:30:00", periods=3, freq="h")
# 2:30 is a nonexistent time
```
Localization of nonexistent times will raise an error by default.

```python
:okexcept:

dti.tz_localize('Europe/Warsaw')
```
Transform nonexistent times to ``NaT`` or shift the times.

```python
dti
dti.tz_localize("Europe/Warsaw", nonexistent="shift_forward")
dti.tz_localize("Europe/Warsaw", nonexistent="shift_backward")
dti.tz_localize("Europe/Warsaw", nonexistent=pd.Timedelta(1, unit="h"))
dti.tz_localize("Europe/Warsaw", nonexistent="NaT")
```


### Time zone Series operations
A `Series` with time zone **naive** values is
represented with a dtype of ``datetime64[ns]``.

```python
s_naive = pd.Series(pd.date_range("20130101", periods=3))
s_naive
```
A `Series` with a time zone **aware** values is
represented with a dtype of ``datetime64[ns, tz]`` where ``tz`` is the time zone

```python
s_aware = pd.Series(pd.date_range("20130101", periods=3, tz="US/Eastern"))
s_aware
```
Both of these `Series` time zone information
can be manipulated via the ``.dt`` accessor, see `the dt accessor section <basics.dt_accessors>`.

For example, to localize and convert a naive stamp to time zone aware.

```python
s_naive.dt.tz_localize("UTC").dt.tz_convert("US/Eastern")
```
Time zone information can also be manipulated using the ``astype`` method.
This method can convert between different timezone-aware dtypes.

```python
# convert to a new time zone
s_aware.astype("datetime64[ns, CET]")
```
> **note.capitalize():**
   Using `Series.to_numpy` on a ``Series``, returns a NumPy array of the data.
   NumPy does not currently support time zones (even though it is *printing* in the local time zone!),
   therefore an object array of Timestamps is returned for time zone aware data:

   ```python
s_naive.to_numpy()
   s_aware.to_numpy()

By converting to an object array of Timestamps, it preserves the time zone
information. For example, when converting back to a Series:



   pd.Series(s_aware.to_numpy())

However, if you want an actual NumPy ``datetime64[ns]`` array (with the values
converted to UTC) instead of an array of objects, you can specify the
``dtype`` argument:



   s_aware.to_numpy(dtype="datetime64[ns]")
```

---

# Time deltas
Timedeltas are differences in times, expressed in difference units, e.g. days, hours, minutes,
seconds. They can be both positive and negative.

``Timedelta`` is a subclass of ``datetime.timedelta``, and behaves in a similar manner,
but allows compatibility with ``np.timedelta64`` types as well as a host of custom representation,
parsing, and attributes.

## Parsing
You can construct a ``Timedelta`` scalar through various arguments, including `ISO 8601 Duration`_ strings.

```python
import datetime

# strings
pd.Timedelta("1 days")
pd.Timedelta("1 days 00:00:00")
pd.Timedelta("1 days 2 hours")
pd.Timedelta("-1 days 2 min 3us")

# like datetime.timedelta
# note: these MUST be specified as keyword arguments
pd.Timedelta(days=1, seconds=1)

# integers with a unit
pd.Timedelta(1, unit="D")

# from a datetime.timedelta/np.timedelta64
pd.Timedelta(datetime.timedelta(days=1, seconds=1))
pd.Timedelta(np.timedelta64(1, "ms"))

# negative Timedeltas have this string repr
# to be more consistent with datetime.timedelta conventions
pd.Timedelta("-1us")

# a NaT
pd.Timedelta("nan")
pd.Timedelta("nat")

# ISO 8601 Duration strings
pd.Timedelta("P0DT0H1M0S")
pd.Timedelta("P0DT0H0M0.000000123S")
```
`DateOffsets<timeseries.offsets>` (``Hour, Minute, Second, Milli, Micro, Nano``) can also be used in construction.

```python
pd.Timedelta(pd.offsets.Second(2))
```
Further, operations among the scalars yield another scalar ``Timedelta``.

```python
pd.Timedelta(pd.offsets.Hour(48)) + pd.Timedelta(pd.offsets.Second(2)) + pd.Timedelta(
    "00:00:00.000123"
)
```
### to_timedelta
Using the top-level ``pd.to_timedelta``, you can convert a scalar, array, list,
or Series from a recognized timedelta format / value into a ``Timedelta`` type.
It will construct Series if the input is a Series, a scalar if the input is
scalar-like, otherwise it will output a ``TimedeltaIndex``.

You can parse a single string to a Timedelta:

```python
pd.to_timedelta("1 days 06:05:01.00003")
pd.to_timedelta("15.5us")
```
or a list/array of strings:

```python
pd.to_timedelta(["1 days 06:05:01.00003", "15.5us", "nan"])
```
The ``unit`` keyword argument specifies the unit of the Timedelta if the input
is numeric:

```python
pd.to_timedelta(np.arange(5), unit="s")
pd.to_timedelta(np.arange(5), unit="D")
```
> **warning.capitalize():**
    If a string or array of strings is passed as an input then the ``unit`` keyword
    argument will be ignored. If a string without units is passed then the default
    unit of nanoseconds is assumed.



### Timedelta limitations
pandas represents ``Timedeltas`` in nanosecond resolution using
64 bit integers. As such, the 64 bit integer limits determine
the ``Timedelta`` limits.

```python
pd.Timedelta.min
pd.Timedelta.max
```


## Operations
You can operate on Series/DataFrames and construct ``timedelta64[ns]`` Series through
subtraction operations on ``datetime64[ns]`` Series, or ``Timestamps``.

```python
s = pd.Series(pd.date_range("2012-1-1", periods=3, freq="D"))
td = pd.Series([pd.Timedelta(days=i) for i in range(3)])
df = pd.DataFrame({"A": s, "B": td})
df
df["C"] = df["A"] + df["B"]
df
df.dtypes

s - s.max()
s - datetime.datetime(2011, 1, 1, 3, 5)
s + datetime.timedelta(minutes=5)
s + pd.offsets.Minute(5)
s + pd.offsets.Minute(5) + pd.offsets.Milli(5)
```
Operations with scalars from a ``timedelta64[ns]`` series:

```python
y = s - s[0]
y
```
Series of timedeltas with ``NaT`` values are supported:

```python
y = s - s.shift()
y
```
Elements can be set to ``NaT`` using ``np.nan`` analogously to datetimes:

```python
y[1] = np.nan
y
```
Operands can also appear in a reversed order (a singular object operated with a Series):

```python
s.max() - s
datetime.datetime(2011, 1, 1, 3, 5) - s
datetime.timedelta(minutes=5) + s
```
``min, max`` and the corresponding ``idxmin, idxmax`` operations are supported on frames:

```python
A = s - pd.Timestamp("20120101") - pd.Timedelta("00:05:05")
B = s - pd.Series(pd.date_range("2012-1-2", periods=3, freq="D"))

df = pd.DataFrame({"A": A, "B": B})
df

df.min()
df.min(axis=1)

df.idxmin()
df.idxmax()
```
``min, max, idxmin, idxmax`` operations are supported on Series as well. A scalar result will be a ``Timedelta``.

```python
df.min().max()
df.min(axis=1).min()

df.min().idxmax()
df.min(axis=1).idxmin()
```
You can fillna on timedeltas, passing a timedelta to get a particular value.

```python
y.fillna(pd.Timedelta(0))
y.fillna(pd.Timedelta(10, unit="s"))
y.fillna(pd.Timedelta("-1 days, 00:00:05"))
```
You can also negate, multiply and use ``abs`` on ``Timedeltas``:

```python
td1 = pd.Timedelta("-1 days 2 hours 3 seconds")
td1
-1 * td1
-td1
abs(td1)
```


## Reductions
Numeric reduction operation for ``timedelta64[ns]`` will return ``Timedelta`` objects. As usual
``NaT`` are skipped during evaluation.

```python
y2 = pd.Series(
    pd.to_timedelta(["-1 days +00:00:05", "nat", "-1 days +00:00:05", "1 days"])
)
y2
y2.mean()
y2.median()
y2.quantile(0.1)
y2.sum()
```


## Frequency conversion
Timedelta Series and ``TimedeltaIndex``, and ``Timedelta`` can be converted to other frequencies by astyping to a specific timedelta dtype.

```python
december = pd.Series(pd.date_range("20121201", periods=4))
january = pd.Series(pd.date_range("20130101", periods=4))
td = january - december

td[2] += datetime.timedelta(minutes=5, seconds=3)
td[3] = np.nan
td

# to seconds
td.astype("timedelta64[s]")
```
For timedelta64 resolutions other than the supported "s", "ms", "us", "ns",
an alternative is to divide by another timedelta object. Note that division by the NumPy scalar is true division, while astyping is equivalent of floor division.

```python
# to days
td / np.timedelta64(1, "D")
```
Dividing or multiplying a ``timedelta64[ns]`` Series by an integer or integer Series
yields another ``timedelta64[ns]`` dtypes Series.

```python
td * -1
td * pd.Series([1, 2, 3, 4])
```
Rounded division (floor-division) of a ``timedelta64[ns]`` Series by a scalar
``Timedelta`` gives a series of integers.

```python
td // pd.Timedelta(days=3, hours=4)
pd.Timedelta(days=3, hours=4) // td
```


The mod (%) and divmod operations are defined for ``Timedelta`` when operating with another timedelta-like or with a numeric argument.

```python
pd.Timedelta(hours=37) % datetime.timedelta(hours=2)

# divmod against a timedelta-like returns a pair (int, Timedelta)
divmod(datetime.timedelta(hours=2), pd.Timedelta(minutes=11))

# divmod against a numeric returns a pair (Timedelta, Timedelta)
divmod(pd.Timedelta(hours=25), 86400000000000)
```
## Attributes
You can access various components of the ``Timedelta`` or ``TimedeltaIndex`` directly using the attributes ``days,seconds,microseconds,nanoseconds``. These are identical to the values returned by ``datetime.timedelta``, in that, for example, the ``.seconds`` attribute represents the number of seconds >= 0 and < 1 day. These are signed according to whether the ``Timedelta`` is signed.

These operations can also be directly accessed via the ``.dt`` property of the ``Series`` as well.

> **note.capitalize():**
   Note that the attributes are NOT the displayed values of the ``Timedelta``. Use ``.components`` to retrieve the displayed values.

For a ``Series``:

```python
td.dt.days
td.dt.seconds
```
You can access the value of the fields for a scalar ``Timedelta`` directly.

```python
tds = pd.Timedelta("31 days 5 min 3 sec")
tds.days
tds.seconds
(-tds).seconds
```
You can use the ``.components`` property to access a reduced form of the timedelta. This returns a ``DataFrame`` indexed
similarly to the ``Series``. These are the *displayed* values of the ``Timedelta``.

```python
td.dt.components
td.dt.components.seconds
```


You can convert a ``Timedelta`` to an `ISO 8601 Duration`_ string with the
``.isoformat`` method

```python
pd.Timedelta(
    days=6, minutes=50, seconds=3, milliseconds=10, microseconds=10, nanoseconds=12
).isoformat()
```
.. _ISO 8601 Duration: https://en.wikipedia.org/wiki/ISO_8601#Durations



## TimedeltaIndex
To generate an index with time delta, you can use either the `TimedeltaIndex` or
the `timedelta_range` constructor.

Using ``TimedeltaIndex`` you can pass string-like, ``Timedelta``, ``timedelta``,
or ``np.timedelta64`` objects. Passing ``np.nan/pd.NaT/nat`` will represent missing values.

```python
pd.TimedeltaIndex(
    [
        "1 days",
        "1 days, 00:00:05",
        np.timedelta64(2, "D"),
        datetime.timedelta(days=2, seconds=2),
    ]
)
```
The string 'infer' can be passed in order to set the frequency of the index as the
inferred frequency upon creation:

```python
pd.TimedeltaIndex(["0 days", "10 days", "20 days"], freq="infer")
```
### Generating ranges of time deltas
Similar to `date_range`, you can construct regular ranges of a ``TimedeltaIndex``
using `timedelta_range`.  The default frequency for ``timedelta_range`` is
calendar day:

```python
pd.timedelta_range(start="1 days", periods=5)
```
Various combinations of ``start``, ``end``, and ``periods`` can be used with
``timedelta_range``:

```python
pd.timedelta_range(start="1 days", end="5 days")

pd.timedelta_range(end="10 days", periods=4)
```
The ``freq`` parameter can passed a variety of `frequency aliases <timeseries.offset_aliases>`:

```python
pd.timedelta_range(start="1 days", end="2 days", freq="30min")

pd.timedelta_range(start="1 days", periods=5, freq="2D5h")
```
Specifying ``start``, ``end``, and ``periods`` will generate a range of evenly spaced
timedeltas from ``start`` to ``end`` inclusively, with ``periods`` number of elements
in the resulting ``TimedeltaIndex``:

```python
pd.timedelta_range("0 days", "4 days", periods=5)

pd.timedelta_range("0 days", "4 days", periods=10)
```
### Using the TimedeltaIndex
Similarly to other of the datetime-like indices, ``DatetimeIndex`` and ``PeriodIndex``, you can use
``TimedeltaIndex`` as the index of pandas objects.

```python
s = pd.Series(
    np.arange(100),
    index=pd.timedelta_range("1 days", periods=100, freq="h"),
)
s
```
Selections work similarly, with coercion on string-likes and slices:

```python
s["1 day":"2 day"]
s["1 day 01:00:00"]
s[pd.Timedelta("1 day 1h")]
```
Furthermore you can use partial string selection and the range will be inferred:

```python
s["1 day":"1 day 5 hours"]
```
### Operations
Finally, the combination of ``TimedeltaIndex`` with ``DatetimeIndex`` allow certain combination operations that are NaT preserving:

```python
tdi = pd.TimedeltaIndex(["1 days", pd.NaT, "2 days"])
tdi.to_list()
dti = pd.date_range("20130101", periods=3)
dti.to_list()
(dti + tdi).to_list()
(dti - tdi).to_list()
```
### Conversions
Similarly to frequency conversion on a ``Series`` above, you can convert these indices to yield another Index.

```python
tdi / np.timedelta64(1, "s")
tdi.astype("timedelta64[s]")
```
Scalars type ops work as well. These can potentially return a *different* type of index.

```python
# adding or timedelta and date -> datelike
tdi + pd.Timestamp("20130101")

# subtraction of a date and a timedelta -> datelike
# note that trying to subtract a date from a Timedelta will raise an exception
(pd.Timestamp("20130101") - tdi).to_list()

# timedelta + timedelta -> timedelta
tdi + pd.Timedelta("10 days")

# division can result in a Timedelta if the divisor is an integer
tdi / 2

# or a float64 Index if the divisor is a Timedelta
tdi / tdi[0]
```


## Resampling
Similar to `timeseries resampling <timeseries.resampling>`, we can resample with a ``TimedeltaIndex``.

```python
s.resample("D").mean()
```

---

# Options and settings
## Overview
pandas has an options API to configure and customize global behavior related to
[DataFrame` display, data behavior and more.

Options have a full "dotted-style", case-insensitive name (e.g. ``display.max_rows``).
You can get/set options directly as attributes of the top-level ``options`` attribute:

```python
import pandas as pd

pd.options.display.max_rows
pd.options.display.max_rows = 999
pd.options.display.max_rows
```
The API is composed of 5 relevant functions, available directly from the ``pandas``
namespace:

* `~pandas.get_option` / `~pandas.set_option` - get/set the value of a single option.
* `~pandas.reset_option` - reset one or more options to their default value.
* `~pandas.describe_option` - print the descriptions of one or more options.
* `~pandas.option_context` - execute a codeblock with a set of options
  that revert to prior settings after execution.

> **note.capitalize():**
   Developers can check out `pandas/core/config_init.py](https://github.com/pandas-dev/pandas/blob/main/pandas/core/config_init.py) for more information.

All of the functions above accept a regexp pattern (``re.search`` style) as an argument,
to match an unambiguous substring:

```python
pd.get_option("display.chop_threshold")
pd.set_option("display.chop_threshold", 2)
pd.get_option("display.chop_threshold")
pd.set_option("chop", 4)
pd.get_option("display.chop_threshold")
```
The following will **not work** because it matches multiple option names, e.g.
``display.max_colwidth``, ``display.max_rows``, ``display.max_columns``:

```python
:okexcept:

pd.get_option("max")
```
> **warning.capitalize():**
    Using this form of shorthand may cause your code to break if new options with similar names are added in future versions.


```python
:suppress:
:okwarning:

pd.reset_option("all")
```


## Available options
You can get a list of available options and their descriptions with `~pandas.describe_option`. When called
with no argument `~pandas.describe_option` will print out the descriptions for all available options.

```python
pd.describe_option()
```
## Getting and setting options
As described above, `~pandas.get_option` and `~pandas.set_option`
are available from the pandas namespace.  To change an option, call
``set_option('option regex', new_value)``.

```python
pd.get_option("mode.sim_interactive")
pd.set_option("mode.sim_interactive", True)
pd.get_option("mode.sim_interactive")
```
> **note.capitalize():**
   The option ``'mode.sim_interactive'`` is mostly used for debugging purposes.

You can use `~pandas.reset_option` to revert to a setting's default value

```python
:suppress:

pd.reset_option("display.max_rows")
```
```python
pd.get_option("display.max_rows")
pd.set_option("display.max_rows", 999)
pd.get_option("display.max_rows")
pd.reset_option("display.max_rows")
pd.get_option("display.max_rows")
```
It's also possible to reset multiple options at once (using a regex):

```python
:okwarning:

pd.reset_option("^display")
```
`~pandas.option_context` context manager has been exposed through
the top-level API, allowing you to execute code with given option values. Option values
are restored automatically when you exit the ``with`` block:

```python
with pd.option_context("display.max_rows", 10, "display.max_columns", 5):
    print(pd.get_option("display.max_rows"))
    print(pd.get_option("display.max_columns"))
print(pd.get_option("display.max_rows"))
print(pd.get_option("display.max_columns"))
```
## Setting startup options in Python/IPython environment
Using startup scripts for the Python/IPython environment to import pandas and set options makes working with pandas more efficient.
To do this, create a ``.py`` or ``.ipy`` script in the startup directory of the desired profile.
An example where the startup folder is in a default IPython profile can be found at:



  $IPYTHONDIR/profile_default/startup

More information can be found in the `IPython documentation
<https://ipython.org/ipython-doc/stable/interactive/tutorial.html#startup-files>`__.  An example startup script for pandas is displayed below:

```python
import pandas as pd

pd.set_option("display.max_rows", 999)
pd.set_option("display.precision", 5)
```


## Frequently used options
The following is a demonstrates the more frequently used display options.

``display.max_rows`` and ``display.max_columns`` sets the maximum number
of rows and columns displayed when a frame is pretty-printed. Truncated
lines are replaced by an ellipsis.

```python
df = pd.DataFrame(np.random.randn(7, 2))
pd.set_option("display.max_rows", 7)
df
pd.set_option("display.max_rows", 5)
df
pd.reset_option("display.max_rows")
```
Once the ``display.max_rows`` is exceeded, the ``display.min_rows`` options
determines how many rows are shown in the truncated repr.

```python
pd.set_option("display.max_rows", 8)
pd.set_option("display.min_rows", 4)
# below max_rows -> all rows shown
df = pd.DataFrame(np.random.randn(7, 2))
df
# above max_rows -> only min_rows (4) rows shown
df = pd.DataFrame(np.random.randn(9, 2))
df
pd.reset_option("display.max_rows")
pd.reset_option("display.min_rows")
```
``display.expand_frame_repr`` allows for the representation of a
`DataFrame` to stretch across pages, wrapped over the all the columns.

```python
df = pd.DataFrame(np.random.randn(5, 10))
pd.set_option("expand_frame_repr", True)
df
pd.set_option("expand_frame_repr", False)
df
pd.reset_option("expand_frame_repr")
```
``display.large_repr`` displays a `DataFrame` that exceed
``max_columns`` or ``max_rows`` as a truncated frame or summary.

```python
df = pd.DataFrame(np.random.randn(10, 10))
pd.set_option("display.max_rows", 5)
pd.set_option("large_repr", "truncate")
df
pd.set_option("large_repr", "info")
df
pd.reset_option("large_repr")
pd.reset_option("display.max_rows")
```
``display.max_colwidth`` sets the maximum width of columns.  Cells
of this length or longer will be truncated with an ellipsis.

```python
df = pd.DataFrame(
    np.array(
        [
            ["foo", "bar", "bim", "uncomfortably long string"],
            ["horse", "cow", "banana", "apple"],
        ]
    )
)
pd.set_option("max_colwidth", 40)
df
pd.set_option("max_colwidth", 6)
df
pd.reset_option("max_colwidth")
```
``display.max_info_columns`` sets a threshold for the number of columns
displayed when calling `~pandas.DataFrame.info`.

```python
df = pd.DataFrame(np.random.randn(10, 10))
pd.set_option("max_info_columns", 11)
df.info()
pd.set_option("max_info_columns", 5)
df.info()
pd.reset_option("max_info_columns")
```
``display.max_info_rows``: `~pandas.DataFrame.info` will usually show null-counts for each column.
For a large `DataFrame`, this can be quite slow. ``max_info_rows`` and ``max_info_cols``
limit this null check to the specified rows and columns respectively. The `~pandas.DataFrame.info`
keyword argument ``show_counts=True`` will override this.

```python
df = pd.DataFrame(np.random.choice([0, 1, np.nan], size=(10, 10)))
df
pd.set_option("max_info_rows", 11)
df.info()
pd.set_option("max_info_rows", 5)
df.info()
pd.reset_option("max_info_rows")
```
``display.precision`` sets the output display precision in terms of decimal places.

```python
df = pd.DataFrame(np.random.randn(5, 5))
pd.set_option("display.precision", 7)
df
pd.set_option("display.precision", 4)
df
```
``display.chop_threshold`` sets the rounding threshold to zero when displaying a
`Series` or `DataFrame`. This setting does not change the
precision at which the number is stored.

```python
df = pd.DataFrame(np.random.randn(6, 6))
pd.set_option("chop_threshold", 0)
df
pd.set_option("chop_threshold", 0.5)
df
pd.reset_option("chop_threshold")
```
``display.colheader_justify`` controls the justification of the headers.
The options are ``'right'``, and ``'left'``.

```python
df = pd.DataFrame(
    np.array([np.random.randn(6), np.random.randint(1, 9, 6) * 0.1, np.zeros(6)]).T,
    columns=["A", "B", "C"],
    dtype="float",
)
pd.set_option("colheader_justify", "right")
df
pd.set_option("colheader_justify", "left")
df
pd.reset_option("colheader_justify")
```


## Number formatting
pandas also allows you to set how numbers are displayed in the console.
This option is not set through the ``set_options`` API.

Use the ``set_eng_float_format`` function
to alter the floating-point formatting of pandas objects to produce a particular
format.

```python
import numpy as np

pd.set_eng_float_format(accuracy=3, use_eng_prefix=True)
s = pd.Series(np.random.randn(5), index=["a", "b", "c", "d", "e"])
s / 1.0e3
s / 1.0e6
```
```python
:suppress:
:okwarning:

pd.reset_option("^display")
```
Use `~pandas.DataFrame.round` to specifically control rounding of an individual `DataFrame`



## Unicode formatting
> **warning.capitalize():**
   Enabling this option will affect the performance for printing of DataFrame and Series (about 2 times slower).
   Use only when it is actually required.

Some East Asian countries use Unicode characters whose width corresponds to two Latin characters.
If a DataFrame or Series contains these characters, the default output mode may not align them properly.

```python
df = pd.DataFrame({"国籍": ["UK", "日本"], "名前": ["Alice", "しのぶ"]})
df
```
Enabling ``display.unicode.east_asian_width`` allows pandas to check each character's "East Asian Width" property.
These characters can be aligned properly by setting this option to ``True``. However, this will result in longer render
times than the standard ``len`` function.

```python
pd.set_option("display.unicode.east_asian_width", True)
df
```
In addition, Unicode characters whose width is "ambiguous" can either be 1 or 2 characters wide depending on the
terminal setting or encoding. The option ``display.unicode.ambiguous_as_wide`` can be used to handle the ambiguity.

By default, an "ambiguous" character's width, such as "¡" (inverted exclamation) in the example below, is taken to be 1.

```python
df = pd.DataFrame({"a": ["xxx", "¡¡"], "b": ["yyy", "¡¡"]})
df
```
Enabling ``display.unicode.ambiguous_as_wide`` makes pandas interpret these characters' widths to be 2.
(Note that this option will only be effective when ``display.unicode.east_asian_width`` is enabled.)

However, setting this option incorrectly for your terminal will cause these characters to be aligned incorrectly:

```python
pd.set_option("display.unicode.ambiguous_as_wide", True)
df
```
```python
:suppress:

pd.set_option("display.unicode.east_asian_width", False)
pd.set_option("display.unicode.ambiguous_as_wide", False)
```


## Table schema display
`DataFrame` and `Series` will publish a Table Schema representation
by default. This can be enabled globally with the
``display.html.table_schema`` option:

```python
pd.set_option("display.html.table_schema", True)
```
Only ``'display.max_rows'`` are serialized and published.


```python
:suppress:

pd.reset_option("display.html.table_schema")
```

---

# Enhancing performance
In this part of the tutorial, we will investigate how to speed up certain
functions operating on pandas `DataFrame` using Cython, Numba and `pandas.eval`.
Generally, using Cython and Numba can offer a larger speedup than using `pandas.eval`
but will require a lot more code.

> **note.capitalize():**
   In addition to following the steps in this tutorial, users interested in enhancing
   performance are highly encouraged to install the
   `recommended dependencies<install.recommended_dependencies>[ for pandas.
   These dependencies are often not installed by default, but will offer speed
   improvements if present.



## Cython (writing C extensions for pandas)
For many use cases writing pandas in pure Python and NumPy is sufficient. In some
computationally heavy applications however, it can be possible to achieve sizable
speed-ups by offloading work to `cython](https://cython.org/)_.

This tutorial assumes you have refactored as much as possible in Python, for example
by trying to remove for-loops and making use of NumPy vectorization. It's always worth
optimising in Python first.

This tutorial walks through a "typical" process of cythonizing a slow computation.
We use an [example from the Cython documentation](https://docs.cython.org/en/latest/src/quickstart/cythonize.html)_
but in the context of pandas. Our final cythonized solution is around 100 times
faster than the pure Python solution.



### Pure Python
We have a [DataFrame` to which we want to apply a function row-wise.

```python
df = pd.DataFrame(
    {
        "a": np.random.randn(1000),
        "b": np.random.randn(1000),
        "N": np.random.randint(100, 1000, (1000), dtype="int64"),
        "x": "x",
    }
)
df
```
Here's the function in pure Python:

```python
def f(x):
    return x * (x - 1)


def integrate_f(a, b, N):
    s = 0
    dx = (b - a) / N
    for i in range(N):
        s += f(a + i * dx)
    return s * dx
```
We achieve our result by using `DataFrame.apply` (row-wise):

```python
%timeit df.apply(lambda x: integrate_f(x["a"], x["b"], x["N"]), axis=1)
```
Let's take a look and see where the time is spent during this operation
using the `prun ipython magic function](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-prun)_:

[``python
# most time consuming 4 calls
%prun -l 4 df.apply(lambda x: integrate_f(x['a'], x['b'], x['N']), axis=1)
```
By far the majority of time is spend inside either ``integrate_f`` or ``f``,
hence we'll concentrate our efforts cythonizing these two functions.



### Plain Cython
First we're going to need to import the Cython magic function to IPython:

```python
:okwarning:

%load_ext Cython
```
Now, let's simply copy our functions over to Cython:



   In [2]: %%cython
      ...: def f_plain(x):
      ...:     return x * (x - 1)
      ...: def integrate_f_plain(a, b, N):
      ...:     s = 0
      ...:     dx = (b - a) / N
      ...:     for i in range(N):
      ...:         s += f_plain(a + i * dx)
      ...:     return s * dx
      ...:


```python
%timeit df.apply(lambda x: integrate_f_plain(x["a"], x["b"], x["N"]), axis=1)
```
This has improved the performance compared to the pure Python approach by one-third.



### Declaring C types
We can annotate the function variables and return types as well as use ``cdef``
and ``cpdef`` to improve performance:



   In [3]: %%cython
      ...: cdef double f_typed(double x) except? -2:
      ...:     return x * (x - 1)
      ...: cpdef double integrate_f_typed(double a, double b, int N):
      ...:     cdef int i
      ...:     cdef double s, dx
      ...:     s = 0
      ...:     dx = (b - a) / N
      ...:     for i in range(N):
      ...:         s += f_typed(a + i * dx)
      ...:     return s * dx
      ...:

```python
%timeit df.apply(lambda x: integrate_f_typed(x["a"], x["b"], x["N"]), axis=1)
```
Annotating the functions with C types yields an over ten times performance improvement compared to
the original Python implementation.



### Using ndarray
When re-profiling, time is spent creating a `Series` from each row, and calling ``__getitem__`` from both
the index and the series (three times for each row). These Python function calls are expensive and
can be improved by passing an ``np.ndarray``.

```python
%prun -l 4 df.apply(lambda x: integrate_f_typed(x['a'], x['b'], x['N']), axis=1)
```


   In [4]: %%cython
      ...: cimport numpy as np
      ...: import numpy as np
      ...: np.import_array()
      ...: cdef double f_typed(double x) except? -2:
      ...:     return x * (x - 1)
      ...: cpdef double integrate_f_typed(double a, double b, int N):
      ...:     cdef int i
      ...:     cdef double s, dx
      ...:     s = 0
      ...:     dx = (b - a) / N
      ...:     for i in range(N):
      ...:         s += f_typed(a + i * dx)
      ...:     return s * dx
      ...: cpdef np.ndarray[double] apply_integrate_f(np.ndarray col_a, np.ndarray col_b,
      ...:                                            np.ndarray col_N):
      ...:     assert (col_a.dtype == np.float64
      ...:             and col_b.dtype == np.float64 and col_N.dtype == np.dtype(int))
      ...:     cdef Py_ssize_t i, n = len(col_N)
      ...:     assert (len(col_a) == len(col_b) == n)
      ...:     cdef np.ndarray[double] res = np.empty(n)
      ...:     for i in range(len(col_a)):
      ...:         res[i] = integrate_f_typed(col_a[i], col_b[i], col_N[i])
      ...:     return res
      ...:


This implementation creates an array of zeros and inserts the result
of ``integrate_f_typed`` applied over each row. Looping over an ``ndarray`` is faster
in Cython than looping over a `Series` object.

Since ``apply_integrate_f`` is typed to accept an ``np.ndarray``, `Series.to_numpy`
calls are needed to utilize this function.

```python
%timeit apply_integrate_f(df['a'].to_numpy(), df['b'].to_numpy(), df['N'].to_numpy())
```
Performance has improved from the prior implementation by almost ten times.



### Disabling compiler directives
The majority of the time is now spent in ``apply_integrate_f``. Disabling Cython's ``boundscheck``
and ``wraparound`` checks can yield more performance.

```python
%prun -l 4 apply_integrate_f(df['a'].to_numpy(), df['b'].to_numpy(), df['N'].to_numpy())
```


   In [5]: %%cython
      ...: cimport cython
      ...: cimport numpy as np
      ...: import numpy as np
      ...: np.import_array()
      ...: cdef np.float64_t f_typed(np.float64_t x) except? -2:
      ...:     return x * (x - 1)
      ...: cpdef np.float64_t integrate_f_typed(np.float64_t a, np.float64_t b, np.int64_t N):
      ...:     cdef np.int64_t i
      ...:     cdef np.float64_t s = 0.0, dx
      ...:     dx = (b - a) / N
      ...:     for i in range(N):
      ...:         s += f_typed(a + i * dx)
      ...:     return s * dx
      ...: @cython.boundscheck(False)
      ...: @cython.wraparound(False)
      ...: cpdef np.ndarray[np.float64_t] apply_integrate_f_wrap(
      ...:     np.ndarray[np.float64_t] col_a,
      ...:     np.ndarray[np.float64_t] col_b,
      ...:     np.ndarray[np.int64_t] col_N
      ...: ):
      ...:     cdef np.int64_t i, n = len(col_N)
      ...:     assert len(col_a) == len(col_b) == n
      ...:     cdef np.ndarray[np.float64_t] res = np.empty(n, dtype=np.float64)
      ...:     for i in range(n):
      ...:         res[i] = integrate_f_typed(col_a[i], col_b[i], col_N[i])
      ...:     return res
      ...:

```python
%timeit apply_integrate_f_wrap(df['a'].to_numpy(), df['b'].to_numpy(), df['N'].to_numpy())
```
However, a loop indexer ``i`` accessing an invalid location in an array would cause a segfault because memory access isn't checked.
For more about ``boundscheck`` and ``wraparound``, see the Cython docs on
`compiler directives](https://cython.readthedocs.io/en/latest/src/userguide/source_files_and_compilation.html#compiler-directives)_.



## Numba (JIT compilation)
An alternative to statically compiling Cython code is to use a dynamic just-in-time (JIT) compiler with [Numba](https://numba.pydata.org/)_.

Numba allows you to write a pure Python function which can be JIT compiled to native machine instructions, similar in performance to C, C++ and Fortran,
by decorating your function with [`@jit``.

Numba works by generating optimized machine code using the LLVM compiler infrastructure at import time, runtime, or statically (using the included pycc tool).
Numba supports compilation of Python to run on either CPU or GPU hardware and is designed to integrate with the Python scientific software stack.

> **note.capitalize():**
    The ``@jit`` compilation will add overhead to the runtime of the function, so performance benefits may not be realized especially when using small data sets.
    Consider `caching](https://numba.readthedocs.io/en/stable/developer/caching.html)_ your function to avoid compilation overhead each time your function is run.

Numba can be used in 2 ways with pandas:

#. Specify the [`engine="numba"`` keyword in select pandas methods
#. Define your own Python function decorated with ``@jit`` and pass the underlying NumPy array of `Series` or `DataFrame` (using `Series.to_numpy`) into the function

### pandas Numba Engine
If Numba is installed, one can specify ``engine="numba"`` in select pandas methods to execute the method using Numba.
Methods that support ``engine="numba"`` will also have an ``engine_kwargs`` keyword that accepts a dictionary that allows one to specify
``"nogil"``, ``"nopython"`` and ``"parallel"`` keys with boolean values to pass into the ``@jit`` decorator.
If ``engine_kwargs`` is not specified, it defaults to ``{"nogil": False, "nopython": True, "parallel": False}`` unless otherwise specified.

> **note.capitalize():**
   In terms of performance, **the first time a function is run using the Numba engine will be slow**
   as Numba will have some function compilation overhead. However, the JIT compiled functions are cached,
   and subsequent calls will be fast. In general, the Numba engine is performant with
   a larger amount of data points (e.g. 1+ million).

   .. code-block:: ipython

      In [1]: data = pd.Series(range(1_000_000))  # noqa: E225

      In [2]: roll = data.rolling(10)

      In [3]: def f(x):
         ...:     return np.sum(x) + 5
      # Run the first time, compilation time will affect performance
      In [4]: %timeit -r 1 -n 1 roll.apply(f, engine='numba', raw=True)
      1.23 s ± 0 ns per loop (mean ± std. dev. of 1 run, 1 loop each)
      # Function is cached and performance will improve
      In [5]: %timeit roll.apply(f, engine='numba', raw=True)
      188 ms ± 1.93 ms per loop (mean ± std. dev. of 7 runs, 10 loops each)

      In [6]: %timeit roll.apply(f, engine='cython', raw=True)
      3.92 s ± 59 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)

If your compute hardware contains multiple CPUs, the largest performance gain can be realized by setting ``parallel`` to ``True``
to leverage more than 1 CPU. Internally, pandas leverages numba to parallelize computations over the columns of a `DataFrame`;
therefore, this performance benefit is only beneficial for a `DataFrame` with a large number of columns.



   In [1]: import numba

   In [2]: numba.set_num_threads(1)

   In [3]: df = pd.DataFrame(np.random.randn(10_000, 100))

   In [4]: roll = df.rolling(100)

   In [5]: %timeit roll.mean(engine="numba", engine_kwargs={"parallel": True})
   347 ms ± 26 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)

   In [6]: numba.set_num_threads(2)

   In [7]: %timeit roll.mean(engine="numba", engine_kwargs={"parallel": True})
   201 ms ± 2.97 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)

### Custom Function Examples
A custom Python function decorated with ``@jit`` can be used with pandas objects by passing their NumPy array
representations with `Series.to_numpy`.

```python
import numba


@numba.jit
def f_plain(x):
    return x * (x - 1)


@numba.jit
def integrate_f_numba(a, b, N):
    s = 0
    dx = (b - a) / N
    for i in range(N):
        s += f_plain(a + i * dx)
    return s * dx


@numba.jit
def apply_integrate_f_numba(col_a, col_b, col_N):
    n = len(col_N)
    result = np.empty(n, dtype="float64")
    assert len(col_a) == len(col_b) == n
    for i in range(n):
        result[i] = integrate_f_numba(col_a[i], col_b[i], col_N[i])
    return result


def compute_numba(df):
    result = apply_integrate_f_numba(
        df["a"].to_numpy(), df["b"].to_numpy(), df["N"].to_numpy()
    )
    return pd.Series(result, index=df.index, name="result")
```


   In [4]: %timeit compute_numba(df)
   1000 loops, best of 3: 798 us per loop

In this example, using Numba was faster than Cython.

Numba can also be used to write vectorized functions that do not require the user to explicitly
loop over the observations of a vector; a vectorized function will be applied to each row automatically.
Consider the following example of doubling each observation:

```python
import numba


def double_every_value_nonumba(x):
    return x * 2


@numba.vectorize
def double_every_value_withnumba(x):  # noqa E501
    return x * 2
```


   # Custom function without numba
   In [5]: %timeit df["col1_doubled"] = df["a"].apply(double_every_value_nonumba)  # noqa E501
   1000 loops, best of 3: 797 us per loop

   # Standard implementation (faster than a custom function)
   In [6]: %timeit df["col1_doubled"] = df["a"] * 2
   1000 loops, best of 3: 233 us per loop

   # Custom function with numba
   In [7]: %timeit df["col1_doubled"] = double_every_value_withnumba(df["a"].to_numpy())
   1000 loops, best of 3: 145 us per loop

### Caveats
Numba is best at accelerating functions that apply numerical functions to NumPy
arrays. If you try to ``@jit`` a function that contains unsupported `Python](https://numba.readthedocs.io/en/stable/reference/pysupported.html)_
or [NumPy](https://numba.readthedocs.io/en/stable/reference/numpysupported.html)_
code, compilation will revert [object mode](https://numba.readthedocs.io/en/stable/glossary.html#term-object-mode)_ which
will mostly likely not speed up your function. If you would
prefer that Numba throw an error if it cannot compile a function in a way that
speeds up your code, pass Numba the argument
``nopython=True`` (e.g.  ``@jit(nopython=True)``). For more on
troubleshooting Numba modes, see the `Numba troubleshooting page
<https://numba.readthedocs.io/en/stable/user/troubleshoot.html>[__.

Using ``parallel=True`` (e.g. ``@jit(parallel=True)``) may result in a ``SIGABRT`` if the threading layer leads to unsafe
behavior. You can first `specify a safe threading layer](https://numba.readthedocs.io/en/stable/user/threading-layer.html#selecting-a-threading-layer-for-safe-parallel-execution)_
before running a JIT function with [`parallel=True``.

Generally if the you encounter a segfault (``SIGSEGV``) while using Numba, please report the issue
to the `Numba issue tracker.](https://github.com/numba/numba/issues/new/choose)_



## Expression evaluation via `~pandas.eval`
The top-level function `pandas.eval` implements performant expression evaluation of
`~pandas.Series` and `~pandas.DataFrame`. Expression evaluation allows operations
to be expressed as strings and can potentially provide a performance improvement
by evaluate arithmetic and boolean expression all at once for large `~pandas.DataFrame`.

> **note.capitalize():**
   You should not use `~pandas.eval` for simple
   expressions or for expressions involving small DataFrames. In fact,
   `~pandas.eval` is many orders of magnitude slower for
   smaller expressions or objects than plain Python. A good rule of thumb is
   to only use `~pandas.eval` when you have a
   `~pandas.core.frame.DataFrame` with more than 10,000 rows.

### Supported syntax
These operations are supported by `pandas.eval`:

* Arithmetic operations except for the left shift (``<<[`) and right shift
  (``>>``) operators, e.g., ``df + 2 * pi / s ** 4 % 42 - the_golden_ratio``
* Comparison operations, including chained comparisons, e.g., ``2]( df < df2``
* Boolean operations, e.g., ``df < df2 and df3 < df4 or not df_bool``
* ``list`` and ``tuple`` literals, e.g., ``[1, 2]`` or ``(1, 2)``
* Attribute access, e.g., ``df.a``
* Subscript expressions, e.g., ``df[0]``
* Simple variable evaluation, e.g., ``pd.eval("df")`` (this is not very useful)
* Math functions: ``sin``, ``cos``, ``exp``, ``log``, ``expm1``, ``log1p``,
  ``sqrt``, ``sinh``, ``cosh``, ``tanh``, ``arcsin``, ``arccos``, ``arctan``, ``arccosh``,
  ``arcsinh``, ``arctanh``, ``abs``, ``arctan2`` and ``log10``.

The following Python syntax is **not** allowed:

* Expressions

    * Function calls other than math functions.
    * ``is``/``is not`` operations
    * ``if`` expressions
    * ``lambda`` expressions
    * ``list``/``set``/``dict`` comprehensions
    * Literal ``dict`` and ``set`` expressions
    * ``yield`` expressions
    * Generator expressions
    * Boolean expressions consisting of only scalar values

* Statements

    * Neither `simple <https://docs.python.org/3/reference/simple_stmts.html)_
      or [compound](https://docs.python.org/3/reference/compound_stmts.html)_
      statements are allowed. This includes ``for``, ``while``, and
      ``if``.

### Local variables
You must *explicitly reference* any local variable that you want to use in an
expression by placing the ``@`` character in front of the name. This mechanism is
the same for both `DataFrame.query` and `DataFrame.eval`. For example,

```python
df = pd.DataFrame(np.random.randn(5, 2), columns=list("ab"))
newcol = np.random.randn(len(df))
df.eval("b + @newcol")
df.query("b < @newcol")
```
If you don't prefix the local variable with ``@``, pandas will raise an
exception telling you the variable is undefined.

When using `DataFrame.eval` and `DataFrame.query`, this allows you
to have a local variable and a `~pandas.DataFrame` column with the same
name in an expression.


```python
a = np.random.randn()
df.query("@a < a")
df.loc[a < df["a"]]  # same as the previous expression
```
> **warning.capitalize():**
   `pandas.eval` will raise an exception if you cannot use the ``@`` prefix because it
   isn't defined in that context.

   ```python
:okexcept:

   a, b = 1, 2
   pd.eval("@a + b")

In this case, you should simply refer to the variables like you would in
standard Python.



   pd.eval("a + b")
```
### `pandas.eval` parsers
There are two different expression syntax parsers.

The default ``'pandas'`` parser allows a more intuitive syntax for expressing
query-like operations (comparisons, conjunctions and disjunctions). In
particular, the precedence of the ``&`` and ``|`` operators is made equal to
the precedence of the corresponding boolean operations ``and`` and ``or``.

For example, the above conjunction can be written without parentheses.
Alternatively, you can use the ``'python'`` parser to enforce strict Python
semantics.

```python
nrows, ncols = 20000, 100
df1, df2, df3, df4 = [pd.DataFrame(np.random.randn(nrows, ncols)) for _ in range(4)]

expr = "(df1 > 0) & (df2 > 0) & (df3 > 0) & (df4 > 0)"
x = pd.eval(expr, parser="python")
expr_no_parens = "df1 > 0 & df2 > 0 & df3 > 0 & df4 > 0"
y = pd.eval(expr_no_parens, parser="pandas")
np.all(x == y)
```
The same expression can be "anded" together with the word `and` as
well:

```python
expr = "(df1 > 0) & (df2 > 0) & (df3 > 0) & (df4 > 0)"
x = pd.eval(expr, parser="python")
expr_with_ands = "df1 > 0 and df2 > 0 and df3 > 0 and df4 > 0"
y = pd.eval(expr_with_ands, parser="pandas")
np.all(x == y)
```
The `and` and `or` operators here have the same precedence that they would
in Python.


### `pandas.eval` engines
There are two different expression engines.

The ``'numexpr'`` engine is the more performant engine that can yield performance improvements
compared to standard Python syntax for large `DataFrame`. This engine requires the
optional dependency ``numexpr`` to be installed.

The ``'python'`` engine is generally *not* useful except for testing
other evaluation engines against it. You will achieve **no** performance
benefits using `~pandas.eval` with ``engine='python'`` and may
incur a performance hit.

```python
%timeit df1 + df2 + df3 + df4
```
```python
%timeit pd.eval("df1 + df2 + df3 + df4", engine="python")
```
### The `DataFrame.eval` method
In addition to the top level `pandas.eval` function you can also
evaluate an expression in the "context" of a `~pandas.DataFrame`.

```python
:suppress:

try:
    del a
except NameError:
    pass

try:
    del b
except NameError:
    pass
```
```python
df = pd.DataFrame(np.random.randn(5, 2), columns=["a", "b"])
df.eval("a + b")
```
Any expression that is a valid `pandas.eval` expression is also a valid
`DataFrame.eval` expression, with the added benefit that you don't have to
prefix the name of the `~pandas.DataFrame` to the column(s) you're
interested in evaluating.

In addition, you can perform assignment of columns within an expression.
This allows for *formulaic evaluation*. The assignment target can be a
new column name or an existing column name, and it must be a valid Python
identifier.

```python
df = pd.DataFrame(dict(a=range(5), b=range(5, 10)))
df = df.eval("c = a + b")
df = df.eval("d = a + b + c")
df = df.eval("a = 1")
df
```
A copy of the `DataFrame` with the
new or modified columns is returned, and the original frame is unchanged.

```python
df
df.eval("e = a - c")
df
```
Multiple column assignments can be performed by using a multi-line string.

```python
df.eval(
    """
c = a + b
d = a + b + c
a = 1""",
)
```
The equivalent in standard Python would be

```python
df = pd.DataFrame(dict(a=range(5), b=range(5, 10)))
df["c"] = df["a"] + df["b"]
df["d"] = df["a"] + df["b"] + df["c"]
df["a"] = 1
df
```
### `~pandas.eval` performance comparison
`pandas.eval` works well with expressions containing large arrays.

```python
nrows, ncols = 20000, 100
df1, df2, df3, df4 = [pd.DataFrame(np.random.randn(nrows, ncols)) for _ in range(4)]
```
`DataFrame` arithmetic:

```python
%timeit df1 + df2 + df3 + df4
```
```python
%timeit pd.eval("df1 + df2 + df3 + df4")
```
`DataFrame` comparison:

```python
%timeit (df1 > 0) & (df2 > 0) & (df3 > 0) & (df4 > 0)
```
```python
%timeit pd.eval("(df1 > 0) & (df2 > 0) & (df3 > 0) & (df4 > 0)")
```
`DataFrame` arithmetic with unaligned axes.

```python
s = pd.Series(np.random.randn(50))
%timeit df1 + df2 + df3 + df4 + s
```
```python
%timeit pd.eval("df1 + df2 + df3 + df4 + s")
```
> **note.capitalize():**
   Operations such as

   ```python
1 and 2  # would parse to 1 & 2, but should evaluate to 2
   3 or 4  # would parse to 3 | 4, but should evaluate to 3
   ~1  # this is okay, but slower when using eval

should be performed in Python. An exception will be raised if you try to
perform any boolean/bitwise operations with scalar operands that are not
of type ``bool`` or ``np.bool_``.
```
Here is a plot showing the running time of
`pandas.eval` as function of the size of the frame involved in the
computation. The two lines are two different engines.

..
    The eval-perf.png figure below was generated with /doc/scripts/eval_performance.py



You will only see the performance benefits of using the ``numexpr`` engine with `pandas.eval` if your `~pandas.DataFrame`
has more than approximately 100,000 rows.

This plot was created using a `DataFrame` with 3 columns each containing
floating point values generated using ``numpy.random.randn()``.

### Expression evaluation limitations with ``numexpr``
Expressions that would result in an object dtype or involve datetime operations
because of ``NaT`` must be evaluated in Python space, but part of an expression
can still be evaluated with ``numexpr``. For example:

```python
df = pd.DataFrame(
    {"strings": np.repeat(list("cba"), 3), "nums": np.repeat(range(3), 3)}
)
df
df.query("strings == 'a' and nums == 1")
```
The numeric part of the comparison (``nums == 1``) will be evaluated by
``numexpr`` and the object part of the comparison (``"strings == 'a'``) will
be evaluated by Python.

---

# Scaling to large datasets
pandas provides data structures for in-memory analytics, which makes using pandas
to analyze datasets that are larger than memory somewhat tricky. Even datasets
that are a sizable fraction of memory become unwieldy, as some pandas operations need
to make intermediate copies.

This document provides a few recommendations for scaling your analysis to larger datasets.
It's a complement to `enhancingperf`, which focuses on speeding up analysis
for datasets that fit in memory.

## Load less data
Suppose our raw dataset on disk has many columns.

```python
:okwarning:

import pandas as pd
import numpy as np

def make_timeseries(start="2000-01-01", end="2000-12-31", freq="1D", seed=None):
    index = pd.date_range(start=start, end=end, freq=freq, name="timestamp")
    n = len(index)
    state = np.random.RandomState(seed)
    columns = {
        "name": state.choice(["Alice", "Bob", "Charlie"], size=n),
        "id": state.poisson(1000, size=n),
        "x": state.rand(n) * 2 - 1,
        "y": state.rand(n) * 2 - 1,
    }
    df = pd.DataFrame(columns, index=index, columns=sorted(columns))
    if df.index[-1] == end:
        df = df.iloc[:-1]
    return df

timeseries = [
    make_timeseries(freq="1min", seed=i).rename(columns=lambda x: f"{x}_{i}")
    for i in range(10)
]
ts_wide = pd.concat(timeseries, axis=1)
ts_wide.head()
ts_wide.to_parquet("timeseries_wide.parquet")
```
To load the columns we want, we have two options.
Option 1 loads in all the data and then filters to what we need.

```python
columns = ["id_0", "name_0", "x_0", "y_0"]

pd.read_parquet("timeseries_wide.parquet")[columns]
```
Option 2 only loads the columns we request.

```python
pd.read_parquet("timeseries_wide.parquet", columns=columns)
```
```python
:suppress:

import os

os.remove("timeseries_wide.parquet")
```
If we were to measure the memory usage of the two calls, we'd see that specifying
``columns`` uses about 1/10th the memory in this case.

With `pandas.read_csv`, you can specify ``usecols`` to limit the columns
read into memory. Not all file formats that can be read by pandas provide an option
to read a subset of columns.

## Use efficient datatypes
The default pandas data types are not the most memory efficient. This is
especially true for text data columns with relatively few unique values (commonly
referred to as "low-cardinality" data). By using more efficient data types, you
can store larger datasets in memory.

```python
:okwarning:

ts = make_timeseries(freq="30s", seed=0)
ts.to_parquet("timeseries.parquet")
ts = pd.read_parquet("timeseries.parquet")
ts
```
```python
:suppress:

os.remove("timeseries.parquet")
```
Now, let's inspect the data types and memory usage to see where we should focus our
attention.

```python
ts.dtypes
```
```python
ts.memory_usage(deep=True)  # memory usage in bytes
```
The ``name`` column is taking up much more memory than any other. It has just a
few unique values, so it's a good candidate for converting to a
`pandas.Categorical`. With a `pandas.Categorical`, we store each unique name once and use
space-efficient integers to know which specific name is used in each row.


```python
ts2 = ts.copy()
ts2["name"] = ts2["name"].astype("category")
ts2.memory_usage(deep=True)
```
We can go a bit further and downcast the numeric columns to their smallest types
using `pandas.to_numeric`.

```python
ts2["id"] = pd.to_numeric(ts2["id"], downcast="unsigned")
ts2[["x", "y"]] = ts2[["x", "y"]].apply(pd.to_numeric, downcast="float")
ts2.dtypes
```
```python
ts2.memory_usage(deep=True)
```
```python
reduction = ts2.memory_usage(deep=True).sum() / ts.memory_usage(deep=True).sum()
print(f"{reduction:0.2f}")
```
In all, we've reduced the in-memory footprint of this dataset to 1/5 of its
original size.

See `categorical` for more on `pandas.Categorical` and `basics.dtypes`
for an overview of all of pandas' dtypes.

## Use chunking
Some workloads can be achieved with chunking by splitting a large problem into a bunch of small problems. For example,
converting an individual CSV file into a Parquet file and repeating that for each file in a directory. As long as each chunk
fits in memory, you can work with datasets that are much larger than memory.

> **note.capitalize():**
   Chunking works well when the operation you're performing requires zero or minimal
   coordination between chunks. For more complicated workflows, you're better off
   `using other libraries <scale.other_libraries>[.

Suppose we have an even larger "logical dataset" on disk that's a directory of parquet
files. Each file in the directory represents a different year of the entire dataset.

```python
:okwarning:

import pathlib

N = 12
starts = [f"20{i:>02d}-01-01" for i in range(N)]
ends = [f"20{i:>02d}-12-13" for i in range(N)]

pathlib.Path("data/timeseries").mkdir(exist_ok=True)

for i, (start, end) in enumerate(zip(starts, ends)):
    ts = make_timeseries(start=start, end=end, freq="1min", seed=i)
    ts.to_parquet(f"data/timeseries/ts-{i:0>2d}.parquet")
```
::

   data
   └── timeseries
       ├── ts-00.parquet
       ├── ts-01.parquet
       ├── ts-02.parquet
       ├── ts-03.parquet
       ├── ts-04.parquet
       ├── ts-05.parquet
       ├── ts-06.parquet
       ├── ts-07.parquet
       ├── ts-08.parquet
       ├── ts-09.parquet
       ├── ts-10.parquet
       └── ts-11.parquet

Now we'll implement an out-of-core `pandas.Series.value_counts`. The peak memory usage of this
workflow is the single largest chunk, plus a small series storing the unique value
counts up to this point. As long as each individual file fits in memory, this will
work for arbitrary-sized datasets.

```python
%%time
files = pathlib.Path("data/timeseries/").glob("ts*.parquet")
counts = pd.Series(dtype=int)
for path in files:
    df = pd.read_parquet(path)
    counts = counts.add(df["name"].value_counts(), fill_value=0)
counts.astype(int)
```
Some readers, like `pandas.read_csv`, offer parameters to control the
``chunksize`` when reading a single file.

Manually chunking is an OK option for workflows that don't
require too sophisticated of operations. Some operations, like `pandas.DataFrame.groupby`, are
much harder to do chunkwise. In these cases, you may be better switching to a
different library that implements these out-of-core algorithms for you.



## Use Other Libraries
There are other libraries which provide similar APIs to pandas and work nicely with pandas DataFrame,
and can give you the ability to scale your large dataset processing and analytics
by parallel runtime, distributed memory, clustering, etc. You can find more information
in `the ecosystem page](https://pandas.pydata.org/community/ecosystem.html#out-of-core).

---

# Sparse data structures
pandas provides data structures for efficiently storing sparse data.
These are not necessarily sparse in the typical "mostly 0". Rather, you can view these
objects as being "compressed" where any data matching a specific value ([`NaN`` / missing value, though any value
can be chosen, including 0) is omitted. The compressed values are not actually stored in the array.

```python
arr = np.random.randn(10)
arr[2:-2] = np.nan
ts = pd.Series(pd.arrays.SparseArray(arr))
ts
```
Notice the dtype, ``Sparse[float64, nan]``. The ``nan`` means that elements in the
array that are ``nan`` aren't actually stored, only the non-``nan`` elements are.
Those non-``nan`` elements have a ``float64`` dtype.

The sparse objects exist for memory efficiency reasons. Suppose you had a
large, mostly NA `DataFrame`:

```python
df = pd.DataFrame(np.random.randn(10000, 4))
df.iloc[:9998] = np.nan
sdf = df.astype(pd.SparseDtype("float", np.nan))
sdf.head()
sdf.dtypes
sdf.sparse.density
```
As you can see, the density (% of values that have not been "compressed") is
extremely low. This sparse object takes up much less memory on disk (pickled)
and in the Python interpreter.

```python
f'dense: {df.memory_usage().sum()} bytes'
f'sparse: {sdf.memory_usage().sum()} bytes'
```
Functionally, their behavior should be nearly
identical to their dense counterparts.



## SparseArray
`arrays.SparseArray` is a `~pandas.api.extensions.ExtensionArray`
for storing an array of sparse values (see `basics.dtypes` for more
on extension arrays). It is a 1-dimensional ndarray-like object storing
only values distinct from the ``fill_value``:

```python
arr = np.random.randn(10)
arr[2:5] = np.nan
arr[7:8] = np.nan
sparr = pd.arrays.SparseArray(arr)
sparr
```
A sparse array can be converted to a regular (dense) ndarray with `numpy.asarray`

```python
np.asarray(sparr)
```


## SparseDtype
The `SparseArray.dtype` property stores two pieces of information

1. The dtype of the non-sparse values
2. The scalar fill value


```python
sparr.dtype
```
A `SparseDtype` may be constructed by passing only a dtype

```python
pd.SparseDtype(np.dtype('datetime64[ns]'))
```
in which case a default fill value will be used (for NumPy dtypes this is often the
"missing" value for that dtype). To override this default an explicit fill value may be
passed instead

```python
pd.SparseDtype(np.dtype('datetime64[ns]'),
               fill_value=pd.Timestamp('2017-01-01'))
```
Finally, the string alias ``'Sparse[dtype]'`` may be used to specify a sparse dtype
in many places

```python
pd.array([1, 0, 0, 2], dtype='Sparse[int]')
```


## Sparse accessor
pandas provides a ``.sparse`` accessor, similar to ``.str`` for string data, ``.cat``
for categorical data, and ``.dt`` for datetime-like data. This namespace provides
attributes and methods that are specific to sparse data.

```python
s = pd.Series([0, 0, 1, 2], dtype="Sparse[int]")
s.sparse.density
s.sparse.fill_value
```
This accessor is available only on data with ``SparseDtype``, and on the `Series`
class itself for creating a Series with sparse data from a scipy COO matrix with.

A ``.sparse`` accessor has been added for `DataFrame` as well.
See `api.frame.sparse` for more.



## Sparse calculation
You can apply NumPy `ufuncs](https://numpy.org/doc/stable/reference/ufuncs.html)
to `arrays.SparseArray` and get a `arrays.SparseArray` as a result.

```python
arr = pd.arrays.SparseArray([1., np.nan, np.nan, -2., np.nan])
np.abs(arr)
```
The *ufunc* is also applied to ``fill_value``. This is needed to get
the correct dense result.

```python
arr = pd.arrays.SparseArray([1., -1, -1, -2., -1], fill_value=-1)
np.abs(arr)
np.abs(arr).to_dense()
```
**Conversion**

To convert data from sparse to dense, use the ``.sparse`` accessors

```python
sdf.sparse.to_dense()
```
From dense to sparse, use `DataFrame.astype` with a `SparseDtype`.

```python
dense = pd.DataFrame({"A": [1, 0, 0, 1]})
dtype = pd.SparseDtype(int, fill_value=0)
dense.astype(dtype)
```


## Interaction with *scipy.sparse*
Use `DataFrame.sparse.from_spmatrix` to create a `DataFrame` with sparse values from a sparse matrix.

```python
from scipy.sparse import csr_matrix

arr = np.random.random(size=(1000, 5))
arr[arr < .9] = 0

sp_arr = csr_matrix(arr)
sp_arr

sdf = pd.DataFrame.sparse.from_spmatrix(sp_arr)
sdf.head()
sdf.dtypes
```
All sparse formats are supported, but matrices that are not in `COOrdinate <scipy.sparse>` format will be converted, copying data as needed.
To convert back to sparse SciPy matrix in COO format, you can use the `DataFrame.sparse.to_coo` method:

```python
sdf.sparse.to_coo()
```
`Series.sparse.to_coo` is implemented for transforming a `Series` with sparse values indexed by a `MultiIndex` to a `scipy.sparse.coo_matrix`.

The method requires a `MultiIndex` with two or more levels.

```python
s = pd.Series([3.0, np.nan, 1.0, 3.0, np.nan, np.nan])
s.index = pd.MultiIndex.from_tuples(
    [
        (1, 2, "a", 0),
        (1, 2, "a", 1),
        (1, 1, "b", 0),
        (1, 1, "b", 1),
        (2, 1, "b", 0),
        (2, 1, "b", 1),
    ],
    names=["A", "B", "C", "D"],
)
ss = s.astype('Sparse')
ss
```
In the example below, we transform the `Series` to a sparse representation of a 2-d array by specifying that the first and second ``MultiIndex`` levels define labels for the rows and the third and fourth levels define labels for the columns. We also specify that the column and row labels should be sorted in the final sparse representation.

```python
A, rows, columns = ss.sparse.to_coo(
    row_levels=["A", "B"], column_levels=["C", "D"], sort_labels=True
)

A
A.todense()
rows
columns
```
Specifying different row and column labels (and not sorting them) yields a different sparse matrix:

```python
A, rows, columns = ss.sparse.to_coo(
    row_levels=["A", "B", "C"], column_levels=["D"], sort_labels=False
)

A
A.todense()
rows
columns
```
A convenience method `Series.sparse.from_coo` is implemented for creating a `Series` with sparse values from a ``scipy.sparse.coo_matrix``.

```python
from scipy import sparse
A = sparse.coo_matrix(([3.0, 1.0, 2.0], ([1, 0, 0], [0, 2, 3])), shape=(3, 4))
A
A.todense()
```
The default behaviour (with ``dense_index=False``) simply returns a `Series` containing
only the non-null entries.

```python
ss = pd.Series.sparse.from_coo(A)
ss
```
Specifying ``dense_index=True`` will result in an index that is the Cartesian product of the
row and columns coordinates of the matrix. Note that this will consume a significant amount of memory
(relative to ``dense_index=False``) if the sparse matrix is large (and sparse) enough.

```python
ss_dense = pd.Series.sparse.from_coo(A, dense_index=True)
ss_dense
```

---

# 
# Migration guide for the new string data type (pandas 3.0)
The upcoming pandas 3.0 release introduces a new, default string data type. This
will most likely cause some work when upgrading to pandas 3.0, and this page
provides an overview of the issues you might run into and gives guidance on how
to address them.

This new dtype is already available in the pandas 2.3 release, and you can
enable it with:

[``python
pd.options.future.infer_string = True
```
This allows you to test your code before the final 3.0 release.

## Background
Historically, pandas has always used the NumPy ``object`` dtype as the default
to store text data. This has two primary drawbacks. First, ``object`` dtype is
not specific to strings: any Python object can be stored in an ``object``-dtype
array, not just strings, and seeing ``object`` as the dtype for a column with
strings is confusing for users. Second, this is not always very efficient (both
performance wise and for memory usage).

Since pandas 1.0, an opt-in string data type has been available, but this has
not yet been made the default, and uses the ``pd.NA`` scalar to represent
missing values.

Pandas 3.0 changes the default dtype for strings to a new string data type,
a variant of the existing optional string data type but using ``NaN`` as the
missing value indicator, to be consistent with the other default data types.

To improve performance, the new string data type will use the ``pyarrow``
package by default, if installed (and otherwise it uses object dtype under the
hood as a fallback).

See `PDEP-14: Dedicated string data type for pandas 3.0](https://pandas.pydata.org/pdeps/0014-string-dtype.html)_
for more background and details.

.. - brief primer on the new dtype

.. - Main characteristics:
..    - inferred by default (Default inference of a string dtype)
..    - only strings (setitem with non string fails)
..    - missing values sentinel is always NaN and uses NaN semantics

.. - Breaking changes:
..    - dtype is no longer object dtype
..    - None gets coerced to NaN
..    - setitem raises an error for non-string data

## Brief introduction to the new default string dtype
By default, pandas will infer this new string dtype instead of object dtype for
string data (when creating pandas objects, such as in constructors or IO
functions).

Being a default dtype means that the string dtype will be used in IO methods or
constructors when the dtype is being inferred and the input is inferred to be
string data:

```python
>>> pd.Series(["a", "b", None])
0      a
1      b
2    NaN
dtype: str
```
It can also be specified explicitly using the ``"str"`` alias:

```python
>>> pd.Series(["a", "b", None], dtype="str")
0      a
1      b
2    NaN
dtype: str
```
Similarly, functions like `read_csv`, `read_parquet`, and others
will now use the new string dtype when reading string data.

In contrast to the current object dtype, the new string dtype will only store
strings. This also means that it will raise an error if you try to store a
non-string value in it (see below for more details).

Missing values with the new string dtype are always represented as ``NaN`` (``np.nan``),
and the missing value behavior is similar to other default dtypes.

This new string dtype should otherwise behave the same as the existing
``object`` dtype users are used to. For example, all string-specific methods
through the ``str`` accessor will work the same:

```python
>>> ser = pd.Series(["a", "b", None], dtype="str")
>>> ser.str.upper()
0    A
1    B
2  NaN
dtype: str
```
> **note.capitalize():**
   The new default string dtype is an instance of the `pandas.StringDtype`
   class. The dtype can be constructed as ``pd.StringDtype(na_value=np.nan)``,
   but for general usage we recommend to use the shorter ``"str"`` alias.

## Overview of behavior differences and how to address them
### The dtype is no longer a numpy "object" dtype
When inferring or reading string data, the data type of the resulting DataFrame
column or Series will silently start being the new ``"str"`` dtype instead of
the numpy ``"object"`` dtype, and this can have some impact on your code.

The new string dtype is a pandas data type ("extension dtype"), and no longer a
numpy ``np.dtype`` instance. Therefore, passing the dtype of a string column to
numpy functions will no longer work (e.g. passing it to a ``dtype=`` argument
of a numpy function, or using ``np.issubdtype`` to check the dtype).

#### Checking the dtype
When checking the dtype, code might currently do something like:

```python
>>> ser = pd.Series(["a", "b", "c"])
>>> ser.dtype == "object"
```
to check for columns with string data (by checking for the dtype being
``"object"``). This will no longer work in pandas 3+, since ``ser.dtype`` will
now be ``"str"`` with the new default string dtype, and the above check will
return ``False``.

To check for columns with string data, you should instead use:

```python
>>> ser.dtype == "str"
```
**How to write compatible code**

For code that should work on both pandas 2.x and 3.x, you can use the
`pandas.api.types.is_string_dtype` function:

```python
>>> pd.api.types.is_string_dtype(ser.dtype)
True
```
This will return ``True`` for both the object dtype and the string dtypes.

#### Hardcoded use of object dtype
If you have code where the dtype is hardcoded in constructors, like

```python
>>> pd.Series(["a", "b", "c"], dtype="object")
```
this will keep using the object dtype. You will want to update this code to
ensure you get the benefits of the new string dtype.

**How to write compatible code?**

First, in many cases it can be sufficient to remove the specific data type, and
let pandas do the inference. But if you want to be specific, you can specify the
``"str"`` dtype:

```python
>>> pd.Series(["a", "b", "c"], dtype="str")
```
This is actually compatible with pandas 2.x as well, since in pandas < 3,
``dtype="str"`` was essentially treated as an alias for object dtype.



   While using ``dtype="str"`` in constructors is compatible with pandas 2.x,
   specifying it as the dtype in `~Series.astype` runs into the issue
   of also stringifying missing values in pandas 2.x. See the section
   `string_migration_guide-astype_str` for more details.


### The missing value sentinel is now always NaN
When using object dtype, multiple possible missing value sentinels are
supported, including ``None`` and ``np.nan``. With the new default string dtype,
the missing value sentinel is always NaN (``np.nan``):

```python
# with object dtype, None is preserved as None and seen as missing
>>> ser = pd.Series(["a", "b", None], dtype="object")
>>> ser
0       a
1       b
2    None
dtype: object
>>> print(ser[2])
None

# with the new string dtype, any missing value like None is coerced to NaN
>>> ser = pd.Series(["a", "b", None], dtype="str")
>>> ser
0      a
1      b
2    NaN
dtype: str
>>> print(ser[2])
nan
```
Generally this should be no problem when relying on missing value behavior in
pandas methods (for example, ``ser.isna()`` will give the same result as before).
But when you relied on the exact value of ``None`` being present, that can
impact your code.

**How to write compatible code?**

When checking for a missing value, instead of checking for the exact value of
``None`` or ``np.nan``, you should use the `pandas.isna` function. This is
the most robust way to check for missing values, as it will work regardless of
the dtype and the exact missing value sentinel:

```python
>>> pd.isna(ser[2])
True
```
One caveat: this function works both on scalars and on array-likes, and in the
latter case it will return an array of bools. When using it in a Boolean context
(for example, ``if pd.isna(..): ..``) be sure to only pass a scalar to it.

### "setitem" operations will now raise an error for non-string data
With the new string dtype, any attempt to set a non-string value in a Series or
DataFrame will raise an error:

```python
>>> ser = pd.Series(["a", "b", None], dtype="str")
>>> ser[1] = 2.5
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
...
TypeError: Invalid value '2.5' for dtype 'str'. Value should be a string or missing value, got 'float' instead.
```
If you relied on the flexible nature of object dtype being able to hold any
Python object, but your initial data was inferred as strings, your code might be
impacted by this change.

**How to write compatible code?**

You can update your code to ensure you only set string values in such columns,
or otherwise you can explicitly ensure the column has object dtype first. This
can be done by specifying the dtype explicitly in the constructor, or by using
the `~pandas.Series.astype` method:

```python
>>> ser = pd.Series(["a", "b", None], dtype="str")
>>> ser = ser.astype("object")
>>> ser[1] = 2.5
```
This ``astype("object")`` call will be redundant when using pandas 2.x, but
this code will work for all versions.

### Invalid unicode input
Python allows to have a built-in ``str`` object that represents invalid unicode
data. And since the ``object`` dtype can hold any Python object, you can have a
pandas Series with such invalid unicode data:

```python
>>> ser = pd.Series(["\u2600", "\ud83d"], dtype=object)
>>> ser
0    ☀
1    \ud83d
dtype: object
```
However, when using the string dtype using ``pyarrow`` under the hood, this can
only store valid unicode data, and otherwise it will raise an error:

```python
>>> ser = pd.Series(["\u2600", "\ud83d"])
---------------------------------------------------------------------------
UnicodeEncodeError                        Traceback (most recent call last)
...
UnicodeEncodeError: 'utf-8' codec can't encode character '\ud83d' in position 0: surrogates not allowed
```
If you want to keep the previous behaviour, you can explicitly specify
``dtype=object`` to keep working with object dtype.

When you have byte data that you want to convert to strings using ``decode()``,
the `~pandas.Series.str.decode` method now has a ``dtype`` parameter to be
able to specify object dtype instead of the default of string dtype for this use
case.

### `Series.values` now returns an `~pandas.api.extensions.ExtensionArray`
With object dtype, using ``.values`` on a Series will return the underlying NumPy array.

```python
>>> ser = pd.Series(["a", "b", np.nan], dtype="object")
>>> type(ser.values)
<class 'numpy.ndarray'>
```
However with the new string dtype, the underlying ExtensionArray is returned instead.

```python
>>> ser = pd.Series(["a", "b", pd.NA], dtype="str")
>>> ser.values
<ArrowStringArray>
['a', 'b', nan]
Length: 3, dtype: str
```
If your code requires a NumPy array, you should use `Series.to_numpy`.

```python
>>> ser = pd.Series(["a", "b", pd.NA], dtype="str")
>>> ser.to_numpy()
['a' 'b' nan]
```
In general, you should always prefer `Series.to_numpy` to get a NumPy array or `Series.array` to get an ExtensionArray over using `Series.values`.

### Notable bug fixes


#### ``astype(str)`` preserving missing values
The stringifying of missing values is a long standing "bug" or misfeature, as
discussed in https://github.com/pandas-dev/pandas/issues/25353, but fixing it
introduces a significant behaviour change.

With pandas < 3, when using ``astype(str)`` or ``astype("str")``, the operation
would convert every element to a string, including the missing values:

```python
# OLD behavior in pandas < 3
>>> ser = pd.Series([1.5, np.nan])
>>> ser
0    1.5
1    NaN
dtype: float64
>>> ser.astype("str")
0    1.5
1    nan
dtype: object
>>> ser.astype("str").to_numpy()
array(['1.5', 'nan'], dtype=object)
```
Note how ``NaN`` (``np.nan``) was converted to the string ``"nan"``. This was
not the intended behavior, and it was inconsistent with how other dtypes handled
missing values.

With pandas 3, this behavior has been fixed, and now ``astype("str")`` will cast
to the new string dtype, which preserves the missing values:

```python
# NEW behavior in pandas 3
>>> pd.options.future.infer_string = True
>>> ser = pd.Series([1.5, np.nan])
>>> ser.astype("str")
0    1.5
1    NaN
dtype: str
>>> ser.astype("str").to_numpy()
array(['1.5', nan], dtype=object)
```
If you want to preserve the old behaviour of converting every object to a
string, you can use ``ser.map(str)`` instead. If you want do such conversion
while preserving the missing values in a way that works with both pandas 2.x and
3.x, you can use ``ser.map(str, na_action="ignore")`` (for pandas 3.x only, you
can do ``ser.astype("str")``).

If you want to convert to object or string dtype for pandas 2.x and 3.x,
respectively, without needing to stringify each individual element, you will
have to use a conditional check on the pandas version.
For example, to convert a categorical Series with string categories to its
dense non-categorical version with object or string dtype:

```python
>>> import pandas as pd
>>> ser = pd.Series(["a", np.nan], dtype="category")
>>> ser.astype(object if pd.__version__ < "3" else "str")
```
#### ``prod()`` raising for string data
In pandas < 3, calling the `~pandas.Series.prod` method on a Series with
string data would generally raise an error, except when the Series was empty or
contained only a single string (potentially with missing values):

```python
>>> ser = pd.Series(["a", None], dtype=object)
>>> ser.prod()
'a'
```
When the Series contains multiple strings, it will raise a ``TypeError``. This
behaviour stays the same in pandas 3 when using the flexible ``object`` dtype.
But by virtue of using the new string dtype, this will generally consistently
raise an error regardless of the number of strings:

```python
>>> ser = pd.Series(["a", None], dtype="str")
>>> ser.prod()
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
...
TypeError: Cannot perform reduction 'prod' with string dtype
```
.. For existing users of the nullable ``StringDtype``
.. --------------------------------------------------

.. TODO

---

# Frequently Asked Questions (FAQ)


## DataFrame memory usage
The memory usage of a `DataFrame` (including the index) is shown when calling
the `~DataFrame.info`. A configuration option, ``display.memory_usage``
(see `the list of options <options.available>`), specifies if the
`DataFrame` memory usage will be displayed when invoking the `~DataFrame.info`
method.

For example, the memory usage of the `DataFrame` below is shown
when calling `~DataFrame.info`:

```python
dtypes = [
    "int64",
    "float64",
    "datetime64[ns]",
    "timedelta64[ns]",
    "complex128",
    "object",
    "bool",
]
n = 5000
data = {t: np.random.randint(100, size=n).astype(t) for t in dtypes}
df = pd.DataFrame(data)
df["categorical"] = df["object"].astype("category")

df.info()
```
The ``+`` symbol indicates that the true memory usage could be higher, because
pandas does not count the memory used by values in columns with
``dtype=object``.

Passing ``memory_usage='deep'`` will enable a more accurate memory usage report,
accounting for the full usage of the contained objects. This is optional
as it can be expensive to do this deeper introspection.

```python
df.info(memory_usage="deep")
```
By default the display option is set to ``True`` but can be explicitly
overridden by passing the ``memory_usage`` argument when invoking `~DataFrame.info`.

The memory usage of each column can be found by calling the
`~DataFrame.memory_usage` method. This returns a `Series` with an index
represented by column names and memory usage of each column shown in bytes. For
the `DataFrame` above, the memory usage of each column and the total memory
usage can be found with the `~DataFrame.memory_usage` method:

```python
df.memory_usage()

# total memory usage of dataframe
df.memory_usage().sum()
```
By default the memory usage of the `DataFrame` index is shown in the
returned `Series`, the memory usage of the index can be suppressed by passing
the ``index=False`` argument:

```python
df.memory_usage(index=False)
```
The memory usage displayed by the `~DataFrame.info` method utilizes the
`~DataFrame.memory_usage` method to determine the memory usage of a
`DataFrame` while also formatting the output in human-readable units (base-2
representation; i.e. 1KB = 1024 bytes).

See also `Categorical Memory Usage <categorical.memory>`.



## Using if/truth statements with pandas
pandas follows the NumPy convention of raising an error when you try to convert
something to a ``bool``. This happens in an ``if``-statement or when using the
boolean operations: ``and``, ``or``, and ``not``. It is not clear what the result
of the following code should be:

```python
>>> if pd.Series([False, True, False]):
...     pass
```
Should it be ``True`` because it's not zero-length, or ``False`` because there
are ``False`` values? It is unclear, so instead, pandas raises a ``ValueError``:

```python
:okexcept:

if pd.Series([False, True, False]):
    print("I was true")
```
You need to explicitly choose what you want to do with the `DataFrame`, e.g.
use `~DataFrame.any`, `~DataFrame.all` or `~DataFrame.empty`.
Alternatively, you might want to compare if the pandas object is ``None``:

```python
if pd.Series([False, True, False]) is not None:
    print("I was not None")
```
Below is how to check if any of the values are ``True``:

```python
if pd.Series([False, True, False]).any():
    print("I am any")
```
### Bitwise Boolean
Bitwise boolean operators like ``==`` and ``!=`` return a boolean `Series`
which performs an element-wise comparison when compared to a scalar.

```python
s = pd.Series(range(5))
s == 4
```
See `boolean comparisons<basics.compare>` for more examples.

### Using the ``in`` operator
Using the Python ``in`` operator on a `Series` tests for membership in the
**index**, not membership among the values.

```python
s = pd.Series(range(5), index=list("abcde"))
2 in s
'b' in s
```
If this behavior is surprising, keep in mind that using ``in`` on a Python
dictionary tests keys, not values, and `Series` are dict-like.
To test for membership in the values, use the method `~pandas.Series.isin`:

```python
s.isin([2])
s.isin([2]).any()
```
For `DataFrame`, likewise, ``in`` applies to the column axis,
testing for membership in the list of column names.



## Mutating with User Defined Function (UDF) methods
This section applies to pandas methods that take a UDF. In particular, the methods
`DataFrame.apply`, `DataFrame.aggregate`, `DataFrame.transform`, and
`DataFrame.filter`.

It is a general rule in programming that one should not mutate a container
while it is being iterated over. Mutation will invalidate the iterator,
causing unexpected behavior. Consider the example:

```python
values = [0, 1, 2, 3, 4, 5]
n_removed = 0
for k, value in enumerate(values):
    idx = k - n_removed
    if value % 2 == 1:
        del values[idx]
        n_removed += 1
    else:
        values[idx] = value + 1
values
```
One probably would have expected that the result would be ``[1, 3, 5]``.
When using a pandas method that takes a UDF, internally pandas is often
iterating over the
`DataFrame` or other pandas object. Therefore, if the UDF mutates (changes)
the `DataFrame`, unexpected behavior can arise.

Here is a similar example with `DataFrame.apply`:

```python
:okexcept:

def f(s):
    s.pop("a")
    return s

df = pd.DataFrame({"a": [1, 2, 3], "b": [4, 5, 6]})
df.apply(f, axis="columns")
```
To resolve this issue, one can make a copy so that the mutation does
not apply to the container being iterated over.

```python
values = [0, 1, 2, 3, 4, 5]
n_removed = 0
for k, value in enumerate(values.copy()):
    idx = k - n_removed
    if value % 2 == 1:
        del values[idx]
        n_removed += 1
    else:
        values[idx] = value + 1
values
```
```python
def f(s):
    s = s.copy()
    s.pop("a")
    return s

df = pd.DataFrame({"a": [1, 2, 3], 'b': [4, 5, 6]})
df.apply(f, axis="columns")
```
## Missing value representation for NumPy types
### ``np.nan`` as the ``NA`` representation for NumPy types
For lack of ``NA`` (missing) support from the ground up in NumPy and Python in
general, ``NA`` could have been represented with:

* A *masked array* solution: an array of data and an array of boolean values
  indicating whether a value is there or is missing.
* Using a special sentinel value, bit pattern, or set of sentinel values to
  denote ``NA`` across the dtypes.

The special value ``np.nan`` (Not-A-Number) was chosen as the ``NA`` value for NumPy types, and there are API
functions like `DataFrame.isna` and `DataFrame.notna` which can be used across the dtypes to
detect NA values. However, this choice has a downside of coercing missing integer data as float types as
shown in `gotchas.intna`.

### ``NA`` type promotions for NumPy types
When introducing NAs into an existing `Series` or `DataFrame` via
`~Series.reindex` or some other means, boolean and integer types will be
promoted to a different dtype in order to store the NAs. The promotions are
summarized in this table:


   :header: "Typeclass","Promotion dtype for storing NAs"
   :widths: 40,60

   ``floating``, no change
   ``object``, no change
   ``integer``, cast to ``float64``
   ``boolean``, cast to ``object``



### Support for integer ``NA``
In the absence of high performance ``NA`` support being built into NumPy from
the ground up, the primary casualty is the ability to represent NAs in integer
arrays. For example:

```python
s = pd.Series([1, 2, 3, 4, 5], index=list("abcde"))
s
s.dtype

s2 = s.reindex(["a", "b", "c", "f", "u"])
s2
s2.dtype
```
This trade-off is made largely for memory and performance reasons, and also so
that the resulting `Series` continues to be "numeric".

If you need to represent integers with possibly missing values, use one of
the nullable-integer extension dtypes provided by pandas or pyarrow

* `Int8Dtype`
* `Int16Dtype`
* `Int32Dtype`
* `Int64Dtype`
* `ArrowDtype`

```python
s_int = pd.Series([1, 2, 3, 4, 5], index=list("abcde"), dtype=pd.Int64Dtype())
s_int
s_int.dtype

s2_int = s_int.reindex(["a", "b", "c", "f", "u"])
s2_int
s2_int.dtype

s_int_pa = pd.Series([1, 2, None], dtype="int64[pyarrow]")
s_int_pa
```
See `integer_na` and `pyarrow` for more.

### Why not make NumPy like R?
Many people have suggested that NumPy should simply emulate the ``NA`` support
present in the more domain-specific statistical programming language `R
<https://www.r-project.org/>[__. Part of the reason is the
`NumPy type hierarchy](https://numpy.org/doc/stable/user/basics.types.html)_.

The R language, by contrast, only has a handful of built-in data types:
[`integer``, ``numeric`` (floating-point), ``character``, and
``boolean``. ``NA`` types are implemented by reserving special bit patterns for
each type to be used as the missing value. While doing this with the full NumPy
type hierarchy would be possible, it would be a more substantial trade-off
(especially for the 8- and 16-bit data types) and implementation undertaking.

However, R ``NA`` semantics are now available by using masked NumPy types such as `Int64Dtype`
or PyArrow types (`ArrowDtype`).


## Differences with NumPy
For `Series` and `DataFrame` objects, `~DataFrame.var` normalizes by
``N-1`` to produce `unbiased estimates of the population variance](https://en.wikipedia.org/wiki/Bias_of_an_estimator)_, while NumPy's
[numpy.var` normalizes by N, which measures the variance of the sample. Note that
`~DataFrame.cov` normalizes by ``N-1`` in both pandas and NumPy.



## Thread-safety
pandas is not 100% thread safe. The known issues relate to
the `~DataFrame.copy` method. If you are doing a lot of copying of
`DataFrame` objects shared among threads, we recommend holding locks inside
the threads where the data copying occurs.

See `this link](https://stackoverflow.com/questions/13592618/python-pandas-dataframe-thread-safe)_
for more information.


## Byte-ordering issues
Occasionally you may have to deal with data that were created on a machine with
a different byte order than the one on which you are running Python. A common
symptom of this issue is an error like::

    Traceback
        ...
    ValueError: Big-endian buffer not supported on little-endian compiler

To deal
with this issue you should convert the underlying NumPy array to the native
system byte order *before* passing it to `Series` or `DataFrame`
constructors using something similar to the following:

```python
x = np.array(list(range(10)), ">i4")  # big endian
newx = x.byteswap().view(x.dtype.newbyteorder())  # force native byteorder
s = pd.Series(newx)
```
See `the NumPy documentation on byte order
<https://numpy.org/doc/stable/user/byteswapping.html>`__ for more
details.

---

# Cookbook
This is a repository for *short and sweet* examples and links for useful pandas recipes.
We encourage users to add to this documentation.

Adding interesting links and/or inline examples to this section is a great *First Pull Request*.

Simplified, condensed, new-user friendly, in-line examples have been inserted where possible to
augment the Stack-Overflow and GitHub links.  Many of the links contain expanded information,
above what the in-line examples offer.

pandas (pd) and NumPy (np) are the only two abbreviated imported modules. The rest are kept
explicitly imported for newer users.

## Idioms


These are some neat pandas ``idioms``

`if-then/if-then-else on one column, and assignment to another one or more columns:
<https://stackoverflow.com/questions/17128302/python-pandas-idiom-for-if-then-else>[__

```python
df = pd.DataFrame(
    {"AAA": [4, 5, 6, 7], "BBB": [10, 20, 30, 40], "CCC": [100, 50, -30, -50]}
)
df
```
If-then...
# An if-then on one column

```python
df.loc[df.AAA >= 5, "BBB"] = -1
df
```
An if-then with assignment to 2 columns:

```python
df.loc[df.AAA >= 5, ["BBB", "CCC"]] = 555
df
```
Add another line with different logic, to do the -else

```python
df.loc[df.AAA]( 5, ["BBB", "CCC"]] = 2000
df
```
Or use pandas where after you've set up a mask

```python
df_mask = pd.DataFrame(
    {"AAA": [True] * 4, "BBB": [False] * 4, "CCC": [True, False] * 2}
)
df.where(df_mask, -1000)
```
`if-then-else using NumPy's where()
<https://stackoverflow.com/questions/19913659/pandas-conditional-creation-of-a-series-dataframe-column)_

```python
df = pd.DataFrame(
    {"AAA": [4, 5, 6, 7], "BBB": [10, 20, 30, 40], "CCC": [100, 50, -30, -50]}
)
df
df["logic"] = np.where(df["AAA"] > 5, "high", "low")
df
```
Splitting
`Split a frame with a boolean criterion
<https://stackoverflow.com/questions/14957116/how-to-split-a-dataframe-according-to-a-boolean-criterion>`__

```python
df = pd.DataFrame(
    {"AAA": [4, 5, 6, 7], "BBB": [10, 20, 30, 40], "CCC": [100, 50, -30, -50]}
)
df

df[df.AAA <= 5]
df[df.AAA > 5]
```
Building criteria
# `Select with multi-column criteria
<https://stackoverflow.com/questions/15315452/selecting-with-complex-criteria-from-pandas-dataframe>`__

```python
df = pd.DataFrame(
    {"AAA": [4, 5, 6, 7], "BBB": [10, 20, 30, 40], "CCC": [100, 50, -30, -50]}
)
df
```
...and (without assignment returns a Series)

```python
df.loc[(df["BBB"] < 25) & (df["CCC"] >= -40), "AAA"]
```
...or (without assignment returns a Series)

```python
df.loc[(df["BBB"] > 25) | (df["CCC"] >= -40), "AAA"]
```
...or (with assignment modifies the DataFrame.)

```python
df.loc[(df["BBB"] > 25) | (df["CCC"] >= 75), "AAA"] = 999
df
```
`Select rows with data closest to certain value using argsort
<https://stackoverflow.com/questions/17758023/return-rows-in-a-dataframe-closest-to-a-user-defined-number>`__

```python
df = pd.DataFrame(
    {"AAA": [4, 5, 6, 7], "BBB": [10, 20, 30, 40], "CCC": [100, 50, -30, -50]}
)
df
aValue = 43.0
df.loc[(df.CCC - aValue).abs().argsort()]
```
`Dynamically reduce a list of criteria using a binary operators
<https://stackoverflow.com/questions/21058254/pandas-boolean-operation-in-a-python-list/21058331>`__

```python
df = pd.DataFrame(
    {"AAA": [4, 5, 6, 7], "BBB": [10, 20, 30, 40], "CCC": [100, 50, -30, -50]}
)
df

Crit1 = df.AAA <= 5.5
Crit2 = df.BBB == 10.0
Crit3 = df.CCC > -40.0
```
One could hard code:

```python
AllCrit = Crit1 & Crit2 & Crit3
```
...Or it can be done with a list of dynamically built criteria

```python
import functools

CritList = [Crit1, Crit2, Crit3]
AllCrit = functools.reduce(lambda x, y: x & y, CritList)

df[AllCrit]
```


## Selection
DataFrames
The `indexing <indexing>` docs.

`Using both row labels and value conditionals
<https://stackoverflow.com/questions/14725068/pandas-using-row-labels-in-boolean-indexing>[__

```python
df = pd.DataFrame(
    {"AAA": [4, 5, 6, 7], "BBB": [10, 20, 30, 40], "CCC": [100, 50, -30, -50]}
)
df

df[(df.AAA](= 6) & (df.index.isin([0, 2, 4]))]
```
Use loc for label-oriented slicing and iloc positional slicing `2904`

```python
df = pd.DataFrame(
    {"AAA": [4, 5, 6, 7], "BBB": [10, 20, 30, 40], "CCC": [100, 50, -30, -50]},
    index=["foo", "bar", "boo", "kar"],
)
```
There are 2 explicit slicing methods, with a third general case

1. Positional-oriented (Python slicing style : exclusive of end)
2. Label-oriented (Non-Python slicing style : inclusive of end)
3. General (Either slicing style : depends on if the slice contains labels or positions)

```python
df.iloc[0:3]  # Positional

df.loc["bar":"kar"]  # Label

# Generic
df[0:3]
df["bar":"kar"]
```
Ambiguity arises when an index consists of integers with a non-zero start or non-unit increment.

```python
data = {"AAA": [4, 5, 6, 7], "BBB": [10, 20, 30, 40], "CCC": [100, 50, -30, -50]}
df2 = pd.DataFrame(data=data, index=[1, 2, 3, 4])  # Note index starts at 1.
df2.iloc[1:3]  # Position-oriented
df2.loc[1:3]  # Label-oriented
```
`Using inverse operator (~) to take the complement of a mask
<https://stackoverflow.com/q/14986510)_

[``python
df = pd.DataFrame(
    {"AAA": [4, 5, 6, 7], "BBB": [10, 20, 30, 40], "CCC": [100, 50, -30, -50]}
)
df

df[~((df.AAA](= 6) & (df.index.isin([0, 2, 4])))]
```
New columns
# `Efficiently and dynamically creating new columns using DataFrame.map (previously named applymap)
<https://stackoverflow.com/questions/16575868/efficiently-creating-additional-columns-in-a-pandas-dataframe-using-map)_

```python
df = pd.DataFrame({"AAA": [1, 2, 1, 3], "BBB": [1, 1, 2, 2], "CCC": [2, 1, 3, 1]})
df

source_cols = df.columns  # Or some subset would work too
new_cols = [str(x) + "_cat" for x in source_cols]
categories = {1: "Alpha", 2: "Beta", 3: "Charlie"}

df[new_cols] = df[source_cols].map(categories.get)
df
```
`Keep other columns when using min() with groupby
<https://stackoverflow.com/q/23394476>`__

```python
df = pd.DataFrame(
    {"AAA": [1, 1, 1, 2, 2, 2, 3, 3], "BBB": [2, 1, 3, 4, 5, 1, 2, 3]}
)
df
```
Method 1 : idxmin() to get the index of the minimums

```python
df.loc[df.groupby("AAA")["BBB"].idxmin()]
```
Method 2 : sort then take first of each

```python
df.sort_values(by="BBB").groupby("AAA", as_index=False).first()
```
Notice the same results, with the exception of the index.



## Multiindexing
The `multindexing <advanced.hierarchical>` docs.

`Creating a MultiIndex from a labeled frame
<https://stackoverflow.com/questions/14916358/reshaping-dataframes-in-pandas-based-on-column-labels>`__

```python
df = pd.DataFrame(
    {
        "row": [0, 1, 2],
        "One_X": [1.1, 1.1, 1.1],
        "One_Y": [1.2, 1.2, 1.2],
        "Two_X": [1.11, 1.11, 1.11],
        "Two_Y": [1.22, 1.22, 1.22],
    }
)
df

# As Labelled Index
df = df.set_index("row")
df
# With Hierarchical Columns
df.columns = pd.MultiIndex.from_tuples([tuple(c.split("_")) for c in df.columns])
df
# Now stack & Reset
df = df.stack(0).reset_index(1)
df
# And fix the labels (Notice the label 'level_1' got added automatically)
df.columns = ["Sample", "All_X", "All_Y"]
df
```
Arithmetic
`Performing arithmetic with a MultiIndex that needs broadcasting
<https://stackoverflow.com/questions/19501510/divide-entire-pandas-multiindex-dataframe-by-dataframe-variable/19502176#19502176>`__

```python
cols = pd.MultiIndex.from_tuples(
    [(x, y) for x in ["A", "B", "C"] for y in ["O", "I"]]
)
df = pd.DataFrame(np.random.randn(2, 6), index=["n", "m"], columns=cols)
df
df = df.div(df["C"], level=1)
df
```
Slicing
# `Slicing a MultiIndex with xs
<https://stackoverflow.com/questions/12590131/how-to-slice-multindex-columns-in-pandas-dataframes>`__

```python
coords = [("AA", "one"), ("AA", "six"), ("BB", "one"), ("BB", "two"), ("BB", "six")]
index = pd.MultiIndex.from_tuples(coords)
df = pd.DataFrame([11, 22, 33, 44, 55], index, ["MyData"])
df
```
To take the cross section of the 1st level and 1st axis the index:

```python
# Note : level and axis are optional, and default to zero
df.xs("BB", level=0, axis=0)
```
...and now the 2nd level of the 1st axis.

```python
df.xs("six", level=1, axis=0)
```
`Slicing a MultiIndex with xs, method #2
<https://stackoverflow.com/questions/14964493/multiindex-based-indexing-in-pandas>`__

```python
import itertools

index = list(itertools.product(["Ada", "Quinn", "Violet"], ["Comp", "Math", "Sci"]))
headr = list(itertools.product(["Exams", "Labs"], ["I", "II"]))
indx = pd.MultiIndex.from_tuples(index, names=["Student", "Course"])
cols = pd.MultiIndex.from_tuples(headr)  # Notice these are un-named
data = [[70 + x + y + (x * y) % 3 for x in range(4)] for y in range(9)]
df = pd.DataFrame(data, indx, cols)
df

All = slice(None)
df.loc["Violet"]
df.loc[(All, "Math"), All]
df.loc[(slice("Ada", "Quinn"), "Math"), All]
df.loc[(All, "Math"), ("Exams")]
df.loc[(All, "Math"), (All, "II")]
```
`Setting portions of a MultiIndex with xs
<https://stackoverflow.com/questions/19319432/pandas-selecting-a-lower-level-in-a-dataframe-to-do-a-ffill>`__

Sorting
`Sort by specific column or an ordered list of columns, with a MultiIndex
<https://stackoverflow.com/q/14733871>`__

```python
df.sort_values(by=("Labs", "II"), ascending=False)
```
Partial selection, the need for sortedness `2995`

Levels
# `Prepending a level to a multiindex
<https://stackoverflow.com/questions/14744068/prepend-a-level-to-a-pandas-multiindex>`__

`Flatten Hierarchical columns
<https://stackoverflow.com/q/14507794>`__



## Missing data
The `missing data<missing_data>` docs.

Fill forward a reversed timeseries

```python
df = pd.DataFrame(
    np.random.randn(6, 1),
    index=pd.date_range("2013-08-01", periods=6, freq="B"),
    columns=list("A"),
)
df.loc[df.index[3], "A"] = np.nan
df
df.bfill()
```
`cumsum reset at NaN values
<https://stackoverflow.com/questions/18196811/cumsum-reset-at-nan>`__

Replace
`Using replace with backrefs
<https://stackoverflow.com/questions/16818871/extracting-value-and-creating-new-column-out-of-it>`__



## Grouping
The `grouping <groupby>` docs.

`Basic grouping with apply
<https://stackoverflow.com/questions/15322632/python-pandas-df-groupy-agg-column-reference-in-agg>`__

Unlike agg, apply's callable is passed a sub-DataFrame which gives you access to all the columns

```python
df = pd.DataFrame(
    {
        "animal": "cat dog cat fish dog cat cat".split(),
        "size": list("SSMMMLL"),
        "weight": [8, 10, 11, 1, 20, 12, 12],
        "adult": [False] * 5 + [True] * 2,
    }
)
df

# List the size of the animals with the highest weight.
df.groupby("animal").apply(lambda subf: subf["size"][subf["weight"].idxmax()])
```
`Using get_group
<https://stackoverflow.com/questions/14734533/how-to-access-pandas-groupby-dataframe-by-key>`__

```python
gb = df.groupby("animal")
gb.get_group("cat")
```
`Apply to different items in a group
<https://stackoverflow.com/questions/15262134/apply-different-functions-to-different-items-in-group-object-python-pandas>`__

```python
def GrowUp(x):
    avg_weight = sum(x[x["size"] == "S"].weight * 1.5)
    avg_weight += sum(x[x["size"] == "M"].weight * 1.25)
    avg_weight += sum(x[x["size"] == "L"].weight)
    avg_weight /= len(x)
    return pd.Series(["L", avg_weight, True], index=["size", "weight", "adult"])


expected_df = gb.apply(GrowUp)
expected_df
```
`Expanding apply
<https://stackoverflow.com/questions/14542145/reductions-down-a-column-in-pandas>`__

```python
S = pd.Series([i / 100.0 for i in range(1, 11)])

def cum_ret(x, y):
    return x * (1 + y)

def red(x):
    return functools.reduce(cum_ret, x, 1.0)

S.expanding().apply(red, raw=True)
```
`Replacing some values with mean of the rest of a group
<https://stackoverflow.com/questions/14760757/replacing-values-with-groupby-means>[__

```python
df = pd.DataFrame({"A": [1, 1, 2, 2], "B": [1, -1, 1, 2]})
gb = df.groupby("A")

def replace(g):
    mask = g]( 0
    return g.where(~mask, g[~mask].mean())

gb.transform(replace)
```
`Sort groups by aggregated data
<https://stackoverflow.com/questions/14941366/pandas-sort-by-group-aggregate-and-column)_

```python
df = pd.DataFrame(
    {
        "code": ["foo", "bar", "baz"] * 2,
        "data": [0.16, -0.21, 0.33, 0.45, -0.59, 0.62],
        "flag": [False, True] * 3,
    }
)

code_groups = df.groupby("code")

agg_n_sort_order = code_groups[["data"]].transform("sum").sort_values(by="data")

sorted_df = df.loc[agg_n_sort_order.index]

sorted_df
```
`Create multiple aggregated columns
<https://stackoverflow.com/questions/14897100/create-multiple-columns-in-pandas-aggregation-function>`__

```python
rng = pd.date_range(start="2014-10-07", periods=10, freq="2min")
ts = pd.Series(data=list(range(10)), index=rng)

def MyCust(x):
    if len(x) > 2:
        return x.iloc[1] * 1.234
    return pd.NaT

mhc = {"Mean": "mean", "Max": "max", "Custom": MyCust}
ts.resample("5min").apply(mhc)
ts
```
`Create a value counts column and reassign back to the DataFrame
<https://stackoverflow.com/q/17709270>`__

```python
df = pd.DataFrame(
    {"Color": "Red Red Red Blue".split(), "Value": [100, 150, 50, 50]}
)
df
df["Counts"] = df.groupby(["Color"]).transform(len)
df
```
`Shift groups of the values in a column based on the index
<https://stackoverflow.com/q/23198053/190597>`__

```python
df = pd.DataFrame(
    {"line_race": [10, 10, 8, 10, 10, 8], "beyer": [99, 102, 103, 103, 88, 100]},
    index=[
        "Last Gunfighter",
        "Last Gunfighter",
        "Last Gunfighter",
        "Paynter",
        "Paynter",
        "Paynter",
    ],
)
df
df["beyer_shifted"] = df.groupby(level=0)["beyer"].shift(1)
df
```
`Select row with maximum value from each group
<https://stackoverflow.com/q/26701849/190597>`__

```python
df = pd.DataFrame(
    {
        "host": ["other", "other", "that", "this", "this"],
        "service": ["mail", "web", "mail", "mail", "web"],
        "no": [1, 2, 1, 2, 1],
    }
).set_index(["host", "service"])
mask = df.groupby(level=0).agg("idxmax")
df_count = df.loc[mask["no"]].reset_index()
df_count
```
`Grouping like Python's itertools.groupby
<https://stackoverflow.com/q/29142487/846892>`__

```python
df = pd.DataFrame([0, 1, 0, 1, 1, 1, 0, 1, 1], columns=["A"])
df["A"].groupby((df["A"] != df["A"].shift()).cumsum()).groups
df["A"].groupby((df["A"] != df["A"].shift()).cumsum()).cumsum()
```
Expanding data
# `Alignment and to-date
<https://stackoverflow.com/questions/15489011/python-time-series-alignment-and-to-date-functions>`__

`Rolling Computation window based on values instead of counts
<https://stackoverflow.com/questions/14300768/pandas-rolling-computation-with-window-based-on-values-instead-of-counts>`__

`Rolling Mean by Time Interval
<https://stackoverflow.com/questions/15771472/pandas-rolling-mean-by-time-interval>`__

Splitting
`Splitting a frame
<https://stackoverflow.com/questions/13353233/best-way-to-split-a-dataframe-given-an-edge/15449992#15449992>`__

Create a list of dataframes, split using a delineation based on logic included in rows.

```python
df = pd.DataFrame(
    data={
        "Case": ["A", "A", "A", "B", "A", "A", "B", "A", "A"],
        "Data": np.random.randn(9),
    }
)

dfs = list(
    zip(
        *df.groupby(
            (1 * (df["Case"] == "B"))
            .cumsum()
            .rolling(window=3, min_periods=1)
            .median()
        )
    )
)[-1]

dfs[0]
dfs[1]
dfs[2]
```


Pivot
# The `Pivot <reshaping.pivot>` docs.

`Partial sums and subtotals
<https://stackoverflow.com/a/15574875>`__

```python
df = pd.DataFrame(
    data={
        "Province": ["ON", "QC", "BC", "AL", "AL", "MN", "ON"],
        "City": [
            "Toronto",
            "Montreal",
            "Vancouver",
            "Calgary",
            "Edmonton",
            "Winnipeg",
            "Windsor",
        ],
        "Sales": [13, 6, 16, 8, 4, 3, 1],
    }
)
table = pd.pivot_table(
    df,
    values=["Sales"],
    index=["Province"],
    columns=["City"],
    aggfunc="sum",
    margins=True,
)
table.stack("City")
```
`Frequency table like plyr in R
<https://stackoverflow.com/questions/15589354/frequency-tables-in-pandas-like-plyr-in-r>`__

```python
grades = [48, 99, 75, 80, 42, 80, 72, 68, 36, 78]
df = pd.DataFrame(
    {
        "ID": ["x%d" % r for r in range(10)],
        "Gender": ["F", "M", "F", "M", "F", "M", "F", "M", "M", "M"],
        "ExamYear": [
            "2007",
            "2007",
            "2007",
            "2008",
            "2008",
            "2008",
            "2008",
            "2009",
            "2009",
            "2009",
        ],
        "Class": [
            "algebra",
            "stats",
            "bio",
            "algebra",
            "algebra",
            "stats",
            "stats",
            "algebra",
            "bio",
            "bio",
        ],
        "Participated": [
            "yes",
            "yes",
            "yes",
            "yes",
            "no",
            "yes",
            "yes",
            "yes",
            "yes",
            "yes",
        ],
        "Passed": ["yes" if x > 50 else "no" for x in grades],
        "Employed": [
            True,
            True,
            True,
            False,
            False,
            False,
            False,
            True,
            True,
            False,
        ],
        "Grade": grades,
    }
)

df.groupby("ExamYear").agg(
    {
        "Participated": lambda x: x.value_counts()["yes"],
        "Passed": lambda x: sum(x == "yes"),
        "Employed": lambda x: sum(x),
        "Grade": lambda x: sum(x) / len(x),
    }
)
```
`Plot pandas DataFrame with year over year data
<https://stackoverflow.com/questions/30379789/plot-pandas-data-frame-with-year-over-year-data>`__

To create year and month cross tabulation:

```python
df = pd.DataFrame(
    {"value": np.random.randn(36)},
    index=pd.date_range("2011-01-01", freq="ME", periods=36),
)

pd.pivot_table(
    df, index=df.index.month, columns=df.index.year, values="value", aggfunc="sum"
)
```
Apply
`Rolling apply to organize - Turning embedded lists into a MultiIndex frame
<https://stackoverflow.com/questions/17349981/converting-pandas-dataframe-with-categorical-values-into-binary-values>`__

```python
df = pd.DataFrame(
    data={
        "A": [[2, 4, 8, 16], [100, 200], [10, 20, 30]],
        "B": [["a", "b", "c"], ["jj", "kk"], ["ccc"]],
    },
    index=["I", "II", "III"],
)

def SeriesFromSubList(aList):
    return pd.Series(aList)

df_orgz = pd.concat(
    {ind: row.apply(SeriesFromSubList) for ind, row in df.iterrows()}
)
df_orgz
```
`Rolling apply with a DataFrame returning a Series
<https://stackoverflow.com/questions/19121854/using-rolling-apply-on-a-dataframe-object>`__

Rolling Apply to multiple columns where function calculates a Series before a Scalar from the Series is returned

```python
df = pd.DataFrame(
    data=np.random.randn(2000, 2) / 10000,
    index=pd.date_range("2001-01-01", periods=2000),
    columns=["A", "B"],
)
df

def gm(df, const):
    v = ((((df["A"] + df["B"]) + 1).cumprod()) - 1) * const
    return v.iloc[-1]

s = pd.Series(
    {
        df.index[i]: gm(df.iloc[i: min(i + 51, len(df) - 1)], 5)
        for i in range(len(df) - 50)
    }
)
s
```
`Rolling apply with a DataFrame returning a Scalar
<https://stackoverflow.com/questions/21040766/python-pandas-rolling-apply-two-column-input-into-function/21045831#21045831>`__

Rolling Apply to multiple columns where function returns a Scalar (Volume Weighted Average Price)

```python
rng = pd.date_range(start="2014-01-01", periods=100)
df = pd.DataFrame(
    {
        "Open": np.random.randn(len(rng)),
        "Close": np.random.randn(len(rng)),
        "Volume": np.random.randint(100, 2000, len(rng)),
    },
    index=rng,
)
df

def vwap(bars):
    return (bars.Close * bars.Volume).sum() / bars.Volume.sum()

window = 5
s = pd.concat(
    [
        (pd.Series(vwap(df.iloc[i: i + window]), index=[df.index[i + window]]))
        for i in range(len(df) - window)
    ]
)
s.round(2)
```
## Timeseries
`Between times
<https://stackoverflow.com/questions/14539992/pandas-drop-rows-outside-of-time-range>`__

`Using indexer between time
<https://stackoverflow.com/questions/17559885/pandas-dataframe-mask-based-on-index>`__

`Constructing a datetime range that excludes weekends and includes only certain times
<https://stackoverflow.com/a/24014440>`__

`Vectorized Lookup
<https://stackoverflow.com/questions/13893227/vectorized-look-up-of-values-in-pandas-dataframe>`__

`Aggregation and plotting time series
<https://nipunbatra.github.io/blog/posts/2013-05-01-aggregation-timeseries.html>`__

Turn a matrix with hours in columns and days in rows into a continuous row sequence in the form of a time series.
`How to rearrange a Python pandas DataFrame?
<https://stackoverflow.com/questions/15432659/how-to-rearrange-a-python-pandas-dataframe>`__

`Dealing with duplicates when reindexing a timeseries to a specified frequency
<https://stackoverflow.com/questions/22244383/pandas-df-refill-adding-two-columns-of-different-shape>`__

Calculate the first day of the month for each entry in a DatetimeIndex

```python
dates = pd.date_range("2000-01-01", periods=5)
dates.to_period(freq="M").to_timestamp()
```


Resampling
# The `Resample <timeseries.resampling>` docs.

`Using Grouper instead of TimeGrouper for time grouping of values
<https://stackoverflow.com/questions/15297053/how-can-i-divide-single-values-of-a-dataframe-by-monthly-averages>`__

`Time grouping with some missing values
<https://stackoverflow.com/questions/33637312/pandas-grouper-by-frequency-with-completeness-requirement>`__

Valid frequency arguments to Grouper `Timeseries <timeseries.offset_aliases>`

`Grouping using a MultiIndex
<https://stackoverflow.com/questions/41483763/pandas-timegrouper-on-multiindex>`__

Using TimeGrouper and another grouping to create subgroups, then apply a custom function `3791`

`Resampling with custom periods
<https://stackoverflow.com/questions/15408156/resampling-with-custom-periods>`__

`Resample intraday frame without adding new days
<https://stackoverflow.com/questions/14898574/resample-intraday-pandas-dataframe-without-add-new-days>`__

`Resample minute data
<https://stackoverflow.com/questions/14861023/resampling-minute-data>[__

`Resample with groupby](https://stackoverflow.com/q/18677271/564538)_



## Merge
The `Join <merging.join>` docs.

`Concatenate two dataframes with overlapping index (emulate R rbind)
<https://stackoverflow.com/questions/14988480/pandas-version-of-rbind>`__

```python
rng = pd.date_range("2000-01-01", periods=6)
df1 = pd.DataFrame(np.random.randn(6, 3), index=rng, columns=["A", "B", "C"])
df2 = df1.copy()
```
Depending on df construction, ``ignore_index`` may be needed

```python
df = pd.concat([df1, df2], ignore_index=True)
df
```
Self Join of a DataFrame `2996`

```python
df = pd.DataFrame(
    data={
        "Area": ["A"] * 5 + ["C"] * 2,
        "Bins": [110] * 2 + [160] * 3 + [40] * 2,
        "Test_0": [0, 1, 0, 1, 2, 0, 1],
        "Data": np.random.randn(7),
    }
)
df

df["Test_1"] = df["Test_0"] - 1

pd.merge(
    df,
    df,
    left_on=["Bins", "Area", "Test_0"],
    right_on=["Bins", "Area", "Test_1"],
    suffixes=("_L", "_R"),
)
```
`How to set the index and join
<https://stackoverflow.com/questions/14341805/pandas-merge-pd-merge-how-to-set-the-index-and-join>`__

`KDB like asof join
<https://stackoverflow.com/questions/12322289/kdb-like-asof-join-for-timeseries-data-in-pandas/12336039#12336039>`__

`Join with a criteria based on the values
<https://stackoverflow.com/questions/15581829/how-to-perform-an-inner-or-outer-join-of-dataframes-with-pandas-on-non-simplisti>`__

`Using searchsorted to merge based on values inside a range
<https://stackoverflow.com/questions/25125626/pandas-merge-with-logic/2512764>`__



## Plotting
The `Plotting <visualization>` docs.

`Make Matplotlib look like R
<https://stackoverflow.com/questions/14349055/making-matplotlib-graphs-look-like-r-by-default>`__

`Setting x-axis major and minor labels
<https://stackoverflow.com/questions/12945971/pandas-timeseries-plot-setting-x-axis-major-and-minor-ticks-and-labels>`__

`Plotting multiple charts in an IPython Jupyter notebook
<https://stackoverflow.com/questions/16392921/make-more-than-one-chart-in-same-ipython-notebook-cell>`__

`Creating a multi-line plot
<https://stackoverflow.com/questions/16568964/make-a-multiline-plot-from-csv-file-in-matplotlib>`__

`Plotting a heatmap
<https://stackoverflow.com/questions/17050202/plot-timeseries-of-histograms-in-python>`__

`Annotate a time-series plot
<https://stackoverflow.com/questions/11067368/annotate-time-series-plot-in-matplotlib>`__

`Annotate a time-series plot #2
<https://stackoverflow.com/questions/17891493/annotating-points-from-a-pandas-dataframe-in-matplotlib-plot>`__

`Generate Embedded plots in excel files using Pandas, Vincent and xlsxwriter
<https://pandas-xlsxwriter-charts.readthedocs.io/>`__

`Boxplot for each quartile of a stratifying variable
<https://stackoverflow.com/questions/23232989/boxplot-stratified-by-column-in-python-pandas>`__

```python
df = pd.DataFrame(
    {
        "stratifying_var": np.random.uniform(0, 100, 20),
        "price": np.random.normal(100, 5, 20),
    }
)

df["quartiles"] = pd.qcut(
    df["stratifying_var"], 4, labels=["0-25%", "25-50%", "50-75%", "75-100%"]
)

@savefig quartile_boxplot.png
df.boxplot(column="price", by="quartiles")
```
## Data in/out
`Performance comparison of SQL vs HDF5
<https://stackoverflow.com/q/16628329>`__



CSV
The `CSV <io.read_csv_table>[ docs

`read_csv in action](https://www.datacamp.com/tutorial/pandas-read-csv)_

`appending to a csv
<https://stackoverflow.com/questions/17134942/pandas-dataframe-output-end-of-csv>`__

`Reading a csv chunk-by-chunk
<https://stackoverflow.com/questions/11622652/large-persistent-dataframe-in-pandas/12193309#12193309>`__

`Reading only certain rows of a csv chunk-by-chunk
<https://stackoverflow.com/questions/19674212/pandas-data-frame-select-rows-and-clear-memory>`__

`Reading the first few lines of a frame
<https://stackoverflow.com/questions/15008970/way-to-read-first-few-lines-for-pandas-dataframe>`__

Reading a file that is compressed but not by ``gzip/bz2`` (the native compressed formats which ``read_csv`` understands).
This example shows a ``WinZipped`` file, but is a general application of opening the file within a context manager and
using that handle to read.
`See here
<https://stackoverflow.com/questions/17789907/pandas-convert-winzipped-csv-file-to-data-frame>`__

`Inferring dtypes from a file
<https://stackoverflow.com/questions/15555005/get-inferred-dataframe-types-iteratively-using-chunksize>`__

Dealing with bad lines `2886`

`Write a multi-row index CSV without writing duplicates
<https://stackoverflow.com/questions/17349574/pandas-write-multiindex-rows-with-to-csv>`__



#### Reading multiple files to create a single DataFrame
The best way to combine multiple files into a single DataFrame is to read the individual frames one by one, put all
of the individual frames into a list, and then combine the frames in the list using `pd.concat`:

```python
for i in range(3):
    data = pd.DataFrame(np.random.randn(10, 4))
    data.to_csv("file_{}.csv".format(i))

files = ["file_0.csv", "file_1.csv", "file_2.csv"]
result = pd.concat([pd.read_csv(f) for f in files], ignore_index=True)
```
You can use the same approach to read all files matching a pattern.  Here is an example using ``glob``:

```python
import glob
import os

files = glob.glob("file_*.csv")
result = pd.concat([pd.read_csv(f) for f in files], ignore_index=True)
```
Finally, this strategy will work with the other ``pd.read_*(...)`` functions described in the `io docs<io>`.

```python
:suppress:

for i in range(3):
    os.remove("file_{}.csv".format(i))
```
#### Parsing date components in multi-columns
Parsing date components in multi-columns is faster with a format

```python
i = pd.date_range("20000101", periods=10000)
df = pd.DataFrame({"year": i.year, "month": i.month, "day": i.day})
df.head()

%timeit pd.to_datetime(df.year * 10000 + df.month * 100 + df.day, format='%Y%m%d')
ds = df.apply(lambda x: "%04d%02d%02d" % (x["year"], x["month"], x["day"]), axis=1)
ds.head()
%timeit pd.to_datetime(ds)
```
#### Skip row between header and data
```python
data = """;;;;
 ;;;;
 ;;;;
 ;;;;
 ;;;;
 ;;;;
;;;;
 ;;;;
 ;;;;
;;;;
date;Param1;Param2;Param4;Param5
    ;m²;°C;m²;m
;;;;
01.01.1990 00:00;1;1;2;3
01.01.1990 01:00;5;3;4;5
01.01.1990 02:00;9;5;6;7
01.01.1990 03:00;13;7;8;9
01.01.1990 04:00;17;9;10;11
01.01.1990 05:00;21;11;12;13
"""
```
Option 1: pass rows explicitly to skip rows
"""""""""""""""""""""""""""""""""""""""""""

```python
from io import StringIO

pd.read_csv(
    StringIO(data),
    sep=";",
    skiprows=[11, 12],
    index_col=0,
    parse_dates=True,
    header=10,
)
```
Option 2: read column names and then data
"""""""""""""""""""""""""""""""""""""""""

```python
pd.read_csv(StringIO(data), sep=";", header=10, nrows=10).columns
columns = pd.read_csv(StringIO(data), sep=";", header=10, nrows=10).columns
pd.read_csv(
    StringIO(data), sep=";", index_col=0, header=12, parse_dates=True, names=columns
)
```


SQL
# The `SQL <io.sql>` docs

`Reading from databases with SQL
<https://stackoverflow.com/questions/10065051/python-pandas-and-databases-like-mysql>`__



Excel
The `Excel <io.excel>` docs

`Reading from a filelike handle
<https://stackoverflow.com/questions/15588713/sheets-of-excel-workbook-from-a-url-into-a-pandas-dataframe>`__

`Modifying formatting in XlsxWriter output
<https://pbpython.com/improve-pandas-excel-output.html>[__

Loading only visible sheets `19842#issuecomment-892150745`



HTML
# `Reading HTML tables from a server that cannot handle the default request
header](https://stackoverflow.com/a/18939272/564538)_



HDFStore
The `HDFStores <io.hdf5>` docs

`Simple queries with a Timestamp Index
<https://stackoverflow.com/questions/13926089/selecting-columns-from-pandas-hdfstore-table>`__

Managing heterogeneous data using a linked multiple table hierarchy `3032`

`Merging on-disk tables with millions of rows
<https://stackoverflow.com/questions/14614512/merging-two-tables-with-millions-of-rows-in-python/14617925#14617925>`__

`Avoiding inconsistencies when writing to a store from multiple processes/threads
<https://stackoverflow.com/a/29014295/2858145>`__

De-duplicating a large store by chunks, essentially a recursive reduction operation. Shows a function for taking in data from
csv file and creating a store by chunks, with date parsing as well.
`See here
<https://stackoverflow.com/questions/16110252/need-to-compare-very-large-files-around-1-5gb-in-python/16110391#16110391>`__

`Creating a store chunk-by-chunk from a csv file
<https://stackoverflow.com/questions/20428355/appending-column-to-frame-of-hdf-file-in-pandas/20428786#20428786>`__

`Appending to a store, while creating a unique index
<https://stackoverflow.com/questions/16997048/how-does-one-append-large-amounts-of-data-to-a-pandas-hdfstore-and-get-a-natural/16999397#16999397>`__

`Large Data work flows
<https://stackoverflow.com/q/14262433>`__

`Reading in a sequence of files, then providing a global unique index to a store while appending
<https://stackoverflow.com/questions/16997048/how-does-one-append-large-amounts-of-data-to-a-pandas-hdfstore-and-get-a-natural>`__

`Groupby on a HDFStore with low group density
<https://stackoverflow.com/questions/15798209/pandas-group-by-query-on-large-data-in-hdfstore>`__

`Groupby on a HDFStore with high group density
<https://stackoverflow.com/questions/25459982/trouble-with-grouby-on-millions-of-keys-on-a-chunked-file-in-python-pandas/25471765#25471765>`__

`Hierarchical queries on a HDFStore
<https://stackoverflow.com/questions/22777284/improve-query-performance-from-a-large-hdfstore-table-with-pandas/22820780#22820780>`__

`Counting with a HDFStore
<https://stackoverflow.com/questions/20497897/converting-dict-of-dicts-into-pandas-dataframe-memory-issues>`__

`Troubleshoot HDFStore exceptions
<https://stackoverflow.com/questions/15488809/how-to-trouble-shoot-hdfstore-exception-cannot-find-the-correct-atom-type>`__

`Setting min_itemsize with strings
<https://stackoverflow.com/questions/15988871/hdfstore-appendstring-dataframe-fails-when-string-column-contents-are-longer>`__

`Using ptrepack to create a completely-sorted-index on a store
<https://stackoverflow.com/questions/17893370/ptrepack-sortby-needs-full-index>`__

Storing Attributes to a group node

```python
df = pd.DataFrame(np.random.randn(8, 3))
store = pd.HDFStore("test.h5")
store.put("df", df)

# you can store an arbitrary Python object via pickle
store.get_storer("df").attrs.my_attribute = {"A": 10}
store.get_storer("df").attrs.my_attribute
```
```python
:suppress:

store.close()
os.remove("test.h5")
```
You can create or load a HDFStore in-memory  by passing the ``driver``
parameter to PyTables. Changes are only written to disk when the HDFStore
is closed.

```python
store = pd.HDFStore("test.h5", "w", driver="H5FD_CORE")

df = pd.DataFrame(np.random.randn(8, 3))
store["test"] = df

# only after closing the store, data is written to disk:
store.close()
```
```python
:suppress:

os.remove("test.h5")
```


Binary files
# pandas readily accepts NumPy record arrays, if you need to read in a binary
file consisting of an array of C structs. For example, given this C program
in a file called ``main.c`` compiled with ``gcc main.c -std=gnu99`` on a
64-bit machine,



   #include <stdio.h>
   #include <stdint.h>

   typedef struct _Data
   {
       int32_t count;
       double avg;
       float scale;
   } Data;

   int main(int argc, const char *argv[])
   {
       size_t n = 10;
       Data d[n];

       for (int i = 0; i < n; ++i)
       {
           d[i].count = i;
           d[i].avg = i + 1.0;
           d[i].scale = (float) i + 2.0f;
       }

       FILE *file = fopen("binary.dat", "wb");
       fwrite(&d, sizeof(Data), n, file);
       fclose(file);

       return 0;
   }

the following Python code will read the binary file ``'binary.dat'`` into a
pandas ``DataFrame``, where each element of the struct corresponds to a column
in the frame:

```python
names = "count", "avg", "scale"

# note that the offsets are larger than the size of the type because of
# struct padding
offsets = 0, 8, 16
formats = "i4", "f8", "f4"
dt = np.dtype({"names": names, "offsets": offsets, "formats": formats}, align=True)
df = pd.DataFrame(np.fromfile("binary.dat", dt))
```
> **note.capitalize():**
   The offsets of the structure elements may be different depending on the
   architecture of the machine on which the file was created. Using a raw
   binary file format like this for general data storage is not recommended, as
   it is not cross platform. We recommended either HDF5 or parquet, both of
   which are supported by pandas' IO facilities.

## Computation
`Numerical integration (sample-based) of a time series
<https://nbviewer.ipython.org/gist/metakermit/5720498>[__

Correlation
Often it's useful to obtain the lower (or upper) triangular form of a correlation matrix calculated from `DataFrame.corr`.  This can be achieved by passing a boolean mask to ``where`` as follows:

```python
df = pd.DataFrame(np.random.random(size=(100, 5)))

corr_mat = df.corr()
mask = np.tril(np.ones_like(corr_mat, dtype=np.bool_), k=-1)

corr_mat.where(mask)
```
The ``method`` argument within ``DataFrame.corr`` can accept a callable in addition to the named correlation types.  Here we compute the `distance correlation](https://en.wikipedia.org/wiki/Distance_correlation)_ matrix for a ``DataFrame`` object.

```python
def distcorr(x, y):
    n = len(x)
    a = np.zeros(shape=(n, n))
    b = np.zeros(shape=(n, n))
    for i in range(n):
        for j in range(i + 1, n):
            a[i, j] = abs(x[i] - x[j])
            b[i, j] = abs(y[i] - y[j])
    a += a.T
    b += b.T
    a_bar = np.vstack([np.nanmean(a, axis=0)] * n)
    b_bar = np.vstack([np.nanmean(b, axis=0)] * n)
    A = a - a_bar - a_bar.T + np.full(shape=(n, n), fill_value=a_bar.mean())
    B = b - b_bar - b_bar.T + np.full(shape=(n, n), fill_value=b_bar.mean())
    cov_ab = np.sqrt(np.nansum(A * B)) / n
    std_a = np.sqrt(np.sqrt(np.nansum(A ** 2)) / n)
    std_b = np.sqrt(np.sqrt(np.nansum(B ** 2)) / n)
    return cov_ab / std_a / std_b


df = pd.DataFrame(np.random.normal(size=(100, 3)))
df.corr(method=distcorr)
```
## Timedeltas
The `Timedeltas <timedeltas.timedeltas>` docs.

`Using timedeltas
<https://github.com/pandas-dev/pandas/pull/2899>`__

```python
import datetime

s = pd.Series(pd.date_range("2012-1-1", periods=3, freq="D"))

s - s.max()

s.max() - s

s - datetime.datetime(2011, 1, 1, 3, 5)

s + datetime.timedelta(minutes=5)

datetime.datetime(2011, 1, 1, 3, 5) - s

datetime.timedelta(minutes=5) + s
```
`Adding and subtracting deltas and dates
<https://stackoverflow.com/questions/16385785/add-days-to-dates-in-dataframe>`__

```python
deltas = pd.Series([datetime.timedelta(days=i) for i in range(3)])

df = pd.DataFrame({"A": s, "B": deltas})
df

df["New Dates"] = df["A"] + df["B"]

df["Delta"] = df["A"] - df["New Dates"]
df

df.dtypes
```
`Another example
<https://stackoverflow.com/questions/15683588/iterating-through-a-pandas-dataframe>`__

Values can be set to NaT using np.nan, similar to datetime

```python
y = s - s.shift()
y

y[1] = np.nan
y
```
## Creating example data
To create a dataframe from every combination of some given values, like R's ``expand.grid()``
function, we can create a dict where the keys are column names and the values are lists
of the data values:

```python
def expand_grid(data_dict):
    rows = itertools.product(*data_dict.values())
    return pd.DataFrame.from_records(rows, columns=data_dict.keys())


df = expand_grid(
    {"height": [60, 70], "weight": [100, 140, 180], "sex": ["Male", "Female"]}
)
df
```
## Constant Series
To assess if a series has a constant value, we can check if ``series.nunique() <= 1``.
However, a more performant approach, that does not count all unique values first, is:

```python
v = s.to_numpy()
is_constant = v.shape[0] == 0 or (s[0] == s).all()
```
This approach assumes that the series does not contain missing values.
For the case that we would drop NA values, we can simply remove those values first:

```python
v = s.dropna().to_numpy()
is_constant = v.shape[0] == 0 or (s[0] == s).all()
```
If missing values are considered distinct from any other value, then one could use:

```python
v = s.to_numpy()
is_constant = v.shape[0] == 0 or (s[0] == s).all() or not pd.notna(v).any()
```
(Note that this example does not disambiguate between ``np.nan``, ``pd.NA`` and ``None``)

---

