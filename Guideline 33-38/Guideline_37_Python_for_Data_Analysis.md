# Guideline 37: Python for Data Analysis

Real data rarely arrives as a tidy array of numbers. It arrives as a table: customers with names and cities, sales with dates and amounts, students with grades in several subjects. Each column has its own type, each row has its own meaning, and the first questions are always the same. Which rows matter? What is the average per group? How do I attach this table to that one?

NumPy is built for uniform grids of numbers. For labeled, mixed-type tables, Python's standard tool is **pandas**. This guideline is deliberately simplified: it covers the operations you will use in almost every analysis, in the order you tend to need them.

## 37.1 Pandas

Pandas is an external library built on top of NumPy (Guideline 34), so it inherits NumPy's speed. It is installed with pip and imported with the alias `pd`:

```bash
pip install pandas
```

```python
import numpy as np
import pandas as pd
```

Pandas has two core structures:

| Structure | What it is | Comparable to |
|---|---|---|
| `Series` | A single labeled column of values | A NumPy array with an index |
| `DataFrame` | A table of rows and columns; every column is a `Series` | A spreadsheet or a database table |

In practice, data usually comes from a file rather than being typed in. Reading and writing are one-liners:

```python
df = pd.read_csv("employees.csv")      # load a CSV file
df.to_csv("output.csv", index=False)   # save it (index=False skips the row numbers)

# Also available: pd.read_excel(), pd.read_json(), pd.read_sql()
```

To keep every example below self-contained, the data is created directly in code.

## 37.2 Pandas Series and DataFrames

**Series.** A `Series` is a list of values with a label on each one. If no index is given, the labels are the numbers 0, 1, 2, ...:

```python
s = pd.Series([10, 20, 30], index=["a", "b", "c"])

print(s)
# a    10
# b    20
# c    30
# dtype: int64

print(s["b"])       # 20
print(s * 2)        # every value doubled, labels kept
print(s.mean())     # 20.0
```

**DataFrame from a dictionary of lists.** The most direct way to build a table is a dictionary where each key is a column name and each value is the list of that column's entries. All lists must be the same length. This table is used for the rest of the guideline:

```python
data = {
    "name":       ["Alice", "Bob", "Charlie", "Diana", "Ethan", "Fiona"],
    "department": ["Sales", "IT", "IT", "HR", "Sales", "IT"],
    "age":        [28, 35, 42, 31, 26, 38],
    "salary":     [52000, 68000, 81000, 59000, 48000, 75000],
    "city":       ["Paris", "Tokyo", "Paris", "Lima", "Tokyo", "Paris"],
}

df = pd.DataFrame(data)
print(df)
#       name  department  age  salary   city
# 0    Alice       Sales   28   52000  Paris
# 1      Bob          IT   35   68000  Tokyo
# 2  Charlie          IT   42   81000  Paris
# 3    Diana          HR   31   59000   Lima
# 4    Ethan       Sales   26   48000  Tokyo
# 5    Fiona          IT   38   75000  Paris
```

The numbers down the left side are the **index**, the row labels. Pandas numbers them from 0 by default.

**Setting the index.** Any column can become the index with `set_index()`, which is useful when a column uniquely identifies each row. It returns a new table, and `reset_index()` reverses it:

```python
emp = df.set_index("name")
print(emp)
#            department  age  salary   city
# name
# Alice           Sales   28   52000  Paris
# Bob                IT   35   68000  Tokyo
# Charlie            IT   42   81000  Paris
# Diana              HR   31   59000   Lima
# Ethan           Sales   26   48000  Tokyo
# Fiona              IT   38   75000  Paris

back = emp.reset_index()    # "name" becomes an ordinary column again
```

**Looking at the data: `head`, `tail`, and length.** For a large table, printing everything is impractical, so peek at the ends instead:

```python
print(df.head(3))       # first 3 rows (the default is 5)
#       name  department  age  salary   city
# 0    Alice       Sales   28   52000  Paris
# 1      Bob          IT   35   68000  Tokyo
# 2  Charlie          IT   42   81000  Paris

print(df.tail(2))       # last 2 rows
#     name  department  age  salary   city
# 4  Ethan       Sales   26   48000  Tokyo
# 5  Fiona          IT   38   75000  Paris

print(len(df))          # 6        number of rows
print(df.shape)         # (6, 5)   (rows, columns)
print(df.columns.tolist())   # ['name', 'department', 'age', 'salary', 'city']
```

**`info`: the structure at a glance.** This reports the number of rows, each column's data type, and how many values are not missing:

```python
df.info()
# <class 'pandas.core.frame.DataFrame'>
# RangeIndex: 6 entries, 0 to 5
# Data columns (total 5 columns):
#  #   Column      Non-Null Count  Dtype
# ---  ------      --------------  -----
#  0   name        6 non-null      object
#  1   department  6 non-null      object
#  2   age         6 non-null      int64
#  3   salary      6 non-null      int64
#  4   city        6 non-null      object
# dtypes: int64(2), object(3)
# memory usage: 372.0+ bytes
```

Text columns appear as `object` in most versions, and as `str` in the newest ones, and the memory figure varies between machines. Comparing `Non-Null Count` against the number of rows is the quickest way to spot missing data.

**`describe`: the summary.** For every numeric column, this gives the count, mean, standard deviation, minimum, quartiles, and maximum:

```python
print(df.describe().round(2))
#          age    salary
# count   6.00      6.00
# mean   33.33  63833.33
# std     6.12  13044.79
# min    26.00  48000.00
# 25%    28.75  53750.00
# 50%    33.00  63500.00
# 75%    37.25  73250.00
# max    42.00  81000.00
```

**Retrieving a column.** Square brackets with a column name return a `Series`. A *list* of names returns a smaller `DataFrame`:

```python
print(df["age"])
# 0    28
# 1    35
# 2    42
# 3    31
# 4    26
# 5    38
# Name: age, dtype: int64

subset = df[["name", "salary"]]     # note the double brackets: a list inside the brackets
```

Columns without spaces in their names can also be reached as `df.age`, but the bracket form is safer, because it works for every name and cannot be confused with a built-in method.

**Unique values.** These answer "what categories exist, and how common is each?":

```python
print(df["department"].unique().tolist())   # ['Sales', 'IT', 'HR']
print(df["department"].nunique())           # 3     how many different values

print(df["department"].value_counts())
# department
# IT       3
# Sales    2
# HR       1
# Name: count, dtype: int64
```

**Duplicates.** `duplicated()` marks every row (or value) that repeats an earlier one, and `drop_duplicates()` removes them:

```python
print(df["city"].duplicated().tolist())
# [False, False, True, False, True, True]   Paris, Tokyo, Paris repeat earlier cities

print(df.duplicated().sum())      # 0     no row of the whole table is repeated

# Add a repeated row to see the effect
dup = pd.concat([df, df.iloc[[0]]], ignore_index=True)
print(dup.duplicated().tolist())              # [False, False, False, False, False, False, True]
print(len(dup.drop_duplicates()))             # 6

print(len(df.drop_duplicates(subset="city")))   # 3     keeps the first row for each city
```

By default, the first occurrence counts as the original and later ones as duplicates. The option `keep=False` marks *all* copies instead.

## 37.3 Indexing and Slicing with `loc` and `iloc`

Pandas offers two ways to pick rows and columns, and the difference is the most important thing to learn in this section:

| | `df.loc[...]` | `df.iloc[...]` |
|---|---|---|
| Selects by | **Labels** (index values and column names) | **Positions** (0, 1, 2, ...) |
| Slice end | **Included** | **Excluded**, as with lists |
| Form | `df.loc[rows, columns]` | `df.iloc[rows, columns]` |

The examples use `emp`, the table indexed by name, so labels and positions look different:

```python
# loc: by label
print(emp.loc["Bob", "salary"])                  # 68000
print(emp.loc[["Alice", "Fiona"], "salary"].tolist())   # [52000, 75000]

print(emp.loc["Bob":"Diana", ["age", "salary"]])
#          age  salary
# name
# Bob       35   68000
# Charlie   42   81000
# Diana     31   59000      <- "Diana" is included

# iloc: by position
print(emp.iloc[0, 1])                            # 28     first row, second column (age)

print(emp.iloc[1:3, [1, 2]])
#          age  salary
# name
# Bob       35   68000
# Charlie   42   81000      <- position 3 is excluded
```

A single number or label returns one row as a `Series`; a slice or a list returns a `DataFrame`. The colon `:` on its own means "everything":

```python
print(emp.iloc[-1].tolist())        # ['IT', 38, 75000, 'Paris']   last row
print(emp.iloc[:, 0].tolist())      # ['Sales', 'IT', 'IT', 'HR', 'Sales', 'IT']   first column
print(emp.iloc[::2].index.tolist()) # ['Alice', 'Charlie', 'Ethan']   every second row
```

`loc` also accepts a **condition** in place of row labels, which combines selecting and filtering in one step (filtering is covered in the next section):

```python
print(emp.loc[emp["age"] > 35, "salary"])
# name
# Charlie    81000
# Fiona      75000
# Name: salary, dtype: int64
```

Be careful with `iloc` on the default index. In `df`, the labels and positions both run 0 to 5, so `loc` and `iloc` look identical. After filtering or sorting, the labels stay attached to their rows while the positions restart, and the two methods start to disagree. Deciding whether you mean "the row labeled 3" or "the fourth row" keeps you safe.

## 37.4 Filtering

Filtering keeps only the rows that satisfy a condition, and the pattern is always `df[condition]`. The condition is a column compared against something, which produces a `True`/`False` value for each row (called a **mask**). Only the `True` rows survive:

```python
mask = df["age"] > 30
print(mask.tolist())     # [False, True, True, True, False, True]

print(df[df["age"] > 30])
#       name  department  age  salary   city
# 1      Bob          IT   35   68000  Tokyo
# 2  Charlie          IT   42   81000  Paris
# 3    Diana          HR   31   59000   Lima
# 5    Fiona          IT   38   75000  Paris
```

Notice that the original row labels are kept. The result is a new table, so filtering never changes `df` itself.

**Comparative conditions.**

| Operator | Meaning | Example |
|---|---|---|
| `>` | Greater than | `df[df["age"] > 30]` |
| `<` | Less than | `df[df["salary"] < 50000]` |
| `>=` | Greater than or equal | `df[df["age"] >= 35]` |
| `<=` | Less than or equal | `df[df["age"] <= 28]` |
| `==` | Equal to | `df[df["department"] == "IT"]` |
| `!=` | Not equal to | `df[df["city"] != "Paris"]` |

**Logical conditions.** Combine several conditions with `&` (and), `|` (or), and `~` (not). Two rules matter here: each condition must be wrapped in **parentheses**, and Python's words `and`, `or`, and `not` do not work on whole columns:

```python
# IT employees earning more than 70000
print(df[(df["department"] == "IT") & (df["salary"] > 70000)])
#       name  department  age  salary   city
# 2  Charlie          IT   42   81000  Paris
# 5    Fiona          IT   38   75000  Paris

# Either in Tokyo, or older than 40
print(df[(df["city"] == "Tokyo") | (df["age"] > 40)]["name"].tolist())
# ['Bob', 'Charlie', 'Ethan']

# Everyone who is NOT in IT
print(df[~(df["department"] == "IT")]["name"].tolist())
# ['Alice', 'Diana', 'Ethan']
```

**Subset-based conditions.** `isin()` tests membership in a list, which replaces a long chain of `|`:

```python
print(df[df["city"].isin(["Paris", "Lima"])]["name"].tolist())
# ['Alice', 'Charlie', 'Diana', 'Fiona']
```

`any()` and `all()` do not filter rows. They collapse a `True`/`False` mask into a single answer, which makes them ideal for checks:

```python
print((df["salary"] > 80000).any())    # True    at least one salary is above 80000
print((df["age"] > 25).all())          # True    everyone is older than 25
print((df["age"] > 30).sum())          # 4       how many rows satisfy the condition
print((df["age"] > 30).mean())         # 0.666...  the fraction that do
```

Counting with `sum()` works because Python treats `True` as 1 and `False` as 0, so the sum of a mask is the number of matches, and its mean is the proportion.

**String conditions.** For text columns, the `.str` accessor unlocks string methods that work on the whole column at once:

```python
names = df["name"]

print(df[names.str.contains("a")]["name"].tolist())
# ['Charlie', 'Diana', 'Ethan', 'Fiona']       "Alice" has no lowercase a

print(df[names.str.contains("a", case=False)]["name"].tolist())
# ['Alice', 'Charlie', 'Diana', 'Ethan', 'Fiona']

print(df[names.str.startswith("C")]["name"].tolist())    # ['Charlie']
print(df[names.str.endswith("a")]["name"].tolist())      # ['Diana', 'Fiona']
```

| Method | Keeps rows where the text... |
|---|---|
| `.str.contains("x")` | contains "x" anywhere (case-sensitive; `case=False` ignores case) |
| `.str.startswith("x")` | begins with "x" |
| `.str.endswith("x")` | ends with "x" |

Two details are worth knowing. `contains()` treats its argument as a *regular expression* by default, so a pattern such as `"."` matches any character, and `regex=False` makes it search for literal text. And if the column has missing values, add `na=False`, so that missing entries count as "no match" instead of causing an error.

## 37.5 Insertion, Deletion, Update

Each example below starts from the original `df`, using a copy so that the earlier sections stay unchanged:

```python
staff = df.copy()
```

### Insert

**A new column** is created by assigning to a name that does not exist yet:

```python
staff["bonus"] = staff["salary"] // 10                # 5200, 6800, 8100, 5900, 4800, 7500
staff["level"] = np.where(staff["salary"] >= 60000, "Senior", "Junior")

print(staff[["name", "salary", "level"]])
#       name  salary   level
# 0    Alice   52000  Junior
# 1      Bob   68000  Senior
# 2  Charlie   81000  Senior
# 3    Diana   59000  Junior
# 4    Ethan   48000  Junior
# 5    Fiona   75000  Senior
```

Assigning always adds the column at the far right. To choose a position, use `insert(position, name, values)`:

```python
staff.insert(1, "id", range(101, 107))     # becomes the second column
```

**A new row** can be added by assigning to a new label with `loc`. To add several rows, build a small table and stack it with `pd.concat`:

```python
staff.loc[6] = ["Grace", 101, "HR", 29, 55000, "Lima", 5500, "Junior"]   # one row

new_rows = pd.DataFrame({"name": ["Hugo"], "department": ["IT"]})
staff = pd.concat([staff, new_rows], ignore_index=True)   # columns not listed become missing (NaN)
```

Adding rows one at a time in a loop is slow, so for many rows, collect them first and create a single table. The older `df.append()` was removed from pandas, and `pd.concat` replaces it.

### Delete

`drop()` removes rows or columns. It returns a new table, so assign the result back if you want to keep it:

```python
staff = df.copy()
staff["bonus"] = staff["salary"] // 10

staff = staff.drop(columns=["bonus"])        # remove a column
staff = staff.drop(columns=["city", "age"])  # or several at once
staff = staff.drop(index=[0, 5])             # remove rows by their labels (here, Alice and Fiona)
print(staff["name"].tolist())                # ['Bob', 'Charlie', 'Diana', 'Ethan']
```

To delete rows by *condition*, do not drop them. Instead, filter to keep the rows you want:

```python
staff = df[df["age"] >= 30]        # keeps Bob, Charlie, Diana, Fiona
```

`del staff["col"]` and `staff.pop("col")` also remove a column, and `pop` hands the removed column back to you.

### Update

Changing existing values is done with `loc`, which selects the target cells and assigns to them:

```python
staff = df.copy()

staff.loc[staff["name"] == "Bob", "salary"] = 70000                 # one cell
staff.loc[staff["department"] == "HR", "department"] = "People"     # every row matching a condition
staff["age"] = staff["age"] + 1                                     # a whole column at once
staff = staff.rename(columns={"city": "location"})                  # rename a column
```

A frequent mistake is *chained indexing*: writing `staff[staff["age"] > 30]["salary"] = 0`. This filters first, and then assigns to the temporary result, so the original table stays unchanged. Use a single `loc` call, as in the examples above.

### Replacing values with `replace` and `map`

Both swap values in a column, but they treat values that are *not* listed differently:

```python
mapping = {"Sales": "S", "IT": "T"}

print(df["department"].replace(mapping).tolist())
# ['S', 'T', 'T', 'HR', 'S', 'T']         unlisted values ("HR") are left alone

print(df["department"].map(mapping).tolist())
# ['S', 'T', 'T', nan, 'S', 'T']          unlisted values become missing
```

| | `replace()` | `map()` |
|---|---|---|
| Unlisted values | Kept as they are | Become `NaN` |
| Best for | Fixing some values (typos, old labels) | Translating *every* value (codes to labels) |
| Also accepts | A single pair: `replace("IT", "Tech")` | A function: `map(str.upper)` |

Both return a new column, so assign the result back to keep it:

```python
staff = df.copy()
staff["department"] = staff["department"].replace({"HR": "People"})
staff["name"] = staff["name"].map(str.upper)            # ALICE, BOB, ...
staff["name"] = staff["name"].map(lambda n: n[:3])      # ALI, BOB, CHA, ...
```

## 37.6 Grouping

Analysis often asks questions about *groups*: the average salary per department, the number of employees per city. `groupby()` answers them by following three steps, which is called **split-apply-combine**:

1. **Split** the table into groups that share a value.
2. **Apply** a calculation to each group separately.
3. **Combine** the results into a new table.

### Aggregating

An **aggregation** boils each group down to a single value:

```python
print(df.groupby("department")["salary"].mean().round(2))
# department
# HR       59000.00
# IT       74666.67
# Sales    50000.00
# Name: salary, dtype: float64
```

The pattern reads left to right: group by `department`, look at the `salary` column, take the `mean`. Groups appear in alphabetical order. Other common aggregations are `sum`, `min`, `max`, `count`, `median`, `std`, and `size`, where `size` counts the rows in each group:

```python
print(df.groupby("department").size())
# department
# HR       1
# IT       3
# Sales    2
# dtype: int64
```

For several results at once, use `agg()` with **named aggregation**, where each new column is defined as `name=(column, function)`:

```python
summary = df.groupby("department").agg(
    total_salary=("salary", "sum"),
    oldest=("age", "max"),
    headcount=("name", "count"),
)
print(summary)
#             total_salary  oldest  headcount
# department
# HR                 59000      31          1
# IT                224000      42          3
# Sales             100000      28          2
```

The group names become the index of the result. Use `.reset_index()` to turn them back into a normal column, or pass `as_index=False` to `groupby()`. Grouping by *several* columns takes a list, for instance `df.groupby(["department", "city"])`.

### Transforming

An aggregation shrinks the table to one row per group. A **transformation** does the calculation per group but returns a result with the **same length as the original**, so every row gets its own group's value. This makes it easy to compare each row to its group:

```python
df2 = df.copy()
df2["dept_avg"] = df2.groupby("department")["salary"].transform("mean")
df2["vs_avg"] = df2["salary"] - df2["dept_avg"]

print(df2["dept_avg"].round(2).tolist())
# [50000.0, 74666.67, 74666.67, 59000.0, 50000.0, 74666.67]

print(df2["vs_avg"].round(2).tolist())
# [2000.0, -6666.67, 6333.33, 0.0, -2000.0, 333.33]
```

Alice earns 2000 above the Sales average, and Bob earns about 6667 below the IT average. Comparing rows to their own group is the most common use of `transform`.

| | Aggregate (`agg`, `mean`, ...) | Transform (`transform`) |
|---|---|---|
| Result has | One row per group | One row per original row |
| Use it to | Summarize each group | Add a group-level value back to every row |

## 37.7 Merging

Data is often spread across several tables. Employees are listed in one, and each department's manager in another. **Merging** combines tables by matching the values of a shared column, the **key**, exactly like a join in a database.

```python
emp_dept = df[["name", "department"]]

depts = pd.DataFrame({
    "department": ["Sales", "IT", "Finance"],
    "manager":    ["Kim", "Lee", "Park"],
})
```

Note that HR has no manager listed, and Finance has no employees. That mismatch is exactly what makes the choice of merge type matter.

**`merge()`.** The `on` argument names the key column, and `how` decides which rows to keep:

```python
print(emp_dept.merge(depts, on="department"))              # how="inner" is the default
#       name  department  manager
# 0    Alice       Sales      Kim
# 1      Bob          IT      Lee
# 2  Charlie          IT      Lee
# 3    Ethan       Sales      Kim
# 4    Fiona          IT      Lee

print(emp_dept.merge(depts, on="department", how="left"))
#       name  department  manager
# 0    Alice       Sales      Kim
# 1      Bob          IT      Lee
# 2  Charlie          IT      Lee
# 3    Diana          HR      NaN
# 4    Ethan       Sales      Kim
# 5    Fiona          IT      Lee
```

Diana disappears from the inner merge, because HR does not appear in `depts`, but the left merge keeps her, and fills the missing manager with `NaN`.

| `how=` | Keeps | Rows without a match get |
|---|---|---|
| `"inner"` (default) | Only keys found in **both** tables | (they are dropped) |
| `"left"` | Every row of the left table | `NaN` in the right table's columns |
| `"right"` | Every row of the right table | `NaN` in the left table's columns |
| `"outer"` | Every row from both tables | `NaN` wherever a side is missing |

Some other options you will meet:

```python
# Key columns with different names
orders.merge(customers, left_on="cust_id", right_on="id")

# Several key columns
a.merge(b, on=["year", "region"])

# Add a column showing where each row came from: "both", "left_only", or "right_only"
emp_dept.merge(depts, on="department", how="outer", indicator=True)

# Raise an error if the keys are not unique on the right, which catches accidental row duplication
emp_dept.merge(depts, on="department", validate="many_to_one")
```

If both tables contain a column with the same name that is *not* a key, pandas keeps both and adds the suffixes `_x` and `_y`, which the `suffixes=("_left", "_right")` argument lets you change.

**`join()`.** This is a shortcut for the common case of merging on the **index**. Called on the left table, it takes the right table (whose index should hold the key) and defaults to a left join:

```python
depts_indexed = depts.set_index("department")

joined = emp_dept.join(depts_indexed, on="department")   # same result as the left merge above
```

The `on="department"` tells `join` to match that *column* of the left table against the *index* of the right one. Without `on`, it matches index against index.

| | `merge()` | `join()` |
|---|---|---|
| Matches on | Any column(s), by default | The index of the right table |
| Default `how` | `"inner"` | `"left"` |
| Best when | Keys are ordinary columns | Tables are already indexed by their key |

## 37.8 Cheatsheets

**`concat`: stacking tables.** Unlike `merge`, `concat` does not match on keys. It simply glues tables together, either below each other (`axis=0`, the default) or side by side (`axis=1`):

```python
part1 = df.iloc[:2]
part2 = df.iloc[2:4]

stacked = pd.concat([part1, part2])
print(stacked.shape)                       # (4, 5)

stacked = pd.concat([part1, part2], ignore_index=True)   # renumber the rows 0, 1, 2, 3

side_by_side = pd.concat([df["name"], df["age"]], axis=1)
print(side_by_side.shape)                  # (6, 2)
```

**`cut` and `qcut`: turning numbers into categories.** `cut` sorts values into bins that *you* define. Each bin includes its right edge, so `(0, 29]` means "above 0, up to and including 29":

```python
groups = pd.cut(df["age"], bins=[0, 29, 39, 100],
                labels=["under 30", "30s", "40+"])

print(groups.tolist())
# ['under 30', '30s', '40+', '30s', 'under 30', '30s']
```

`qcut` instead creates bins that each hold (about) the *same number of rows*, splitting at quantiles:

```python
levels = pd.qcut(df["salary"], q=3, labels=["low", "mid", "high"])

print(levels.tolist())
# ['low', 'mid', 'high', 'mid', 'low', 'high']
```

**`get_dummies`: turning categories into numbers.** Many analysis and machine learning tools need numbers rather than text. **One-hot encoding** creates one column per category, marked `True` for the rows that belong to it:

```python
print(pd.get_dummies(df["department"]))
#       HR     IT  Sales
# 0  False  False   True
# 1  False   True  False
# 2  False   True  False
# 3   True  False  False
# 4  False  False   True
# 5  False   True  False

pd.get_dummies(df["department"], dtype=int)               # 0/1 instead of False/True
pd.get_dummies(df, columns=["department"])                # encode inside the whole table
pd.get_dummies(df["department"], drop_first=True)         # drop one column (avoids redundancy)
```

**Quick reference.**

| Task | Method | Example |
|---|---|---|
| Read / write files | `read_csv`, `to_csv`, `read_excel`, `to_excel` | `pd.read_csv("data.csv")` |
| Inspect | `head`, `tail`, `shape`, `dtypes`, `info`, `describe` | `df.dtypes` |
| Count categories | `value_counts`, `nunique` | `df["city"].value_counts()` |
| Sort | `sort_values` | `df.sort_values("age", ascending=False)` |
| Top / bottom rows | `nlargest`, `nsmallest` | `df.nlargest(2, "salary")` |
| Filter with text | `query` | `df.query("age > 30 and department == 'IT'")` |
| Random rows | `sample` | `df.sample(3)` |
| Missing values (check) | `isna` | `df.isna().sum()` |
| Missing values (remove) | `dropna` | `df.dropna()` |
| Missing values (fill) | `fillna` | `df["age"].fillna(df["age"].mean())` |
| Change data type | `astype` | `df["age"].astype(float)` |
| Convert to numbers / dates | `pd.to_numeric`, `pd.to_datetime` | `pd.to_numeric(col, errors="coerce")` |
| Apply a function | `apply`, `map` | `df["salary"].apply(lambda s: s / 1000)` |
| Clean text | `str.strip`, `str.lower`, `str.upper`, `str.replace` | `df["name"].str.lower()` |
| Running calculations | `cumsum`, `rank`, `diff`, `pct_change`, `rolling` | `df["salary"].cumsum()` |
| Limit values | `clip` | `df["age"].clip(lower=30)` |
| Cross-tabulate | `pivot_table`, `crosstab` | `df.pivot_table(values="salary", index="department", aggfunc="mean")` |
| Wide to long | `melt` | `df.melt(id_vars="name")` |
| Combine tables | `concat`, `merge`, `join` | `pd.concat([a, b])` |
| Categorize numbers | `cut`, `qcut` | `pd.cut(df["age"], bins=3)` |
| One-hot encode | `get_dummies` | `pd.get_dummies(df["city"])` |
| Numeric correlation | `corr` | `df[["age", "salary"]].corr()` |

A few of these in action, using the guideline's table:

```python
print(df.nlargest(2, "salary")["name"].tolist())                 # ['Charlie', 'Fiona']

print(df.sort_values("age", ascending=False)["name"].tolist())
# ['Charlie', 'Fiona', 'Bob', 'Diana', 'Alice', 'Ethan']

print(df.query("age > 30 and department == 'IT'")["name"].tolist())
# ['Bob', 'Charlie', 'Fiona']

s = pd.Series([1, None, 3])
print(s.isna().sum())          # 1
print(s.fillna(0).tolist())    # [1.0, 0.0, 3.0]
```

One habit is worth building from the start. Most pandas methods return a *new* table instead of changing the old one, and the older `inplace=True` option is discouraged. Assign the result back, as in `df = df.dropna()`, so that it is always clear which version of the data you are holding.

---

The ideas in this guideline reduce to a small vocabulary: **select** the columns and rows you need, **filter** by condition, **change** values with `loc`, **group** to summarize, and **merge** to combine. Almost every real analysis is some sequence of those five moves, followed by a chart or a model. Learn them well, and the rest of pandas (there is a great deal more) will read as variations on the same themes.
