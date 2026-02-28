# Pandas Complete Reference Guide
> From Zero to Expert — Everything You Need to Master Pandas

---

## 1. What is Pandas & How It Works

Pandas is the primary data manipulation library for Python. It provides two core data structures — `Series` (1D labeled array) and `DataFrame` (2D labeled table) — and a vast toolkit for loading, cleaning, transforming, aggregating, and analyzing data.

### Pandas vs Plain Python/NumPy

```python
import pandas as pd
import numpy as np

# Without pandas: hard to work with structured data
data = [{"name": "Alice", "age": 30, "score": 95.5},
        {"name": "Bob",   "age": 25, "score": 87.2}]

# Selecting "age" column: [row["age"] for row in data]  → awkward
# Filtering: [r for r in data if r["age"] > 27]         → verbose
# Aggregating: sum(r["score"] for r in data) / len(data) → manual

# With pandas:
df = pd.DataFrame(data)
df["age"]                   # Column as Series
df[df["age"] > 27]          # Filter rows
df["score"].mean()          # Aggregate
df.groupby("age").mean()    # Group and aggregate
```

### The Mental Model

```
pandas Series   =  NumPy array  +  Index (labels)
pandas DataFrame = Dict of Series  (all sharing the same index)

Index is the key differentiator:
  - Enables label-based alignment
  - Makes joins/merges automatic
  - Allows duplicate row labels (unlike dicts)
  - Can be dates, strings, integers, MultiIndex, etc.
```

### Core Object Relationships

```
DataFrame
├── index    → row labels (Index object)
├── columns  → column labels (Index object)
└── values   → NumPy array (2D)

Each column → a Series
Series
├── index  → row labels
└── values → NumPy array (1D)
```

---

## 2. Installation & Setup

```bash
pip install pandas
pip install pandas numpy matplotlib seaborn  # Full analysis stack
pip install pandas[all]                       # All optional dependencies

# With conda
conda install pandas
conda install pandas numpy matplotlib seaborn openpyxl

# Check version
python -c "import pandas; print(pandas.__version__)"
```

```python
import pandas as pd
import numpy as np

# Display settings
pd.set_option("display.max_rows", 100)
pd.set_option("display.max_columns", 50)
pd.set_option("display.width", 120)
pd.set_option("display.float_format", "{:.2f}".format)
pd.set_option("display.max_colwidth", 50)

# Reset all options to defaults
pd.reset_option("all")

# Context manager for temporary settings
with pd.option_context("display.max_rows", 20, "display.max_columns", 10):
    print(df)

# Version info
pd.__version__
pd.show_versions()   # Full dependency versions
```

---

## 3. Series — The 1D Building Block

### Creating a Series

```python
# From a list
s = pd.Series([10, 20, 30, 40, 50])
# 0    10
# 1    20
# ...
# dtype: int64

# With explicit index (labels)
s = pd.Series([10, 20, 30], index=["a", "b", "c"])
# a    10
# b    20
# c    30

# From a dict (keys become index)
s = pd.Series({"alice": 95, "bob": 87, "carol": 92})

# From a scalar (broadcasts)
s = pd.Series(42, index=["a", "b", "c"])  # [42, 42, 42]

# From numpy array
s = pd.Series(np.arange(5), dtype="float64")

# Name the Series
s = pd.Series([1,2,3], name="scores")
s.name = "scores"
```

### Series Attributes & Methods

```python
s = pd.Series([10, 20, 30, 20, 10], index=["a","b","c","d","e"], name="vals")

# Attributes
s.index          # Index(['a','b','c','d','e'])
s.values         # array([10, 20, 30, 20, 10])  ← NumPy array
s.dtype          # dtype('int64')
s.shape          # (5,)
s.size           # 5
s.name           # 'vals'
s.ndim           # 1

# Preview
s.head(3)        # First 3
s.tail(2)        # Last 2

# Indexing
s["a"]           # 10
s[["a","c"]]     # Multiple labels
s["b":"d"]       # Slice by label (INCLUSIVE on both ends!)
s[0]             # 10  (positional — works but can be ambiguous)
s.iloc[0]        # 10  (positional — preferred)
s.loc["a"]       # 10  (label — preferred)

# Math (element-wise, aligned by index)
s + 100
s * 2
s ** 2
np.sqrt(s)

# Aggregations
s.sum()          # 90
s.mean()         # 18.0
s.median()       # 20.0
s.min(), s.max() # 10, 30
s.std()          # Sample std dev
s.var()
s.cumsum()       # Cumulative sum
s.cumprod()
s.describe()     # count, mean, std, min, 25%, 50%, 75%, max

# Boolean
s > 15           # Boolean Series
s[s > 15]        # [20, 30, 20]
s.between(15, 25)  # Boolean: 15 ≤ x ≤ 25

# Unique & value counts
s.unique()       # array([10, 20, 30])
s.nunique()      # 3
s.value_counts() # 10: 2, 20: 2, 30: 1 (sorted by count)
s.value_counts(normalize=True)  # Relative frequencies (proportions)

# Check membership
s.isin([10, 30])  # [T, F, T, F, T]

# Sorting
s.sort_values()
s.sort_values(ascending=False)
s.sort_index()

# Type conversion
s.astype(float)
s.astype(str)

# Alignment — Series automatically aligns on index
s1 = pd.Series([1, 2, 3], index=["a","b","c"])
s2 = pd.Series([10, 20, 30], index=["b","c","d"])
s1 + s2  # a: NaN, b: 12, c: 23, d: NaN  (missing → NaN)
s1.add(s2, fill_value=0)  # Fill missing before operation
```

---

## 4. DataFrame — The Core Object

### Creating a DataFrame

```python
# From dict of lists (most common)
df = pd.DataFrame({
    "name":  ["Alice", "Bob", "Carol", "Dave"],
    "age":   [30, 25, 35, 28],
    "score": [95.5, 87.2, 92.0, 88.8],
    "dept":  ["Eng", "Mkt", "Eng", "Mkt"],
})

# From list of dicts (each dict = one row)
df = pd.DataFrame([
    {"name": "Alice", "age": 30},
    {"name": "Bob",   "age": 25},
])

# From list of lists + column names
df = pd.DataFrame(
    [["Alice", 30], ["Bob", 25]],
    columns=["name", "age"]
)

# From NumPy array
arr = np.random.randn(4, 3)
df = pd.DataFrame(arr, columns=["A", "B", "C"])

# From dict of Series
df = pd.DataFrame({
    "sales": pd.Series([100, 200, 150], index=["Jan","Feb","Mar"]),
    "costs": pd.Series([80, 160, 120],  index=["Jan","Feb","Mar"]),
})

# With explicit index
df = pd.DataFrame(data, index=["r1","r2","r3","r4"])

# Empty DataFrame with schema
df = pd.DataFrame(columns=["name", "age", "score"])

# From CSV (most common in practice — see Section 5)
df = pd.read_csv("data.csv")
```

### DataFrame Attributes

```python
df.shape         # (4, 4) — (rows, columns)
df.ndim          # 2
df.size          # 16  (total elements)
df.index         # RangeIndex(start=0, stop=4, step=1)
df.columns       # Index(['name','age','score','dept'])
df.dtypes        # dtype of each column
df.values        # NumPy array of all values
df.to_numpy()    # Same, preferred
df.axes          # [index, columns]
df.info()        # Column names, dtypes, non-null counts, memory
df.memory_usage(deep=True)  # Memory per column in bytes
```

---

## 5. Reading & Writing Data

### CSV

```python
# Read CSV — most commonly used
df = pd.read_csv("data.csv")

# Full parameter reference
df = pd.read_csv(
    "data.csv",
    sep=",",                    # Delimiter (use sep="\t" for TSV)
    header=0,                   # Row to use as column names (0=first row)
    header=None,                # No header — columns get 0,1,2,...
    names=["a","b","c"],        # Custom column names
    index_col=0,                # Use column 0 as index
    index_col="id",             # Use named column as index
    usecols=["a","b"],          # Only load specific columns
    usecols=[0, 1, 3],          # By position
    dtype={"age": int, "score": float},  # Force dtypes
    dtype=str,                  # Everything as string
    nrows=1000,                 # Only read first 1000 rows
    skiprows=2,                 # Skip first 2 rows
    skiprows=[0, 2, 5],         # Skip specific row numbers
    skipfooter=3,               # Skip last 3 rows (requires engine="python")
    na_values=["N/A", "null", "--"],  # Extra strings to treat as NaN
    keep_default_na=True,       # Keep pandas default NaN strings
    true_values=["yes","Y"],    # Strings to treat as True
    false_values=["no","N"],    # Strings to treat as False
    parse_dates=["date"],       # Parse column as datetime
    parse_dates=[[0, 1]],       # Combine columns 0 and 1 into datetime
    date_format="%Y-%m-%d",     # Date format string
    dayfirst=True,              # DD/MM/YYYY format
    thousands=",",              # Thousand separator in numbers
    decimal=".",                # Decimal separator
    comment="#",                # Lines starting with # are comments
    encoding="utf-8",           # File encoding
    encoding="latin-1",         # For files with special chars
    engine="c",                 # 'c' (fast) or 'python' (more features)
    chunksize=10000,            # Read in chunks (returns iterator)
    compression="gzip",         # 'gzip', 'bz2', 'zip', 'xz', 'zstd'
    low_memory=False,           # Disable mixed-type inference
    on_bad_lines="warn",        # 'error', 'warn', 'skip' for bad lines
)

# Write CSV
df.to_csv("output.csv")
df.to_csv("output.csv",
    index=False,            # Don't write row index (usually what you want)
    sep=",",
    na_rep="",              # String for NaN values
    float_format="%.2f",    # Format floats
    columns=["a","b"],      # Only write specific columns
    header=True,
    encoding="utf-8",
    compression="gzip",
)

# Read large file in chunks
chunks = []
for chunk in pd.read_csv("huge.csv", chunksize=100_000):
    chunks.append(chunk[chunk["value"] > 0])
df = pd.concat(chunks, ignore_index=True)
```

### Excel

```python
# Read Excel
df = pd.read_excel("data.xlsx")
df = pd.read_excel("data.xlsx",
    sheet_name="Sheet1",     # By name
    sheet_name=0,            # By index
    sheet_name=["S1","S2"],  # Multiple sheets → dict of DataFrames
    sheet_name=None,         # All sheets → dict
    header=0,
    index_col=0,
    usecols="A:E",           # Excel column range
    usecols="A,C,F:H",
    nrows=500,
    skiprows=3,
    na_values=["N/A"],
    dtype={"id": str},
)

# Write Excel
df.to_excel("output.xlsx", index=False)
df.to_excel("output.xlsx", sheet_name="Results", index=False)

# Multiple sheets
with pd.ExcelWriter("output.xlsx", engine="openpyxl") as writer:
    df1.to_excel(writer, sheet_name="Sales",    index=False)
    df2.to_excel(writer, sheet_name="Expenses", index=False)

# Append to existing Excel
with pd.ExcelWriter("output.xlsx", engine="openpyxl", mode="a") as writer:
    df.to_excel(writer, sheet_name="NewSheet")
```

### Other Formats

```python
# JSON
df = pd.read_json("data.json")
df = pd.read_json("data.json", orient="records")  # List of dicts
df = pd.read_json("data.json", orient="split")    # {columns, index, data}
df.to_json("output.json", orient="records", indent=2)
df.to_json("output.json", orient="records", lines=True)  # JSONL / NDJSON

# Parquet (fast columnar format — best for large datasets)
df = pd.read_parquet("data.parquet")
df = pd.read_parquet("data.parquet", columns=["a","b"])  # Column pruning
df.to_parquet("output.parquet", index=False)
df.to_parquet("output.parquet", compression="snappy")    # 'snappy','gzip','brotli'

# Feather (extremely fast read/write)
df = pd.read_feather("data.feather")
df.to_feather("output.feather")

# SQL
import sqlite3
conn = sqlite3.connect("database.db")
df = pd.read_sql("SELECT * FROM users WHERE age > 25", conn)
df = pd.read_sql_table("users", conn)        # Entire table
df.to_sql("users", conn, if_exists="replace", index=False)
df.to_sql("users", conn, if_exists="append", index=False)
conn.close()

# SQLAlchemy (production-grade)
from sqlalchemy import create_engine
engine = create_engine("postgresql://user:pass@localhost/mydb")
df = pd.read_sql("SELECT * FROM orders", engine)
df.to_sql("results", engine, if_exists="replace")

# Pickle (preserve dtypes perfectly — not portable across Python versions)
df.to_pickle("data.pkl")
df = pd.read_pickle("data.pkl")

# HTML (reads tables from webpages or HTML strings)
dfs = pd.read_html("https://example.com/data.html")  # Returns list of DataFrames
df = pd.read_html("<table>...</table>")[0]

# Clipboard (copy/paste from spreadsheet)
df = pd.read_clipboard()
df.to_clipboard(index=False)

# From URL directly
df = pd.read_csv("https://example.com/data.csv")
```

---

## 6. Viewing & Inspecting Data

```python
df = pd.read_csv("data.csv")

# Preview
df.head()              # First 5 rows
df.head(10)            # First 10 rows
df.tail()              # Last 5 rows
df.tail(10)
df.sample(5)           # 5 random rows
df.sample(frac=0.1)    # 10% random sample
df.sample(5, random_state=42)  # Reproducible sample

# Structure
df.shape               # (rows, cols)
df.ndim                # 2
df.size                # rows × cols
df.columns             # Column names
df.index               # Row index
df.dtypes              # dtype per column
df.info()              # Concise summary: dtypes, non-null counts, memory

# Statistics
df.describe()                        # Count/mean/std/min/quartiles/max for numeric cols
df.describe(include="all")           # Include non-numeric cols too
df.describe(include="object")        # Only string cols
df.describe(include=[np.number])     # Only numeric cols
df.describe(percentiles=[0.1, 0.5, 0.9])  # Custom percentiles

# Per-column stats
df["age"].mean()
df["age"].median()
df["age"].std()
df["age"].min(), df["age"].max()
df["age"].value_counts()
df["age"].value_counts(normalize=True)  # Proportions
df["age"].value_counts(bins=5)          # Histogram-style bins
df["dept"].nunique()                    # Count of unique values
df["dept"].unique()                     # Array of unique values

# Missing data overview
df.isnull().sum()            # Count NaN per column
df.isnull().sum() / len(df)  # Proportion NaN per column
df.isnull().any()            # Any NaN per column?
df.notnull().all()           # All non-null per column?
df.isnull().sum().sum()      # Total NaN in DataFrame

# Duplicates
df.duplicated().sum()                  # Count duplicate rows
df.duplicated(subset=["name"]).sum()   # Duplicate by column subset
df[df.duplicated(keep=False)]          # Show all duplicates

# Correlations
df.corr()              # Pearson correlation matrix (numeric cols)
df.corr(method="spearman")
df.corr(method="kendall")
df["score"].corr(df["age"])    # Correlation between two columns
df.cov()               # Covariance matrix
```

---

## 7. Selection & Indexing

This is the most critical section — knowing when to use `[]`, `.loc`, `.iloc`, and `.at` is essential.

### Column Selection

```python
# Single column → Series
df["name"]
df.name                  # Attribute access (avoid — breaks if col name has spaces)

# Multiple columns → DataFrame
df[["name", "score"]]
cols = ["name", "score"]
df[cols]

# All columns except some
df.drop(columns=["id", "temp"])
df[[c for c in df.columns if c != "id"]]
```

### Row Selection

```python
# By index label — .loc (label-based, INCLUSIVE on both ends for slices)
df.loc[0]                    # Row with label 0 → Series
df.loc[[0, 2, 4]]            # Rows 0, 2, 4
df.loc[0:3]                  # Rows 0 through 3 INCLUSIVE (label-based!)
df.loc["alice"]              # Row labeled "alice"

# By position — .iloc (position-based, exclusive end like Python)
df.iloc[0]                   # First row → Series
df.iloc[-1]                  # Last row
df.iloc[[0, 2, 4]]           # Rows at positions 0, 2, 4
df.iloc[0:3]                 # Rows 0, 1, 2 (position 3 excluded)
df.iloc[-3:]                 # Last 3 rows
```

### Combined Row + Column Selection

```python
# .loc[row_selector, col_selector]
df.loc[0, "name"]             # Single value: row 0, col "name"
df.loc[0, ["name","score"]]   # Row 0, two columns
df.loc[:, "age"]              # All rows, column "age" → Series
df.loc[:, "age":"score"]      # All rows, columns from "age" to "score"
df.loc[0:2, "name":"score"]   # Rows 0-2, cols "name"-"score"
df.loc[df["age"] > 25, "name"]  # Filter rows, select column

# .iloc[row_pos, col_pos]
df.iloc[0, 1]                 # Row 0, column 1
df.iloc[:, 0]                 # All rows, first column
df.iloc[0:3, 0:2]             # First 3 rows, first 2 columns
df.iloc[[0,2], [1,3]]         # Specific row and column positions
df.iloc[-5:, :]               # Last 5 rows, all columns
```

### Fast Scalar Access

```python
# .at — single value by label (faster than .loc for scalars)
df.at[0, "name"]              # Get
df.at[0, "name"] = "Alicia"   # Set

# .iat — single value by position (faster than .iloc for scalars)
df.iat[0, 1]                  # Get
df.iat[0, 1] = 99             # Set
```

### The `[]` Shortcut — Know What It Does

```python
# DataFrame[]  with a string → column
df["name"]              # Series

# DataFrame[]  with a list → multiple columns
df[["name","age"]]      # DataFrame

# DataFrame[]  with a boolean Series → filter rows
df[df["age"] > 25]      # Filtered DataFrame

# DataFrame[]  with a slice → rows by POSITION (confusing!)
df[0:3]                 # First 3 rows (not recommended — use .iloc)
```

### Setting Values Safely (Avoid SettingWithCopyWarning)

```python
# ❌ WRONG — may or may not work, triggers warning
df[df["age"] > 25]["score"] = 100     # Modifies a copy!

# ✅ CORRECT — use .loc for assignment
df.loc[df["age"] > 25, "score"] = 100

# ✅ CORRECT — or use .copy() explicitly when subsetting
subset = df[df["age"] > 25].copy()
subset["score"] = 100   # Modifies copy intentionally

# Pandas 2.0+ Copy-on-Write (CoW) makes this cleaner:
# df[df["age"] > 25]["score"] = 100 always modifies copy in CoW mode
```

---

## 8. Filtering & Boolean Indexing

```python
df = pd.DataFrame({
    "name":  ["Alice","Bob","Carol","Dave","Eve"],
    "age":   [30, 25, 35, 28, 22],
    "score": [95, 87, 92, 88, 78],
    "dept":  ["Eng","Mkt","Eng","Mkt","Eng"],
})

# Single condition
df[df["age"] > 27]
df[df["dept"] == "Eng"]
df[df["name"] != "Bob"]

# Multiple conditions — use & | ~ with parentheses!
df[(df["age"] > 25) & (df["dept"] == "Eng")]    # AND
df[(df["age"] < 25) | (df["score"] > 90)]       # OR
df[~(df["dept"] == "Mkt")]                       # NOT

# .query() — string-based filtering (cleaner syntax)
df.query("age > 27")
df.query("age > 27 and dept == 'Eng'")
df.query("age > 27 or score > 90")
df.query("dept in ['Eng', 'Mkt']")
df.query("dept not in ['HR']")
df.query("age == age.mean()")          # Use column methods in query
threshold = 27                          # Reference local variables with @
df.query("age > @threshold")
df.query("score.between(85, 95)")      # Range check

# .isin() — match against a list
df[df["dept"].isin(["Eng", "Finance"])]
df[~df["dept"].isin(["Mkt"])]          # NOT in

# .between() — inclusive range
df[df["age"].between(25, 30)]          # 25 ≤ age ≤ 30
df[df["age"].between(25, 30, inclusive="left")]  # 25 ≤ age < 30

# .str methods for string filtering (see Section 13)
df[df["name"].str.startswith("A")]
df[df["name"].str.contains("li")]

# np.where for conditional column creation
df["grade"] = np.where(df["score"] >= 90, "A", "B")

# pd.cut and pd.qcut for binning
df["age_group"] = pd.cut(df["age"],
    bins=[0, 25, 30, 100],
    labels=["young", "mid", "senior"]
)
df["score_quartile"] = pd.qcut(df["score"], q=4,
    labels=["Q1","Q2","Q3","Q4"]
)

# Filter on NaN
df[df["score"].isna()]       # Rows where score is NaN
df[df["score"].notna()]      # Rows where score is not NaN
df.dropna(subset=["score"])  # Drop rows where score is NaN
```

---

## 9. Adding, Removing & Renaming

### Adding Columns

```python
# Simple assignment
df["bonus"] = df["score"] * 0.1
df["full_name"] = df["first"] + " " + df["last"]
df["is_senior"] = df["age"] >= 30

# .assign() — method chain friendly, returns new DataFrame
df = df.assign(
    bonus    = df["score"] * 0.1,
    category = lambda x: pd.cut(x["age"], bins=3, labels=["L","M","H"]),
    upper    = lambda x: x["name"].str.upper(),
)

# Insert at specific position
df.insert(2, "rank", df["score"].rank())  # Insert at column position 2

# From a condition
df["result"] = np.where(df["score"] >= 90, "Pass", "Fail")

# Multiple conditions with np.select
conditions = [df["score"] >= 90, df["score"] >= 75, df["score"] >= 60]
choices    = ["A",              "B",              "C"]
df["grade"] = np.select(conditions, choices, default="F")
```

### Removing Columns & Rows

```python
# Drop columns
df.drop(columns=["bonus"])                      # One column
df.drop(columns=["bonus", "rank"])              # Multiple
df.drop(["bonus"], axis=1)                      # Old style
df.drop(columns=["bonus"], inplace=True)        # Modify in place
df = df.drop(columns=["bonus"])                 # Preferred: reassign

# Drop rows by index label
df.drop(index=0)                                # Drop row with label 0
df.drop(index=[0, 2, 5])                        # Drop multiple rows
df.drop(index=df[df["score"] < 60].index)       # Drop by condition

# Pop — remove and return column (like dict.pop)
col = df.pop("bonus")    # Removes from df, returns as Series

# del — in-place delete
del df["bonus"]
```

### Renaming

```python
# Rename specific columns
df.rename(columns={"name": "full_name", "score": "test_score"})

# Rename with a function
df.rename(columns=str.lower)                    # All to lowercase
df.rename(columns=str.upper)
df.rename(columns=lambda x: x.replace(" ","_"))  # Spaces to underscores

# Rename index
df.rename(index={0: "first", 1: "second"})

# Reset index
df.reset_index()                  # Index becomes a column
df.reset_index(drop=True)         # Discard the old index
df.reset_index(inplace=True)

# Set index
df.set_index("name")              # Use 'name' column as index
df.set_index(["dept","name"])     # Multi-level index
df.set_index("id", drop=False)    # Keep column too (don't drop)

# Clean column names in one shot
df.columns = df.columns.str.strip().str.lower().str.replace(" ", "_")
```

---

## 10. Sorting & Ranking

```python
# Sort by values
df.sort_values("age")                          # Ascending (default)
df.sort_values("age", ascending=False)         # Descending
df.sort_values(["dept","score"])               # Multiple columns
df.sort_values(["dept","score"], ascending=[True, False])  # Mixed order
df.sort_values("name", na_position="last")    # NaN at end (default)
df.sort_values("name", na_position="first")   # NaN at start
df.sort_values("score", inplace=True)

# Sort by index
df.sort_index()
df.sort_index(ascending=False)
df.sort_index(axis=1)               # Sort columns alphabetically

# Ranking
df["rank"] = df["score"].rank()                    # Average rank for ties
df["rank"] = df["score"].rank(method="min")        # Ties get min rank
df["rank"] = df["score"].rank(method="max")        # Ties get max rank
df["rank"] = df["score"].rank(method="first")      # First occurrence wins
df["rank"] = df["score"].rank(method="dense")      # No gaps in rank
df["rank"] = df["score"].rank(ascending=False)     # Rank 1 = highest
df["rank"] = df["score"].rank(pct=True)            # 0-1 percentile rank

# nth largest / smallest
df.nlargest(3, "score")            # Top 3 by score (DataFrame method)
df.nsmallest(3, "score")           # Bottom 3
df.nlargest(3, ["score","age"])    # Break ties by age
df["score"].nlargest(3)            # Series method
```

---

## 11. Data Cleaning & Missing Data

### Detecting Missing Data

```python
df.isnull()                        # Boolean DataFrame
df.isna()                          # Same
df.notnull()
df.notna()

df.isnull().sum()                  # Count per column
df.isnull().mean()                 # Proportion per column
df.isnull().sum(axis=1)            # Count per row
df.isnull().any(axis=1)            # True if row has any NaN

# Heatmap of missing data (with seaborn)
import seaborn as sns
sns.heatmap(df.isnull(), cbar=False, yticklabels=False)
```

### Removing Missing Data

```python
# Drop rows with any NaN
df.dropna()
df.dropna(how="any")               # Same

# Drop rows only if ALL values are NaN
df.dropna(how="all")

# Drop only if NaN in specific columns
df.dropna(subset=["score", "age"])

# Drop columns with NaN
df.dropna(axis=1)
df.dropna(axis=1, how="all")

# Drop rows with fewer than n non-NaN values
df.dropna(thresh=3)                # Keep rows with at least 3 non-NaN
```

### Filling Missing Data

```python
# Fill all NaN with a single value
df.fillna(0)
df.fillna("Unknown")
df.fillna({"score": 0, "name": "Unknown"})  # Column-specific

# Forward fill — propagate last valid value forward
df.fillna(method="ffill")         # Old syntax
df.ffill()                         # New syntax (pandas 2.0+)

# Backward fill
df.fillna(method="bfill")
df.bfill()

# Fill with column stats
df["score"].fillna(df["score"].mean())
df["score"].fillna(df["score"].median())
df["score"].fillna(df.groupby("dept")["score"].transform("mean"))  # Group mean

# interpolate — fill by interpolation
df["score"].interpolate()                    # Linear interpolation
df["score"].interpolate(method="polynomial", order=2)
df["score"].interpolate(method="time")       # For time series

# Replace specific values
df.replace(0, np.nan)              # Replace 0 with NaN
df.replace([0, -1], np.nan)        # Replace multiple values
df.replace({"score": {0: np.nan, -1: np.nan}})  # Column-specific
df.replace({"N/A": np.nan, "null": np.nan})
df.replace(r"^\s+$", np.nan, regex=True)  # Replace whitespace-only strings
```

### Removing Duplicates

```python
df.drop_duplicates()                            # Drop completely duplicate rows
df.drop_duplicates(subset=["name"])             # Duplicate by column
df.drop_duplicates(subset=["name","dept"])      # Multiple columns
df.drop_duplicates(keep="first")                # Keep first (default)
df.drop_duplicates(keep="last")                 # Keep last
df.drop_duplicates(keep=False)                  # Drop ALL duplicates

df.duplicated()                                 # Boolean Series
df[df.duplicated(keep=False)]                   # See all duplicates
```

### Data Cleaning Patterns

```python
# Strip whitespace from strings
df["name"] = df["name"].str.strip()
for col in df.select_dtypes("object"):
    df[col] = df[col].str.strip()

# Standardize case
df["dept"] = df["dept"].str.lower()
df["dept"] = df["dept"].str.title()

# Fix encoding issues
df["text"] = df["text"].str.encode("latin-1").str.decode("utf-8", errors="replace")

# Remove special characters
df["phone"] = df["phone"].str.replace(r"\D", "", regex=True)  # Digits only

# Clip outliers
df["age"] = df["age"].clip(lower=0, upper=120)
q_lo  = df["score"].quantile(0.01)
q_hi  = df["score"].quantile(0.99)
df["score"] = df["score"].clip(q_lo, q_hi)

# Validate and report
print("Shape:", df.shape)
print("Duplicates:", df.duplicated().sum())
print("NaN per col:\n", df.isnull().sum())
print("Dtypes:\n", df.dtypes)
print("Unique per col:\n", df.nunique())
```

---

## 12. Data Types & Type Conversion

### Checking & Changing Types

```python
df.dtypes                              # dtype of each column
df.select_dtypes(include="number")     # Only numeric columns
df.select_dtypes(include="object")     # Only string/object columns
df.select_dtypes(include=["int64","float64"])
df.select_dtypes(exclude="datetime")   # All except datetime

# Convert a single column
df["age"]   = df["age"].astype(int)
df["score"] = df["score"].astype(float)
df["name"]  = df["name"].astype(str)
df["flag"]  = df["flag"].astype(bool)

# Convert multiple columns at once
df = df.astype({"age": int, "score": float})

# Safe conversion (returns NaN for unparseable values)
pd.to_numeric(df["age"])                   # NaN for errors
pd.to_numeric(df["age"], errors="coerce")  # NaN for non-numeric (safe)
pd.to_numeric(df["age"], errors="ignore")  # Keep original on error
pd.to_numeric(df["age"], errors="raise")   # Default: raise ValueError

# Downcasting (use smallest appropriate type)
pd.to_numeric(df["age"], downcast="integer")    # int8 if fits
pd.to_numeric(df["score"], downcast="float")    # float32

# Pandas dtypes (more memory-efficient than NumPy)
df["flag"] = df["flag"].astype("boolean")  # Nullable boolean (can have NA)
df["id"]   = df["id"].astype("Int64")      # Nullable integer (capital I!)
df["name"] = df["name"].astype("string")   # StringDtype (NA instead of NaN)
df["dept"] = df["dept"].astype("category") # CategoricalDtype (memory efficient)

# Infer better types automatically
df = df.convert_dtypes()   # Converts to best nullable types
df = df.infer_objects()    # Infer types for object columns
```

---

## 13. String Operations

All string operations are accessed via `Series.str` — they work on each element and preserve NaN.

```python
s = pd.Series(["Alice Smith", "  bob jones  ", "Carol O'Brien", np.nan])

# Case
s.str.lower()                   # alice smith, ...
s.str.upper()
s.str.title()                   # Alice Smith, ...
s.str.capitalize()
s.str.swapcase()

# Whitespace
s.str.strip()                   # Remove leading/trailing spaces
s.str.lstrip()
s.str.rstrip()
s.str.strip(".")                # Strip specific chars

# Padding
s.str.pad(10)                   # Right-align (left-pad with spaces)
s.str.pad(10, side="right")     # Left-align
s.str.center(20)
s.str.ljust(20)
s.str.rjust(20)
s.str.zfill(5)                  # Zero-pad: "42" → "00042"

# Search & detect
s.str.contains("Alice")         # Boolean Series
s.str.contains("alice", case=False)        # Case-insensitive
s.str.contains(r"\d+", regex=True)         # Regex
s.str.contains("Smith", na=False)          # NaN → False (not NaN)
s.str.startswith("Alice")
s.str.endswith("jones")
s.str.match(r"^[A-Z]")         # Regex match at start
s.str.fullmatch(r"[A-Z][a-z]+") # Full string must match

# Extraction
s.str.extract(r"(\w+)\s(\w+)")  # Capture groups → columns
s.str.extractall(r"(\d+)")      # All matches → multi-index Series
s.str.findall(r"\d+")           # List of all matches per element

# Replacement
s.str.replace("Smith", "Jones")
s.str.replace(r"\s+", "_", regex=True)   # Collapse spaces
s.str.replace(r"[^a-zA-Z\s]", "", regex=True)  # Remove non-alpha

# Split
s.str.split(" ")                 # Split on space → list per element
s.str.split(" ", expand=True)    # Split into separate columns
s.str.split(" ", n=1)            # Max 1 split
s.str.rsplit(".", n=1, expand=True)  # Split from right
s.str.partition(" ")             # Split at first occurrence: 3-col result
s.str.get(0)                     # Get 0th element of each list

# Slicing
s.str[0]                         # First character
s.str[:5]                        # First 5 characters
s.str[-3:]                       # Last 3 characters

# Length & counts
s.str.len()                      # Length of each string
s.str.count("l")                 # Count occurrences of substring
s.str.count(r"[aeiou]", flags=0) # Count regex matches

# Join / cat
s.str.cat(sep=", ")              # Concatenate all elements into one string
s.str.cat(other_series, sep="_") # Join two series element-wise

# Index operations
s.str.index("i")                 # Position of first "i" (raises if not found)
s.str.find("i")                  # Position of first "i" (-1 if not found)
s.str.rfind("i")                 # From right

# Encode / decode
s.str.encode("utf-8")
s.str.decode("utf-8")

# Wrap
s.str.wrap(20)                   # Insert newlines at word boundaries

# Normalize unicode
s.str.normalize("NFC")
```

---

## 14. DateTime Operations

```python
# Parse datetime on read
df = pd.read_csv("data.csv", parse_dates=["date"])
df = pd.read_csv("data.csv", parse_dates=["date"],
                 date_format="%Y-%m-%d")

# Convert column to datetime
df["date"] = pd.to_datetime(df["date"])
df["date"] = pd.to_datetime(df["date"], format="%d/%m/%Y")
df["date"] = pd.to_datetime(df["date"], errors="coerce")  # NaT for unparseable
df["date"] = pd.to_datetime(df[["year","month","day"]])   # From component columns

# Create datetime range
idx = pd.date_range("2024-01-01", periods=365, freq="D")   # Daily
idx = pd.date_range("2024-01-01", "2024-12-31", freq="D")
idx = pd.date_range("2024-01-01", periods=12, freq="ME")    # Month-end
idx = pd.date_range("2024-01-01", periods=52, freq="W")     # Weekly
idx = pd.date_range("2024-01-01", periods=24, freq="h")     # Hourly
idx = pd.date_range("2024-01-01", periods=8, freq="QE")     # Quarter-end

# Frequency aliases
# "D"=day, "W"=week, "ME"=month-end, "MS"=month-start,
# "QE"=quarter-end, "YE"=year-end, "h"=hour, "min"=minute, "s"=second
# "B"=business day, "BMS"=business month start

# .dt accessor — access datetime components
df["date"] = pd.to_datetime(df["date"])
df["date"].dt.year
df["date"].dt.month
df["date"].dt.day
df["date"].dt.hour
df["date"].dt.minute
df["date"].dt.second
df["date"].dt.weekday      # 0=Monday, 6=Sunday
df["date"].dt.day_name()   # "Monday", "Tuesday", etc.
df["date"].dt.month_name() # "January", etc.
df["date"].dt.quarter      # 1-4
df["date"].dt.week         # ISO week number (deprecated: use isocalendar)
df["date"].dt.isocalendar().week
df["date"].dt.is_month_end
df["date"].dt.is_month_start
df["date"].dt.is_year_end
df["date"].dt.is_leap_year
df["date"].dt.dayofyear    # Day of year (1-366)
df["date"].dt.days_in_month

# Truncate / round
df["date"].dt.floor("D")      # Round down to day
df["date"].dt.ceil("h")       # Round up to hour
df["date"].dt.round("min")    # Round to nearest minute

# Time deltas
df["end"]  = pd.to_datetime(df["end"])
df["start"]= pd.to_datetime(df["start"])
df["duration"] = df["end"] - df["start"]     # Timedelta Series
df["duration"].dt.days
df["duration"].dt.seconds
df["duration"].dt.total_seconds()

# Offset by timedelta
df["next_week"] = df["date"] + pd.Timedelta(days=7)
df["last_month"]= df["date"] - pd.DateOffset(months=1)
df["next_year"] = df["date"] + pd.DateOffset(years=1)

# Time zone handling
df["date"].dt.tz_localize("UTC")                      # Attach tz to naive datetime
df["date"].dt.tz_convert("America/New_York")           # Convert between timezones
df["date"].dt.tz_localize(None)                        # Remove timezone

# Filtering dates
df[df["date"] > "2024-06-01"]
df[df["date"].between("2024-01-01","2024-06-30")]
df.loc["2024-01-01":"2024-03-31"]   # Works if date is the index

# Resampling (time series aggregation — date must be index)
df = df.set_index("date")
df.resample("ME").sum()             # Monthly total
df.resample("QE").mean()            # Quarterly mean
df.resample("W").agg({"sales":"sum","visits":"mean"})
df.resample("D").ffill()            # Forward-fill missing days

# Period (fixed frequency periods)
df["period"] = df["date"].dt.to_period("M")   # Month period
df["period"] = df["date"].dt.to_period("Q")   # Quarter
period = pd.Period("2024-01", freq="M")
period.start_time   # Timestamp
period.end_time
```

---

## 15. Apply, Map & Transform

### `apply` — Most Flexible

```python
# Apply function to each column (axis=0, default)
df.apply(lambda col: col.max() - col.min())    # Range per column

# Apply function to each row (axis=1)
df.apply(lambda row: row["score"] * 1.1 if row["dept"] == "Eng" else row["score"],
         axis=1)

# Apply to Series
df["score"].apply(lambda x: x * 2)
df["name"].apply(len)                         # Length of each name
df["score"].apply(lambda x: "Pass" if x >= 60 else "Fail")

# With arguments
def scale(x, factor):
    return x * factor

df["score"].apply(scale, factor=1.1)
df.apply(scale, factor=1.1)               # Applied to each column

# apply returning multiple values (expands to columns)
def stats(col):
    return pd.Series({"mean": col.mean(), "std": col.std()})

df.apply(stats)    # Returns DataFrame with 'mean' and 'std' as columns
df["score"].apply(lambda x: pd.Series({"adj": x*1.1, "raw": x}))
```

### `map` — Element-wise on Series

```python
# map with dict (replaces values)
mapping = {"Eng": "Engineering", "Mkt": "Marketing"}
df["dept"].map(mapping)

# map with function
df["score"].map(lambda x: round(x, 0))

# map leaves NaN for unmapped values (unlike replace)
# If key not in dict → NaN

# For DataFrames, use applymap (old) or map (pandas 2.1+)
df.map(lambda x: x**2)                  # Element-wise (numeric cols)
df.applymap(lambda x: round(x,2))       # Old name (still works)
```

### `transform` — Same Shape as Input

```python
# transform always returns same shape as input — great for groupby
df.groupby("dept")["score"].transform("mean")   # Each row gets its group mean
df.groupby("dept")["score"].transform(lambda x: (x - x.mean()) / x.std())

# Standardize within groups
df["score_zscore"] = df.groupby("dept")["score"].transform(
    lambda x: (x - x.mean()) / x.std()
)

# Add group statistics as new column (without changing original shape)
df["dept_mean_score"] = df.groupby("dept")["score"].transform("mean")
df["dept_rank"]       = df.groupby("dept")["score"].transform("rank")
df["dept_pct"]        = df.groupby("dept")["score"].transform(lambda x: x / x.sum())
```

### Vectorized Alternatives (Faster)

```python
# SLOW: apply with a simple expression
df["score"].apply(lambda x: x * 2)

# FAST: direct vectorized operation
df["score"] * 2

# SLOW: apply for string length
df["name"].apply(len)

# FAST: .str accessor
df["name"].str.len()

# SLOW: apply for conditional
df["score"].apply(lambda x: "A" if x >= 90 else "B" if x >= 75 else "C")

# FAST: np.select
conditions = [df["score"] >= 90, df["score"] >= 75]
choices    = ["A", "B"]
np.select(conditions, choices, default="C")

# Rule: only use apply when no vectorized alternative exists
```

---

## 16. Groupby & Aggregation

The split-apply-combine pattern: split DataFrame into groups, apply a function to each group, combine results.

### Basic Groupby

```python
grouped = df.groupby("dept")

# Single aggregation
grouped["score"].mean()          # Mean score per dept
grouped["score"].sum()
grouped["score"].count()
grouped["score"].min()
grouped["score"].max()
grouped["score"].std()
grouped["score"].var()
grouped["score"].median()
grouped["score"].nunique()
grouped["score"].first()         # First value in group
grouped["score"].last()
grouped["score"].nth(0)          # nth element (0=first, -1=last)

# Multiple columns
grouped[["score","age"]].mean()

# Multiple groups
df.groupby(["dept","level"])["score"].mean()
df.groupby(["dept","level"])["score"].mean().reset_index()  # Flatten multi-index

# Group and count rows
df.groupby("dept").size()            # Count rows per group (includes NaN)
df.groupby("dept")["score"].count()  # Count non-NaN per group
```

### `.agg()` — Multiple Aggregations

```python
# Multiple functions on one column
df.groupby("dept")["score"].agg(["mean","std","min","max","count"])

# Different functions per column
df.groupby("dept").agg({
    "score": ["mean", "std"],
    "age":   ["min", "max"],
    "name":  "count",
})

# Named aggregations (pandas 0.25+) — cleaner column names
df.groupby("dept").agg(
    avg_score  = ("score", "mean"),
    max_score  = ("score", "max"),
    min_age    = ("age",   "min"),
    count      = ("name",  "count"),
)

# Custom function
df.groupby("dept")["score"].agg(lambda x: x.max() - x.min())
df.groupby("dept").agg(
    iqr = ("score", lambda x: x.quantile(0.75) - x.quantile(0.25))
)

# Multiple custom functions
def pct_above_90(x):
    return (x > 90).sum() / len(x)

df.groupby("dept")["score"].agg([np.mean, np.std, pct_above_90])
```

### `.transform()` vs `.agg()`

```python
# agg: one row per group (reduces)
df.groupby("dept")["score"].mean()             # 3 rows (one per dept)

# transform: same number of rows as original (broadcasts back)
df.groupby("dept")["score"].transform("mean")  # Same rows as df
# Use to add group stats as a column in the original DataFrame

df["dept_mean"]  = df.groupby("dept")["score"].transform("mean")
df["dept_rank"]  = df.groupby("dept")["score"].transform("rank")
df["dept_share"] = df.groupby("dept")["score"].transform(lambda x: x/x.sum())
```

### `.filter()` — Keep/Drop Entire Groups

```python
# Keep only groups where mean score > 85
df.groupby("dept").filter(lambda g: g["score"].mean() > 85)

# Keep groups with at least 10 members
df.groupby("dept").filter(lambda g: len(g) >= 10)
```

### `.apply()` on Groups — Most Flexible

```python
# Apply custom function to each group (returns whatever the function returns)
def top_scorers(group, n=2):
    return group.nlargest(n, "score")

df.groupby("dept").apply(top_scorers, n=2)
df.groupby("dept").apply(top_scorers, n=2, include_groups=False)  # pandas 2.2+

# Compute within-group z-score
def z_score(group):
    group = group.copy()
    group["z"] = (group["score"] - group["score"].mean()) / group["score"].std()
    return group

df.groupby("dept").apply(z_score).reset_index(drop=True)
```

### Groupby Options

```python
# as_index=False — get flat DataFrame instead of group labels as index
df.groupby("dept", as_index=False)["score"].mean()

# observed=True — for CategoricalIndex, show only observed categories
df.groupby("dept", observed=True)["score"].mean()

# dropna=False — include NaN as a group
df.groupby("dept", dropna=False)["score"].mean()

# sort=False — preserve group order of first occurrence
df.groupby("dept", sort=False)["score"].mean()

# Groupby with pd.Grouper (for time-based grouping)
df.set_index("date").groupby(pd.Grouper(freq="ME"))["sales"].sum()
df.groupby([pd.Grouper(key="date", freq="QE"), "dept"])["sales"].sum()
```

---

## 17. Merging, Joining & Combining

### `pd.merge()` — SQL-Style Joins

```python
left  = pd.DataFrame({"id": [1,2,3,4], "name": ["A","B","C","D"]})
right = pd.DataFrame({"id": [2,3,5],   "score":[90,85,88]})

# Inner join (only matching rows)
pd.merge(left, right, on="id")                      # on="id"
pd.merge(left, right, on=["id","dept"])              # Multiple keys

# Left join (all left rows, NaN for unmatched right)
pd.merge(left, right, on="id", how="left")

# Right join (all right rows, NaN for unmatched left)
pd.merge(left, right, on="id", how="right")

# Full outer join (all rows, NaN for unmatched)
pd.merge(left, right, on="id", how="outer")

# When key columns have different names
pd.merge(left, right, left_on="user_id", right_on="id")

# Merge on index
pd.merge(left, right, left_index=True, right_index=True)
pd.merge(left, right, left_on="id", right_index=True)

# Suffixes for duplicate column names
pd.merge(left, right, on="id", suffixes=("_left","_right"))

# Validate — check join type
pd.merge(left, right, on="id", validate="one_to_one")    # 1:1
pd.merge(left, right, on="id", validate="one_to_many")   # 1:M
pd.merge(left, right, on="id", validate="many_to_one")   # M:1

# indicator — mark source
pd.merge(left, right, on="id", how="outer", indicator=True)
# _merge: "left_only", "right_only", "both"

# Method version
left.merge(right, on="id")
left.merge(right, on="id", how="left")
```

### `DataFrame.join()` — Join on Index

```python
# join() always merges on index (by default)
left.join(right)                     # Inner join on index
left.join(right, how="left")
left.join(right, lsuffix="_l", rsuffix="_r")

# Join on a column + index
left.set_index("id").join(right.set_index("id"), how="left")
```

### `pd.concat()` — Stack DataFrames

```python
# Stack rows (axis=0, default)
pd.concat([df1, df2])                         # Stack vertically
pd.concat([df1, df2], ignore_index=True)      # Reset index
pd.concat([df1, df2], keys=["df1","df2"])     # Multi-index with source label

# Stack columns (axis=1)
pd.concat([df1, df2], axis=1)                 # Side by side

# Alignment — concat aligns on index and fills NaN for missing
pd.concat([df1, df2], join="inner")           # Only common columns
pd.concat([df1, df2], join="outer")           # All columns (default)

# Append a single row (dict)
new_row = {"name": "Eve", "age": 28, "score": 91}
pd.concat([df, pd.DataFrame([new_row])], ignore_index=True)
```

### `pd.merge_asof()` — Approximate/Nearest Join

```python
# Merge on nearest key (must be sorted)
# Useful for: time-series, range joins
left  = pd.DataFrame({"time": [1, 5, 10], "event": ["A","B","C"]})
right = pd.DataFrame({"time": [3, 7, 15], "price": [100, 200, 300]})

pd.merge_asof(left, right, on="time")           # Backward (last ≤ left)
pd.merge_asof(left, right, on="time", direction="forward")  # First ≥ left
pd.merge_asof(left, right, on="time", direction="nearest")  # Closest

# With tolerance
pd.merge_asof(left, right, on="time", tolerance=2)  # Only match within 2 units
```

### `pd.merge_ordered()` — Ordered Merge

```python
pd.merge_ordered(left, right, on="date", fill_method="ffill")
```

### Updating / Combining Data

```python
# update — update one DataFrame with another's non-NaN values
df1.update(df2)               # In-place: df2 values overwrite df1 where non-NaN

# combine_first — fill NaN in df1 with df2's values
result = df1.combine_first(df2)     # Like a null-safe "df1 COALESCE df2"

# combine — element-wise combination with a function
df1.combine(df2, lambda s1, s2: s1.where(s1 > s2, s2))  # Element-wise max
```

---

## 18. Reshaping — Pivot, Melt, Stack

### `pivot_table` — Aggregate and Reshape

```python
df = pd.DataFrame({
    "date":  ["2024-01","2024-01","2024-02","2024-02"],
    "dept":  ["Eng","Mkt","Eng","Mkt"],
    "sales": [100, 200, 150, 180],
    "costs": [80,  160, 120, 144],
})

# pivot_table: rows=date, cols=dept, values=sales
df.pivot_table(values="sales", index="date", columns="dept")
#        dept  Eng  Mkt
# date
# 2024-01  100  200
# 2024-02  150  180

# With aggregation function
df.pivot_table(values="sales", index="date", columns="dept", aggfunc="sum")
df.pivot_table(values="sales", index="date", columns="dept", aggfunc="mean")
df.pivot_table(values="sales", index="date", columns="dept", aggfunc=["sum","mean"])

# Multiple value columns
df.pivot_table(values=["sales","costs"], index="date", columns="dept", aggfunc="sum")

# Margins (grand totals)
df.pivot_table(values="sales", index="date", columns="dept",
               aggfunc="sum", margins=True, margins_name="Total")

# Fill missing values
df.pivot_table(values="sales", index="date", columns="dept", fill_value=0)
```

### `pivot` — No Aggregation (Unique key required)

```python
# pivot: reshape without aggregation — requires unique index/column combos
df.pivot(index="date", columns="dept", values="sales")
```

### `melt` — Wide to Long

```python
wide = pd.DataFrame({
    "name":  ["Alice","Bob"],
    "math":  [90, 85],
    "english": [88, 92],
    "science": [95, 78],
})

# Melt: convert subject columns into rows
long = pd.melt(wide,
    id_vars=["name"],               # Columns to keep as-is
    value_vars=["math","english","science"],  # Columns to unpivot
    var_name="subject",             # Name for the "variable" column
    value_name="score",             # Name for the "value" column
)
# name  subject  score
# Alice  math     90
# Alice  english  88
# ...

# Melt all non-id columns
pd.melt(wide, id_vars=["name"])
```

### `stack` / `unstack` — Index Level Pivoting

```python
# stack: move column level into row index (wide → long)
df.stack()               # Last column level → innermost row level
df.stack(level=0)        # First column level

# unstack: move row index level into columns (long → wide)
df.unstack()             # Innermost row level → last column level
df.unstack(level=0)      # First row level → columns
df.unstack("dept")       # By name

# Often used after groupby to reshape multi-index results
result = df.groupby(["date","dept"])["sales"].sum()
result.unstack("dept")   # Pivot dept to columns
```

### `pd.crosstab` — Frequency Tables

```python
# Counts of combinations
pd.crosstab(df["dept"], df["grade"])

# Proportions
pd.crosstab(df["dept"], df["grade"], normalize=True)       # All cells
pd.crosstab(df["dept"], df["grade"], normalize="index")    # Row proportions
pd.crosstab(df["dept"], df["grade"], normalize="columns")  # Col proportions

# With aggregation
pd.crosstab(df["dept"], df["grade"], values=df["score"], aggfunc="mean")

# Margins
pd.crosstab(df["dept"], df["grade"], margins=True)
```

### `pd.wide_to_long` — Structured Wide to Long

```python
# For DataFrames with column name patterns like "math_2022","math_2023"
wide = pd.DataFrame({
    "id":        [1, 2],
    "score_2022":[90, 85],
    "score_2023":[92, 88],
    "rank_2022": [1, 2],
    "rank_2023": [1, 3],
})

pd.wide_to_long(wide, stubnames=["score","rank"], i="id", j="year", sep="_")
```

---

## 19. Window Functions

### Rolling Windows

```python
s = pd.Series([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])

# rolling(n) — fixed window of n elements
s.rolling(3).mean()             # 3-period moving average
s.rolling(3).sum()
s.rolling(3).std()
s.rolling(3).min()
s.rolling(3).max()
s.rolling(3).median()
s.rolling(3).var()
s.rolling(3).skew()
s.rolling(3).kurt()
s.rolling(3).apply(lambda x: x[0] + x[-1])   # Custom function

# min_periods — require fewer observations (useful at start)
s.rolling(3, min_periods=1).mean()   # Use available data at edges

# center=True — window centered on current element
s.rolling(3, center=True).mean()

# On DataFrame
df.rolling(7).mean()             # 7-day rolling mean for all numeric columns
df["score"].rolling(7).mean()

# Time-based rolling (index must be DatetimeIndex)
df.set_index("date").rolling("7D").mean()   # 7 calendar days
df.set_index("date").rolling("30D").sum()
```

### Expanding Windows

```python
# expanding — window grows from start to current position
s.expanding().mean()            # Running mean (cumulative)
s.expanding().sum()             # Running sum (same as cumsum)
s.expanding().max()             # Running maximum
s.expanding().std()             # Running std dev

s.expanding(min_periods=5).mean()  # Wait for 5 observations before computing
```

### Exponential Weighted (EWM)

```python
# ewm — exponentially weighted, recent values get more weight
s.ewm(span=10).mean()           # Span-based (similar to rolling(10))
s.ewm(halflife=5).mean()        # Half-life: weight halves every 5 steps
s.ewm(alpha=0.1).mean()         # Alpha: smoothing factor (0 < α ≤ 1)
s.ewm(com=9).mean()             # Center of mass

# Common use: smoothing time series data
df["smooth"] = df["sales"].ewm(span=7).mean()
```

### Shift & Diff

```python
# shift — lag/lead values
s.shift(1)                      # Lag by 1 (forward fill by 1)
s.shift(-1)                     # Lead by 1
s.shift(1, fill_value=0)        # Fill boundary with 0

# pct_change — percent change from previous row
s.pct_change()                  # (x[t] - x[t-1]) / x[t-1]
s.pct_change(periods=3)         # vs 3 periods ago
s.pct_change(fill_method=None)  # Don't fill NaN before computing

# diff — absolute difference
s.diff()                        # x[t] - x[t-1]
s.diff(periods=3)               # x[t] - x[t-3]
s.diff(axis=1)                  # Along columns (for DataFrame)

# Practical: calculate returns, find changes
df["returns"] = df["price"].pct_change()
df["daily_change"] = df["price"].diff()
df["yoy_growth"] = df["revenue"].pct_change(periods=12)  # Year-over-year for monthly data
```

---

## 20. Multi-Index (Hierarchical Index)

### Creating Multi-Index

```python
# From groupby
result = df.groupby(["dept","level"])["score"].mean()
# dept    level
# Eng     Junior    82.5
#         Senior    91.0
# Mkt     Junior    77.0
# dtype: float64

# Explicit MultiIndex
idx = pd.MultiIndex.from_tuples([("Eng","Junior"),("Eng","Senior"),("Mkt","Junior")])
idx = pd.MultiIndex.from_product([["Eng","Mkt"], ["Junior","Senior"]])
idx = pd.MultiIndex.from_arrays([["Eng","Eng","Mkt"], ["Junior","Senior","Junior"]])

df_multi = pd.DataFrame({"score": [82.5, 91.0, 77.0]}, index=idx)
df_multi.index.names = ["dept", "level"]
```

### Selecting from Multi-Index

```python
# .loc with multi-index
df_multi.loc["Eng"]                          # All rows in "Eng"
df_multi.loc[("Eng","Senior")]               # Specific combination
df_multi.loc[["Eng","Mkt"]]                  # Multiple top-level keys

# Cross-section with xs()
df_multi.xs("Senior", level="level")          # All "Senior" rows
df_multi.xs(("Eng","Senior"))                 # Same as .loc[("Eng","Senior")]

# IndexSlice for slicing
idx = pd.IndexSlice
df_multi.loc[idx["Eng", :], :]               # All Eng, all columns
df_multi.loc[idx[:, "Senior"], :]            # All Senior, all columns

# query works with multi-index names
df_multi.query("dept == 'Eng' and level == 'Senior'")
```

### Manipulating Multi-Index

```python
# Flatten multi-index columns (after pivot_table)
df.columns = ["_".join(col).strip() for col in df.columns.values]

# reset_index — turn index levels into columns
df_multi.reset_index()
df_multi.reset_index(level=0)     # Only outer level

# stack / unstack
df_multi.unstack("level")         # Move "level" index to columns
df_multi.unstack()                # Move innermost index to columns

# swaplevel — swap index levels
df_multi.swaplevel()
df_multi.swaplevel("dept","level")

# sort_index — required after many operations
df_multi.sort_index()
df_multi.sort_index(level="dept")
df_multi.sort_index(level=1, ascending=False)

# droplevel
df_multi.droplevel("level")      # Drop one level from index
```

---

## 21. Categorical Data

Categorical dtype saves memory when a column has limited unique values (e.g., departments, status, country codes).

```python
# Convert to categorical
df["dept"] = df["dept"].astype("category")
df["dept"] = pd.Categorical(df["dept"])
df["status"] = pd.Categorical(["active","inactive","active"],
                               categories=["inactive","active"],
                               ordered=True)

# Category info
df["dept"].cat.categories         # Index of categories
df["dept"].cat.codes              # Integer codes per element (memory is stored here)
df["dept"].cat.ordered            # True/False

# Modify categories
df["dept"].cat.add_categories(["Finance"])
df["dept"].cat.remove_categories(["Finance"])
df["dept"].cat.rename_categories({"Eng": "Engineering"})
df["dept"].cat.set_categories(["Eng","Mkt","HR"])
df["dept"].cat.remove_unused_categories()
df["dept"].cat.reorder_categories(["Mkt","Eng"], ordered=True)

# Ordered categorical — enables comparison
sizes = pd.Categorical(["S","M","L","XL"], ordered=True,
                        categories=["S","M","L","XL"])
sizes > "M"   # [False, False, True, True]

# Memory comparison
s_str = pd.Series(["Eng","Mkt","Eng","Mkt"] * 1_000_000)
s_cat = s_str.astype("category")
s_str.memory_usage(deep=True)   # ~60 MB
s_cat.memory_usage(deep=True)   # ~4 MB   → 15x savings

# Groupby with categorical preserves all categories (even empty ones)
df.groupby("dept", observed=False)["score"].mean()  # Shows all cats, even 0-count
df.groupby("dept", observed=True)["score"].mean()   # Only observed categories
```

---

## 22. Performance & Best Practices

### Profiling & Benchmarking

```python
import timeit

# Time a single operation
%timeit df[df["age"] > 25]            # Jupyter magic (1000 runs)
timeit.timeit(lambda: df[df["age"] > 25], number=1000)

# Memory
df.memory_usage(deep=True).sum()      # Bytes used
df.memory_usage(deep=True) / 1e6      # MB per column

import sys
sys.getsizeof(df)
```

### Optimize dtypes (Biggest Win)

```python
def optimize_dtypes(df):
    """Reduce DataFrame memory by downcasting numeric types."""
    for col in df.select_dtypes("integer"):
        df[col] = pd.to_numeric(df[col], downcast="integer")
    for col in df.select_dtypes("float"):
        df[col] = pd.to_numeric(df[col], downcast="float")
    for col in df.select_dtypes("object"):
        if df[col].nunique() / len(df) < 0.5:  # Less than 50% unique values
            df[col] = df[col].astype("category")
    return df

# Typical savings: 50-80% memory reduction
df = optimize_dtypes(df.copy())
```

### Avoid Common Slow Patterns

```python
# ❌ SLOW: iterrows (Python loop, creates Series per row)
for idx, row in df.iterrows():
    df.at[idx, "result"] = row["a"] + row["b"]

# ✅ FAST: vectorized operation
df["result"] = df["a"] + df["b"]

# ❌ SLOW: itertuples (faster than iterrows, but still Python loop)
for row in df.itertuples():
    ...

# ✅ FAST: apply for complex logic, but vectorize where possible
df["result"] = df.apply(lambda r: complex_fn(r["a"], r["b"]), axis=1)

# ❌ SLOW: Growing DataFrame with concat in a loop
results = []
for i in range(1000):
    results.append(pd.DataFrame([compute(i)]))
df = pd.concat(results)  # This is O(n²)!

# ✅ FAST: collect dicts, build DataFrame once
results = [compute(i) for i in range(1000)]
df = pd.DataFrame(results)

# ❌ SLOW: chained indexing
df["col1"][df["col2"] > 5] = 10   # Ambiguous, may modify copy

# ✅ FAST + CORRECT: use .loc
df.loc[df["col2"] > 5, "col1"] = 10

# ❌ SLOW: .apply(lambda x: x.split(","))
df["name"].apply(lambda x: x.split(","))

# ✅ FAST: .str accessor
df["name"].str.split(",")

# ❌ SLOW: filling NaN in a loop
for col in df.columns:
    df[col] = df[col].fillna(df[col].mean())

# ✅ FAST: bulk fill with dict
fill_values = {col: df[col].mean() for col in df.select_dtypes("number")}
df.fillna(fill_values, inplace=True)
```

### Use `eval()` and `query()` for Large DataFrames

```python
# pd.eval — evaluate expression without intermediate allocations
# Fast for large DataFrames (uses numexpr under the hood)
df["result"] = pd.eval("df.a + df.b - df.c * 2")
df.eval("result = a + b - c * 2", inplace=True)
df.eval("a > 0 and b < 100")

# query — faster filtering for large DataFrames
df.query("age > 25 and dept == 'Eng'")           # Uses numexpr
df.query("score.between(80, 95)", engine="python") # Pure Python engine
```

### Chunked Processing for Large Files

```python
# Process CSV in chunks to handle files larger than RAM
total_sales = 0
for chunk in pd.read_csv("huge_file.csv", chunksize=100_000):
    chunk = chunk[chunk["status"] == "completed"]
    total_sales += chunk["amount"].sum()

# Aggregate across chunks
results = []
for chunk in pd.read_csv("huge_file.csv", chunksize=100_000):
    agg = chunk.groupby("dept")["sales"].agg(["sum","count"])
    results.append(agg)
pd.concat(results).groupby(level=0).sum()  # Combine chunk results

# Use Parquet for better performance
df.to_parquet("data.parquet")                    # Column-oriented, compressed
df = pd.read_parquet("data.parquet",
                     columns=["a","b"])           # Column pruning — only reads what you need
```

### Method Chaining (Clean, Modern Style)

```python
# Instead of mutating df multiple times:
df = pd.read_csv("sales.csv")
df = df.rename(columns=str.lower)
df["date"] = pd.to_datetime(df["date"])
df = df[df["status"] == "completed"]
df = df.dropna(subset=["amount"])
df = df.sort_values("date")

# Use method chaining for clarity
df = (
    pd.read_csv("sales.csv")
    .rename(columns=str.lower)
    .assign(date=lambda x: pd.to_datetime(x["date"]))
    .query("status == 'completed'")
    .dropna(subset=["amount"])
    .sort_values("date")
    .reset_index(drop=True)
)

# Pipe for custom functions in chains
def add_features(df):
    return df.assign(
        year      = df["date"].dt.year,
        month     = df["date"].dt.month,
        is_high   = df["amount"] > df["amount"].median(),
    )

df = (
    pd.read_csv("sales.csv")
    .rename(columns=str.lower)
    .assign(date=lambda x: pd.to_datetime(x["date"]))
    .pipe(add_features)
    .groupby(["year","month"])["amount"]
    .sum()
    .reset_index()
)
```

---

## 23. Quick Reference Card

### Essential Operations at a Glance

```python
# Load & save
df = pd.read_csv("f.csv")
df = pd.read_excel("f.xlsx")
df.to_csv("f.csv", index=False)
df.to_parquet("f.parquet")

# Explore
df.head()
df.info()
df.describe()
df.shape
df.dtypes
df.isnull().sum()

# Select
df["col"]                         # Column (Series)
df[["a","b"]]                     # Columns (DataFrame)
df.loc[row_label, col_label]      # Label-based
df.iloc[row_int, col_int]         # Position-based
df[df["col"] > 0]                 # Filter rows
df.query("col > 0 and col2 == 'x'")

# Modify
df["new"] = df["a"] + df["b"]               # New column
df.loc[cond, "col"] = value                  # Conditional assign
df.drop(columns=["col"])                     # Remove column
df.rename(columns={"old":"new"})             # Rename
df.drop_duplicates()                         # Deduplicate
df.fillna(0)                                 # Fill NaN
df.dropna(subset=["col"])                    # Remove NaN rows

# Transform
df.sort_values("col", ascending=False)
df.groupby("col")["val"].mean()
df.pivot_table(values="val", index="r", columns="c")
pd.melt(df, id_vars=["id"], value_vars=["a","b"])
pd.merge(left, right, on="id", how="left")
pd.concat([df1, df2], ignore_index=True)

# Rolling/Window
df["col"].rolling(7).mean()
df["col"].expanding().sum()
df["col"].shift(1)
df["col"].pct_change()
```

### `.loc` vs `.iloc` vs `[]` Decision Tree

```
Do you want a column?
  └─► df["col"]  or  df[["col1","col2"]]

Do you want to filter rows by condition?
  └─► df[df["col"] > 0]  or  df.query("col > 0")

Do you know the LABEL (index value)?
  └─► df.loc[label]  or  df.loc[label, "col"]
      Remember: slices with .loc are INCLUSIVE at both ends

Do you know the POSITION (0, 1, 2...)?
  └─► df.iloc[position]  or  df.iloc[0:5, 2:4]
      Slices with .iloc are exclusive at end (like Python)

Need a single scalar, and it must be fast?
  └─► df.at[label, "col"]     (label-based)
      df.iat[row_pos, col_pos] (position-based)

Assigning to a filtered selection?
  └─► ALWAYS use df.loc[condition, "col"] = value
      NEVER use df[condition]["col"] = value  (SettingWithCopyWarning)
```

### GroupBy Mental Model

```python
# The split-apply-combine pattern:
#
#  df.groupby("key")  →  splits into groups
#      .agg(...)      →  one row per group    (reduces)
#      .transform()   →  same rows as input  (broadcasts)
#      .filter(...)   →  keeps/drops groups
#      .apply(...)    →  most flexible (any return shape)
#
# Named agg (cleanest syntax):
df.groupby("dept").agg(
    avg = ("score","mean"),
    n   = ("score","count"),
    hi  = ("score","max"),
)
```

### Merge / Join Summary

| Type | How | Keeps |
|------|-----|-------|
| Inner | `how="inner"` | Only matching rows |
| Left | `how="left"` | All left rows + matched right |
| Right | `how="right"` | All right rows + matched left |
| Outer | `how="outer"` | All rows from both |
| Semi | `left[left.id.isin(right.id)]` | Left rows that have a match |
| Anti | `left[~left.id.isin(right.id)]` | Left rows with NO match |

### Common String Operations

```python
s.str.lower() / .upper() / .title()
s.str.strip() / .lstrip() / .rstrip()
s.str.replace("old", "new")
s.str.contains("pattern", regex=True)
s.str.startswith("A") / .endswith("z")
s.str.split(" ", expand=True)
s.str.extract(r"(\d+)")
s.str.len()
s.str[:5]                   # Slice characters
```

### DateTime Quick Reference

```python
pd.to_datetime(series)
series.dt.year / .month / .day
series.dt.hour / .minute / .second
series.dt.weekday / .day_name()
series.dt.is_month_end
series.dt.floor("D") / .ceil("h") / .round("min")
series.dt.tz_localize("UTC") / .tz_convert("US/Eastern")
pd.date_range("2024-01-01", periods=12, freq="ME")
df.set_index("date").resample("ME").sum()
```

### Performance Checklist

| Check | Solution |
|-------|---------|
| Object columns with few unique values | `.astype("category")` |
| Float64 everywhere | `pd.to_numeric(col, downcast="float")` |
| `iterrows()` in code | Replace with vectorized op or `.apply()` |
| Growing DataFrame in loop | Collect list of dicts → `pd.DataFrame(list)` |
| Chained indexing `df[mask]["col"] = x` | Use `df.loc[mask, "col"] = x` |
| Slow filter | Try `.query()` with `numexpr` |
| Large CSV, only need some columns | `pd.read_csv(usecols=["a","b"])` |
| Same file read repeatedly | Convert to Parquet once |
| Multiple string `.apply(lambda)` | Use `.str` accessor methods |
| Many NaN checks | Use `.fillna()` with dict for all at once |
