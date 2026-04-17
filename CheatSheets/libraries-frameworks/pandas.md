# Pandas: A Complete Progressive Tutorial

---

## 1. What & Why

Pandas is Python's primary library for working with structured tabular data. It provides two core data structures — `Series` (a labeled one-dimensional array) and `DataFrame` (a labeled two-dimensional table) — plus a comprehensive toolkit for loading, cleaning, transforming, aggregating, and analyzing data.

Why pandas? Because working with CSV files, database query results, or any tabular data in plain Python is verbose and slow. Filtering a list of dictionaries, computing grouped averages, joining two datasets, or reshaping data all require significant code with no optimization. Pandas handles these operations with concise syntax and leverages NumPy's C-compiled array operations under the hood — making it orders of magnitude faster than Python loops for numerical work.

You will use pandas in data science, data engineering, financial analysis, scientific computing, and anywhere that data is tabular. It is the lingua franca of Python data work.

---

## 2. Mental Model

A DataFrame is a dictionary of Series objects, all sharing the same index.

```
DataFrame: sales data

        date         product   region    units   revenue
index: 
  0    2024-01-01   Widget A   North      12     599.88
  1    2024-01-01   Widget B   South       8     479.92
  2    2024-01-02   Widget A   East        5     249.95
  3    2024-01-02   Widget C   North      20    1199.80

Each column is a Series:
  df["units"] → Series([12, 8, 5, 20], index=[0,1,2,3])
  
The index (row labels) enables:
  - Label-based alignment during joins and merges
  - Slicing and selection by label
  - Time-series indexing when index is DatetimeIndex
```

The critical distinction: pandas has two selection systems — `.loc[]` (label-based) and `.iloc[]` (position-based). Most bugs come from confusing these.

---

## 3. Progressive Examples

### Level 1: Creating DataFrames and Basic Operations

```python
import pandas as pd
import numpy as np

# --- Creating DataFrames ---

# From a list of dicts (most common when loading data programmatically)
employees = pd.DataFrame([
    {"name": "Alice",  "dept": "Engineering", "salary": 95000, "years": 4},
    {"name": "Bob",    "dept": "Marketing",   "salary": 72000, "years": 2},
    {"name": "Carol",  "dept": "Engineering", "salary": 110000, "years": 8},
    {"name": "Dave",   "dept": "Marketing",   "salary": 68000, "years": 1},
    {"name": "Eve",    "dept": "Engineering", "salary": 88000, "years": 3},
])

# From a dict of lists (column-oriented)
df = pd.DataFrame({
    "A": [1, 2, 3],
    "B": ["x", "y", "z"],
    "C": [True, False, True],
})

# From CSV, Excel, JSON, SQL
df = pd.read_csv("data.csv")
df = pd.read_excel("data.xlsx", sheet_name="Sheet1")
df = pd.read_json("data.json")

import sqlite3
conn = sqlite3.connect("database.db")
df = pd.read_sql("SELECT * FROM sales WHERE date > '2024-01-01'", conn)

# --- Basic inspection ---
print(employees.shape)      # (5, 4) — rows, columns
print(employees.dtypes)     # data type of each column
print(employees.describe()) # count, mean, std, min, max, quartiles (numeric cols)
print(employees.info())     # dtypes, non-null counts, memory usage
print(employees.head(3))    # first 3 rows
print(employees.tail(2))    # last 2 rows
print(employees.columns.tolist())  # ['name', 'dept', 'salary', 'years']
print(employees.index.tolist())    # [0, 1, 2, 3, 4]

# --- Column selection ---
print(employees["name"])               # Series
print(employees[["name", "salary"]])   # DataFrame (double brackets)
```

### Level 2: Selection, Filtering, and the .loc/.iloc Distinction

```python
# THE MOST IMPORTANT DISTINCTION: .loc vs .iloc

# .loc[row_label, col_label] — label-based
# .iloc[row_position, col_position] — integer position-based

df = pd.DataFrame({"A": [10, 20, 30], "B": [40, 50, 60]}, index=["x", "y", "z"])
#     A   B
# x  10  40
# y  20  50
# z  30  60

# Select single value
print(df.loc["y", "A"])    # 20 — row label "y", column label "A"
print(df.iloc[1, 0])       # 20 — row position 1, column position 0

# Select rows
print(df.loc["x":"y"])     # rows x and y (loc: END INCLUSIVE)
print(df.iloc[0:2])        # rows at position 0,1 (iloc: END EXCLUSIVE, like Python slices)

# Select columns
print(df.loc[:, "A"])      # all rows, column "A"
print(df.iloc[:, 0])       # all rows, first column

# Boolean indexing — the primary filtering mechanism
employees = pd.DataFrame([
    {"name": "Alice",  "dept": "Engineering", "salary": 95000, "years": 4},
    {"name": "Bob",    "dept": "Marketing",   "salary": 72000, "years": 2},
    {"name": "Carol",  "dept": "Engineering", "salary": 110000, "years": 8},
    {"name": "Dave",   "dept": "Marketing",   "salary": 68000, "years": 1},
    {"name": "Eve",    "dept": "Engineering", "salary": 88000, "years": 3},
])

# Single condition
high_earners = employees[employees["salary"] > 85000]

# Multiple conditions — use & (and) | (or) — NOT 'and'/'or'
senior_engineers = employees[
    (employees["dept"] == "Engineering") &
    (employees["years"] >= 4)
]

# NOT a condition
not_marketing = employees[employees["dept"] != "Marketing"]
not_marketing = employees[~(employees["dept"] == "Marketing")]  # equivalent

# .isin() — check membership
target_depts = employees[employees["dept"].isin(["Engineering", "Design"])]

# .between() — range check
mid_salary = employees[employees["salary"].between(70000, 100000)]

# .str accessor for string operations
name_starts_with_a = employees[employees["name"].str.startswith("A")]
has_ing = employees[employees["dept"].str.contains("ing")]

# .query() — readable SQL-like filtering (good for interactive use)
result = employees.query("salary > 85000 and dept == 'Engineering'")
result = employees.query("salary > @threshold", threshold=90000)  # use Python variables with @
```

### Level 3: Transforming Data

```python
# --- Adding and modifying columns ---

df = employees.copy()

# Add a computed column
df["salary_k"] = df["salary"] / 1000           # vectorized arithmetic
df["senior"] = df["years"] >= 5                 # boolean column
df["bonus"] = df["salary"] * df["years"] * 0.01  # depends on two columns

# Apply a function to a column
df["name_upper"] = df["name"].str.upper()
df["name_len"] = df["name"].str.len()

# Apply a custom function — vectorized approach (fastest)
def salary_band(salary):
    if salary < 75000:
        return "Junior"
    elif salary < 100000:
        return "Mid"
    else:
        return "Senior"

# Vectorized with np.where (for two categories)
df["tier"] = np.where(df["salary"] >= 90000, "High", "Standard")

# np.select for multiple categories
conditions = [df["salary"] < 75000, df["salary"] < 95000, df["salary"] >= 95000]
choices = ["Junior", "Mid", "Senior"]
df["band"] = np.select(conditions, choices)

# .apply() — apply a function per row or column (slower, use when vectorization fails)
df["label"] = df["salary"].apply(salary_band)   # per element
df["summary"] = df.apply(lambda row: f"{row['name']} ({row['dept']})", axis=1)  # per row

# --- Renaming and reindexing ---
df = df.rename(columns={"salary": "annual_salary", "years": "years_employed"})
df = df.rename(index={0: "emp_001", 1: "emp_002"})

# --- Type conversion ---
df["annual_salary"] = df["annual_salary"].astype(int)
df["hire_date"] = pd.to_datetime(df["hire_date"])  # parse date strings
df["dept"] = df["dept"].astype("category")         # memory-efficient for low-cardinality strings

# --- Missing data ---
# Check for missing values
print(df.isnull().sum())          # count NaN per column
print(df.isnull().any().any())    # True if ANY NaN anywhere

# Drop rows/cols with missing values
df_clean = df.dropna()                    # any NaN → drop row
df_clean = df.dropna(subset=["salary"])   # only if salary is NaN
df_clean = df.dropna(thresh=3)            # keep rows with at least 3 non-NaN

# Fill missing values
df["salary"] = df["salary"].fillna(df["salary"].median())
df["dept"] = df["dept"].fillna("Unknown")
df["salary"] = df["salary"].ffill()       # forward-fill (useful for time series)
df["salary"] = df["salary"].bfill()       # backward-fill
```

### Level 4: GroupBy and Aggregation

```python
# GroupBy is pandas's most powerful and most misunderstood feature.
# Think of it as "split → apply → combine":
# 1. Split the DataFrame into groups by the key column(s)
# 2. Apply a function to each group
# 3. Combine the results into a new DataFrame

sales = pd.DataFrame({
    "region": ["North", "North", "South", "South", "East", "East"],
    "product": ["A", "B", "A", "B", "A", "B"],
    "units": [120, 85, 95, 110, 70, 130],
    "revenue": [5999, 4249, 4749, 5499, 3499, 6499],
})

# Basic aggregation
print(sales.groupby("region")["revenue"].sum())
print(sales.groupby("region")["units"].mean())

# Multiple aggregations on one column
print(sales.groupby("region")["revenue"].agg(["sum", "mean", "max", "count"]))

# Multiple aggregations on multiple columns
result = sales.groupby("region").agg({
    "revenue": ["sum", "mean"],
    "units": "sum",
})
# Flatten multi-level column index
result.columns = ["_".join(col) for col in result.columns]

# Named aggregations (cleaner, Python 3.7+)
result = sales.groupby("region").agg(
    total_revenue=("revenue", "sum"),
    avg_revenue=("revenue", "mean"),
    total_units=("units", "sum"),
    num_products=("product", "count"),
)

# Group by multiple columns
print(sales.groupby(["region", "product"])["revenue"].sum())

# Transform — returns a Series aligned with the original DataFrame
# (group the data, but keep original shape)
sales["region_avg_revenue"] = sales.groupby("region")["revenue"].transform("mean")
sales["revenue_rank"] = sales.groupby("region")["revenue"].rank(ascending=False)

# Filter groups based on group-level aggregation
# Keep only regions with total revenue > 10000
high_revenue_regions = sales.groupby("region").filter(lambda g: g["revenue"].sum() > 10000)

# apply() for custom group operations
def top_product(group):
    return group.nlargest(1, "revenue")

top_products_per_region = sales.groupby("region").apply(top_product).reset_index(drop=True)
```

### Level 5: Merging, Reshaping, and Time Series

```python
# --- Merging DataFrames ---
orders = pd.DataFrame({
    "order_id": [1, 2, 3, 4],
    "customer_id": [101, 102, 101, 103],
    "amount": [250, 180, 320, 95],
})
customers = pd.DataFrame({
    "customer_id": [101, 102, 104],
    "name": ["Alice", "Bob", "Carol"],
    "city": ["Cairo", "Alex", "Giza"],
})

# pd.merge() — like SQL JOIN
inner  = pd.merge(orders, customers, on="customer_id", how="inner")   # matching rows only
left   = pd.merge(orders, customers, on="customer_id", how="left")    # all orders, NaN for missing customers
right  = pd.merge(orders, customers, on="customer_id", how="right")   # all customers, NaN for missing orders
outer  = pd.merge(orders, customers, on="customer_id", how="outer")   # all rows from both

# Merge on different column names
pd.merge(orders, customers, left_on="customer_id", right_on="id")

# concat — stack DataFrames vertically or horizontally
q1 = pd.DataFrame({"month": ["Jan", "Feb", "Mar"], "sales": [100, 120, 90]})
q2 = pd.DataFrame({"month": ["Apr", "May", "Jun"], "sales": [110, 130, 105]})
full_year = pd.concat([q1, q2], ignore_index=True)   # stack vertically

pd.concat([df1, df2], axis=1)   # stack horizontally (columns)

# --- Reshaping ---
wide = pd.DataFrame({
    "name": ["Alice", "Bob"],
    "jan_sales": [100, 120],
    "feb_sales": [110, 130],
    "mar_sales": [90, 115],
})

# Wide → Long (melt)
long = wide.melt(
    id_vars=["name"],
    value_vars=["jan_sales", "feb_sales", "mar_sales"],
    var_name="month",
    value_name="sales"
)

# Long → Wide (pivot)
pivot = long.pivot(index="name", columns="month", values="sales")

# Pivot table (like Excel pivot tables — handles duplicates via aggregation)
pivot_table = sales.pivot_table(
    values="revenue",
    index="region",
    columns="product",
    aggfunc="sum",
    fill_value=0,
)

# --- Time series ---
ts = pd.DataFrame({
    "date": pd.date_range("2024-01-01", periods=90, freq="D"),
    "sales": np.random.randint(50, 200, 90),
})
ts = ts.set_index("date")

# Resample — aggregate by time period
monthly = ts.resample("ME").sum()   # monthly totals (ME = month end)
weekly = ts.resample("W").mean()    # weekly averages

# Rolling windows — moving averages
ts["7d_rolling_avg"] = ts["sales"].rolling(7).mean()
ts["30d_rolling_avg"] = ts["sales"].rolling(30).mean()

# Date filtering
jan_sales = ts["2024-01"]            # all of January
q1_sales = ts["2024-01":"2024-03"]   # Q1 (loc-style inclusive)
```

### Level 6: Performance and Production Patterns

```python
import pandas as pd
import numpy as np
from pathlib import Path

# --- Performance: vectorization beats apply() beats loops ---

n = 1_000_000
df = pd.DataFrame({"x": np.random.randn(n), "y": np.random.randn(n)})

# SLOWEST: Python loop
# for i in range(len(df)):
#     df.at[i, "z"] = df.at[i, "x"] * 2 + df.at[i, "y"]

# SLOW: apply() (still Python overhead)
df["z_apply"] = df.apply(lambda row: row["x"] * 2 + row["y"], axis=1)

# FAST: vectorized NumPy operations (C-compiled)
df["z_vec"] = df["x"] * 2 + df["y"]

# For complex conditions, np.where and np.select beat apply:
df["category"] = np.where(df["x"] > 0, "positive", "negative")

# --- Memory optimization ---
df = pd.read_csv("large_file.csv")

# Check memory usage
print(df.memory_usage(deep=True).sum() / 1024**2, "MB")

# Optimize dtypes
df["id"] = df["id"].astype("int32")             # int32 instead of int64
df["score"] = df["score"].astype("float32")     # float32 instead of float64
df["category"] = df["category"].astype("category")  # string → categorical (huge savings for low-cardinality)

# Read only needed columns
df = pd.read_csv("large.csv", usecols=["id", "name", "score"])

# Read in chunks for files that don't fit in memory
chunk_list = []
for chunk in pd.read_csv("huge.csv", chunksize=100_000):
    processed = chunk[chunk["score"] > 0.5]   # process each chunk
    chunk_list.append(processed)
df = pd.concat(chunk_list, ignore_index=True)

# --- Production-quality data pipeline ---
def clean_sales_data(filepath: Path) -> pd.DataFrame:
    """Load, validate, and clean raw sales CSV."""
    # Load with type hints
    dtypes = {"order_id": "int32", "region": "category", "amount": "float32"}
    df = pd.read_csv(filepath, dtype=dtypes, parse_dates=["order_date"])

    # Validate expected columns
    required = {"order_id", "region", "amount", "order_date"}
    missing = required - set(df.columns)
    if missing:
        raise ValueError(f"Missing columns: {missing}")

    # Remove duplicates
    n_before = len(df)
    df = df.drop_duplicates(subset=["order_id"])
    print(f"Removed {n_before - len(df)} duplicate orders")

    # Handle missing values
    df["region"] = df["region"].fillna("Unknown")
    df = df.dropna(subset=["amount", "order_date"])

    # Remove outliers (amount > 99th percentile is likely an error)
    threshold = df["amount"].quantile(0.99)
    df = df[df["amount"] <= threshold]

    # Derived columns
    df["year_month"] = df["order_date"].dt.to_period("M")
    df["amount"] = df["amount"].clip(lower=0)   # no negative amounts

    return df.reset_index(drop=True)
```

---

## 4. Common Mistakes & Misconceptions

**Mistake 1: Using `.loc` and `.iloc` interchangeably**

```python
df = pd.DataFrame({"A": [10, 20, 30]}, index=[5, 10, 15])

# WRONG: iloc and loc return different rows when index isn't 0,1,2
print(df.loc[5])   # row with LABEL 5 → value 10
print(df.iloc[5])  # IndexError! only 3 rows

print(df.loc[10])  # row with label 10 → value 20
print(df.iloc[1])  # row at POSITION 1 → value 20
```

**Mistake 2: `SettingWithCopyWarning` — chained indexing**

```python
df = pd.DataFrame({"A": [1, 2, 3], "B": ["x", "y", "z"]})

# WRONG: chained indexing creates a copy, changes don't affect df
filtered = df[df["A"] > 1]
filtered["B"] = "updated"   # SettingWithCopyWarning — may not modify df!

# CORRECT: use .loc for assignment
df.loc[df["A"] > 1, "B"] = "updated"

# OR: assign to the copy explicitly if that's your intent
filtered = df[df["A"] > 1].copy()
filtered["B"] = "updated"   # no warning — we explicitly copied
```

**Mistake 3: Iterating with loops instead of vectorized operations**

```python
# WRONG: 100x+ slower for large DataFrames
for i, row in df.iterrows():
    df.at[i, "doubled"] = row["value"] * 2

# CORRECT: vectorized (C-speed)
df["doubled"] = df["value"] * 2

# If you must apply a complex function, prefer .apply() over iterrows
# and prefer vectorized numpy operations over .apply()
```

**Mistake 4: Not resetting index after filtering**

```python
df = pd.DataFrame({"A": [10, 20, 30, 40, 50]})
filtered = df[df["A"] > 20]
# filtered.index is now [2, 3, 4] — not [0, 1, 2]

# This causes issues with iloc, concat, and other operations that assume 0-based index
print(filtered.iloc[0])   # returns index 2 (position 0), value 30

# CORRECT: reset after filtering when you need 0-based index
filtered = filtered.reset_index(drop=True)
# drop=True: don't add old index as a column
```

**Mistake 5: Memory issues with large string columns**

```python
# String columns use Python object dtype — very memory-hungry
df = pd.read_csv("data.csv")
print(df["category"].dtype)   # object

# If "category" has few unique values (gender, status, country_code...):
df["category"] = df["category"].astype("category")  # can save 10-50x memory

# Check cardinality before converting:
print(df["category"].nunique())  # number of unique values
# Convert to category when unique_count << total_count
```

---

## 5. The "Why Does This Work" Layer

### How pandas Stores Data Internally

A DataFrame's columns are stored as individual numpy arrays (one per column, called "blocks"). When you select a column `df["salary"]`, you get a view into that array — no data is copied. This is why column-wise operations are fast: you're operating on a contiguous block of memory.

Row-wise operations with `.iterrows()` are slow because pandas must create a Python object for each row by collecting values from multiple different arrays. `.apply(axis=1)` has the same issue. Vectorized operations like `df["salary"] * 1.1` operate across one array at C speed.

The index is a separate data structure that maps labels to integer positions. `df.loc["Alice"]` does a hash lookup in the index to find the position, then accesses the underlying array. `df.iloc[0]` skips the hash lookup and accesses directly by position.

### Why GroupBy is Split-Apply-Combine

When you call `df.groupby("region")["revenue"].sum()`, pandas:
1. Splits the index into groups based on unique values in "region"
2. For each group, applies `sum()` to the "revenue" column
3. Combines the results into a new Series with the group labels as index

The "apply" step can be any function — built-in (`sum`, `mean`, `std`) or custom. Built-in aggregations are implemented in Cython (compiled C) — extremely fast. Custom `.apply()` functions run in Python and are slower.

`transform()` is like `apply()` but returns a Series aligned with the original DataFrame index — useful when you want to add a column that contains group-level statistics per row (e.g., the group mean).

---

## 6. Quick Reference

### Reading Data

```python
pd.read_csv("file.csv", usecols=["a","b"], dtype={"id": "int32"}, parse_dates=["date"])
pd.read_excel("file.xlsx", sheet_name="data")
pd.read_json("file.json")
pd.read_sql(query, connection)
pd.read_parquet("file.parquet")   # fastest for large data
```

### Selection

```python
df["col"]              # Series
df[["col1","col2"]]    # DataFrame
df.loc[label]          # row by label
df.loc[r_label, c_label]  # cell by labels
df.iloc[0]             # first row
df.iloc[0, 2]          # row 0, column 2

# Boolean
df[df["col"] > 5]
df[(cond1) & (cond2)]
df[df["col"].isin(values)]
df.query("col > 5 and other == 'x'")
```

### Essential Operations

```python
df.shape           # (rows, cols)
df.dtypes          # column types
df.describe()      # statistics
df.isnull().sum()  # missing per column
df.drop_duplicates()
df.dropna(subset=["col"])
df.fillna(value)
df.rename(columns={"old": "new"})
df.sort_values("col", ascending=False)
df.reset_index(drop=True)
```

### GroupBy Patterns

```python
# Single aggregation
df.groupby("key")["val"].sum()

# Named aggregations
df.groupby("key").agg(
    total=("val", "sum"),
    avg=("val", "mean"),
)

# Transform (keeps original shape)
df["group_mean"] = df.groupby("key")["val"].transform("mean")
```

### Merge Types

| SQL Equivalent | Pandas |
|---------------|--------|
| INNER JOIN | `pd.merge(a, b, how="inner")` |
| LEFT JOIN | `pd.merge(a, b, how="left")` |
| RIGHT JOIN | `pd.merge(a, b, how="right")` |
| FULL OUTER | `pd.merge(a, b, how="outer")` |
| Stack rows | `pd.concat([a, b])` |
| Stack cols | `pd.concat([a, b], axis=1)` |
