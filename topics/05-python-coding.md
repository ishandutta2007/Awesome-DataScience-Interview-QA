# 🐍 Python & Coding

[← Back to main README](../README.md)

---

### Q: What is the difference between a list and a tuple in Python, and when would you use each?

**Answer:**
Lists are **mutable** (can be modified in place) and use more memory; tuples are **immutable** and slightly more memory/performance efficient. Use tuples for fixed collections that shouldn't change (e.g., coordinates, function return values with multiple items, dictionary keys since they're hashable) and lists for collections you'll be modifying (appending, sorting, removing elements).

---

### Q: Explain the difference between `is` and `==` in Python.

**Answer:**
`==` checks **value equality** (calls `__eq__`), while `is` checks **identity** — whether two references point to the exact same object in memory. For small integers and interned strings, CPython may reuse objects so `is` can appear to work like `==` by coincidence — but this is an implementation detail, not guaranteed behavior, so `is` should only be used for identity checks like `x is None`.

```python
a = [1, 2, 3]
b = [1, 2, 3]
a == b  # True (same values)
a is b  # False (different objects)
```

---

### Q: What are Python generators, and why would you use one instead of a list?

**Answer:**
A generator (using `yield`) produces values **lazily, one at a time**, instead of computing and storing an entire sequence in memory upfront. This makes generators memory-efficient for large or infinite sequences and for streaming pipelines, since values are only computed when requested. The tradeoff is you can only iterate through a generator once (it's not re-usable/indexable like a list).

```python
def squares(n):
    for i in range(n):
        yield i ** 2

for val in squares(1_000_000):  # constant memory, not O(n)
    process(val)
```

---

### Q: What is the Global Interpreter Lock (GIL), and how does it affect multithreading in Python?

**Answer:**
The GIL is a mutex in CPython that ensures only one thread executes Python bytecode at a time, even on multi-core machines. This means **CPU-bound multithreading doesn't give a speedup** in pure Python (threads compete for the GIL) — for CPU-bound work, use **multiprocessing** (separate processes, separate GILs) or vectorized libraries (NumPy releases the GIL during heavy C-level computation). Multithreading still helps for **I/O-bound work** (network calls, file I/O), since the GIL is released during blocking I/O operations.

---

### Q: What's the difference between `*args` and `**kwargs`?

**Answer:**
`*args` collects extra **positional** arguments into a tuple; `**kwargs` collects extra **keyword** arguments into a dictionary. They let functions accept a variable number of arguments.

```python
def example(*args, **kwargs):
    print(args)    # tuple, e.g. (1, 2, 3)
    print(kwargs)  # dict, e.g. {'a': 1, 'b': 2}

example(1, 2, 3, a=1, b=2)
```

---

### Q: How does Python's list comprehension differ from a generator expression, performance-wise?

**Answer:**
`[x**2 for x in range(n)]` (list comprehension) builds the **entire list in memory immediately**. `(x**2 for x in range(n))` (generator expression, same syntax with parentheses) produces values **lazily**, one at a time, and doesn't hold the whole sequence in memory. For a one-time pass over large data (e.g., feeding into `sum()` or a loop), the generator expression is more memory-efficient; for cases needing random access, length, or multiple iterations, a list is necessary.

---

### Q: Write a function to find duplicate elements in a list efficiently.

**Answer:**
```python
def find_duplicates(items):
    seen = set()
    duplicates = set()
    for item in items:
        if item in seen:
            duplicates.add(item)
        else:
            seen.add(item)
    return list(duplicates)
```
This runs in **O(n) time** using two hash sets, versus an O(n²) nested-loop approach. Using `collections.Counter` is an equally clean one-liner alternative: `[k for k, v in Counter(items).items() if v > 1]`.

---

### Q: Explain how `pandas.merge()` differs from `pandas.concat()`.

**Answer:**
`merge()` performs a **SQL-style join** on one or more key columns (or indexes), combining columns from two DataFrames based on matching values — analogous to SQL `JOIN`. `concat()` simply **stacks** DataFrames along an axis (rows by default, `axis=0`, or columns with `axis=1`) without any key-based matching logic — analogous to SQL `UNION`/appending. Use `merge()` when combining related data by a common key; use `concat()` when combining datasets with the same structure (e.g., appending monthly files).

---

### Q: What's the difference between `.loc` and `.iloc` in pandas?

**Answer:**
`.loc` selects rows/columns by **label** (index name or column name), and importantly, **includes the end of a slice range** (`df.loc[0:5]` includes row labeled 5). `.iloc` selects by **integer position**, following standard Python slicing rules (exclusive of the end index). Mixing them up is a very common source of off-by-one bugs, especially after filtering/sorting a DataFrame changes its index.

---

### Q: How would you efficiently apply a function to every row of a large pandas DataFrame, and what are the performance tradeoffs?

**Answer:**
`df.apply(func, axis=1)` is readable but **row-wise apply is slow** (effectively a Python-level loop under the hood). Prefer **vectorized operations** (e.g., `df['c'] = df['a'] + df['b']`) whenever the logic can be expressed that way — these use optimized C/NumPy under the hood and are typically 10-100x faster. When true row-wise custom logic is unavoidable, consider `.apply` with `raw=True` for simple numeric ops, `np.vectorize`, or libraries like `numba`/`Cython` for performance-critical custom functions on very large data.

---

### Q: What is a decorator in Python, and write a simple example (e.g., a timing decorator).

**Answer:**
A decorator is a function that wraps another function to extend/modify its behavior without changing its source code, using the `@decorator_name` syntax.

```python
import time
from functools import wraps

def timer(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"{func.__name__} took {time.time() - start:.4f}s")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(1)
```

---

### Q: How do you handle memory efficiency when working with a dataset too large to fit in RAM, using Python/pandas?

**Answer:**
Options include: (1) reading in **chunks** with `pd.read_csv(chunksize=...)` and processing incrementally, (2) specifying tighter **dtypes** upfront (e.g., `int32` instead of `int64`, `category` dtype for low-cardinality strings) to cut memory footprint, (3) using **columnar/out-of-core formats** like Parquet with column pruning (only read needed columns), (4) switching to a distributed/lazy-evaluation engine like **Dask, Polars, or Spark** which can process data larger than memory, or (5) pushing aggregation work into a database/warehouse query instead of pulling raw rows into Python.

---

### Q: What's the difference between shallow copy and deep copy in Python?

**Answer:**
A **shallow copy** (`copy.copy()` or `list(orig)`) creates a new outer object, but nested/mutable objects inside it are still **references to the same underlying objects** — modifying a nested list in the copy affects the original. A **deep copy** (`copy.deepcopy()`) recursively copies all nested objects, producing a fully independent structure. This matters a lot with nested lists/dicts or objects containing mutable attributes.

---

### Q: Write a Python function to check if two strings are anagrams of each other, and state its time complexity.

**Answer:**
```python
from collections import Counter

def is_anagram(s1: str, s2: str) -> bool:
    return Counter(s1) == Counter(s2)
```
Time complexity: **O(n)**, where n is the string length — building each Counter is O(n) and comparing two Counters is O(k) where k is the number of unique characters (bounded, so effectively O(n) overall). A sort-based approach (`sorted(s1) == sorted(s2)`) is a simpler alternative but runs in **O(n log n)**.

---
