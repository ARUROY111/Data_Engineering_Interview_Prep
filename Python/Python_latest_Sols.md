# Python Interview Master Guide

A single reference for Python interviews: core concepts, OOP, memory, concurrency, pandas, and the questions that are being asked in recent interviews (including data-engineering flavoured ones).

**How to use:** Read the short answer first (what you'd say out loud), then the example. Questions that appeared more than once in your list are merged and cross-referenced.

## Table of Contents
1. [Part 1: Fundamentals](#part-1-fundamentals)
2. [Part 2: Core Language and OOP](#part-2-core-language-and-oop)
3. [Part 3: Advanced and Ecosystem](#part-3-advanced-and-ecosystem)
4. [Part 4: Recently Asked Questions](#part-4-recently-asked-questions)
5. [Part 5: Coding Round Cheat Sheet](#part-5-coding-round-cheat-sheet)
6. [Part 6: Quick Revision Sheet](#part-6-quick-revision-sheet)

---

# Part 1: Fundamentals

### 1. What is the difference between a module and a package?
- **Module:** a single `.py` file containing functions, classes, and variables. Imported with `import mymodule`.
- **Package:** a directory of modules (and sub-packages), traditionally with an `__init__.py` file. (Since Python 3.3, directories without `__init__.py` work as *namespace packages*, but regular packages with `__init__.py` are still the norm.)

```
mypkg/            <- package
    __init__.py
    utils.py      <- module
    db/           <- sub-package
        __init__.py
        conn.py
```
```python
from mypkg.utils import clean_data
from mypkg.db import conn
```
A **library** is a loose term for a collection of packages (e.g., pandas).

### 2. Is Python a compiled or an interpreted language?
Both, in a sense. Python source is first **compiled to bytecode** (`.pyc` files in `__pycache__`), and the bytecode is then **interpreted** by the Python Virtual Machine (PVM). So it is usually called an *interpreted* language, but the accurate answer is "compiled to bytecode, then interpreted by the PVM". Whether a language is compiled or interpreted is a property of the implementation (CPython, PyPy, Jython), not of the language itself.

### 3. Benefits of using Python in the present scenario
- Simple, readable syntax and fast development.
- Dominant in **data engineering, data science, ML/AI, automation, and backend**.
- Huge ecosystem: pandas, NumPy, PySpark, Airflow, boto3, scikit-learn, PyTorch, FastAPI.
- First-class cloud support (AWS Lambda, Glue, Azure Functions, GCP).
- Cross-platform, open source, large community.
- Great as "glue" language: integrates with C/C++, Java, SQL, REST APIs.
- Dynamic typing plus optional type hints for maintainable code.

### 4. Global, protected, and private attributes
Python has no true access modifiers; it uses **naming conventions**.

| Type | Syntax | Meaning |
|---|---|---|
| Public | `name` | Accessible everywhere |
| Protected | `_name` | Convention: for use inside the class and subclasses |
| Private | `__name` | Name-mangled to `_ClassName__name` to avoid accidental access/overriding |

```python
class Account:
    def __init__(self):
        self.owner = "A"       # public
        self._balance = 100    # protected (convention)
        self.__pin = 1234      # private (name mangled)

a = Account()
print(a.owner)           # A
print(a._balance)        # 100 (works, but discouraged)
# print(a.__pin)         # AttributeError
print(a._Account__pin)   # 1234 (mangling, not real security)
```
**Global** (in the "global variable" sense) means a variable defined at module level. Use the `global` keyword to modify it inside a function.

### 5. Is Python case sensitive?
Yes. `name`, `Name`, and `NAME` are three different identifiers. Keywords are also case sensitive (`True` is valid, `true` is not).

### 6. What is pandas?
An open-source Python library for **data manipulation and analysis**, built on NumPy. It provides two main structures: **Series** (1-D labelled array) and **DataFrame** (2-D labelled table). It is used for reading/writing CSV, Excel, JSON, Parquet, SQL; cleaning, filtering, grouping, joining, reshaping, and time-series work.
```python
import pandas as pd
df = pd.read_csv("sales.csv")
print(df.groupby("region")["amount"].sum())
```

### 7. How is exception handling done in Python?
Using `try / except / else / finally` and `raise`.
```python
try:
    x = int(input())
    result = 10 / x
except ValueError:
    print("Not a number")
except ZeroDivisionError as e:
    print("Cannot divide by zero:", e)
else:
    print("Success:", result)       # runs only if no exception
finally:
    print("Always runs")            # cleanup
```
- Raise your own: `raise ValueError("bad input")`.
- Custom exceptions: `class MyError(Exception): pass`.
- Never use a bare `except:`; catch specific exceptions. Use `raise ... from e` to preserve the cause.

### 8. Difference between `for` loop and `while` loop
- **for:** iterates over a sequence/iterable; the number of iterations is known/finite by the iterable.
- **while:** repeats as long as a condition is true; used when the number of iterations is unknown.

```python
for i in range(3):
    print(i)

n = 0
while n < 3:
    print(n)
    n += 1
```
Both support `break`, `continue`, and an `else` clause (runs if the loop was not terminated by `break`).

### 9. Is indentation required in Python?
Yes. Indentation defines code blocks (there are no braces). Inconsistent indentation raises `IndentationError`. Convention (PEP 8): 4 spaces; don't mix tabs and spaces.

### 10. What is the use of `self` in Python?
`self` is the conventional name of the first parameter of instance methods; it refers to the **current instance** of the class. It lets methods access and set instance attributes and call other methods. Python passes it automatically: `obj.method()` is equivalent to `Class.method(obj)`.
```python
class Dog:
    def __init__(self, name):
        self.name = name
    def bark(self):
        return f"{self.name} says woof"
```

---

# Part 2: Core Language and OOP

### 11. How does Python manage memory? Role of reference counting and garbage collection (also covers "How is memory management done in Python?")
- **Private heap:** all objects live in a private heap managed by the Python memory manager (CPython uses `pymalloc` for small objects).
- **Reference counting:** each object tracks how many references point to it. When the count hits **0**, memory is freed immediately.
- **Cyclic garbage collector (`gc` module):** reference counting can't free **reference cycles** (A refers to B, B refers to A). The GC periodically detects unreachable cycles and frees them. It is generational (generations 0, 1, 2); young objects are collected more often.
```python
import sys, gc
a = []
print(sys.getrefcount(a))   # count (+1 for the argument itself)
b = a                        # count increases
del b                        # count decreases
gc.collect()                 # force a collection
```
- Other points: dynamic typing means everything is an object; small ints and some strings are cached/interned; `__del__` is a finalizer, not guaranteed to run promptly; use `with` blocks / `weakref` for resource management.

### 12. Does Python support multiple inheritance?
Yes. A class can inherit from more than one parent. Python resolves attribute lookup using the **MRO (Method Resolution Order)** computed by the **C3 linearization** algorithm, which also solves the "diamond problem".
```python
class A:
    def hi(self): print("A")
class B(A):
    def hi(self): print("B")
class C(A):
    def hi(self): print("C")
class D(B, C): pass

D().hi()          # B
print(D.__mro__)  # D, B, C, A, object
```
`super()` follows the MRO, which is how cooperative multiple inheritance works.

### 13. (Merged into Q11: memory management)

### 14. How to delete a file using Python
```python
import os
os.remove("file.txt")            # raises FileNotFoundError if missing

from pathlib import Path
Path("file.txt").unlink(missing_ok=True)   # modern, Python 3.8+

import shutil
shutil.rmtree("folder")          # delete a directory tree
```
Check first with `os.path.exists()` or just handle `FileNotFoundError` (EAFP style).

### 15. Which sorting technique is used by `sort()` and `sorted()`?
**Timsort**, a hybrid of merge sort and insertion sort. It is **stable**, O(n log n) worst case, and O(n) on already (or nearly) sorted data. `list.sort()` sorts **in place** and returns `None`; `sorted()` returns a **new list** from any iterable.
```python
data = ["bb", "a", "ccc"]
sorted(data, key=len, reverse=True)   # ['ccc', 'bb', 'a']
```

### 16. Difference between list and tuple
| | List | Tuple |
|---|---|---|
| Syntax | `[1, 2]` | `(1, 2)` |
| Mutability | Mutable | Immutable |
| Hashable | No | Yes (if all items are hashable), so usable as dict keys/set members |
| Speed/memory | Slightly slower, more memory | Slightly faster, less memory |
| Use | Homogeneous, changing collections | Fixed records, return multiple values |

Note: a tuple containing a list can still have that inner list mutated.

### 17. What is slicing?
Extracting a portion of a sequence using `seq[start:stop:step]` (stop is exclusive; negative indices count from the end). Works on lists, tuples, strings.
```python
s = [0, 1, 2, 3, 4, 5]
s[1:4]    # [1, 2, 3]
s[::2]    # [0, 2, 4]
s[::-1]   # reversed
s[-2:]    # [4, 5]
```
Slicing creates a **shallow copy** for lists.

### 18. How is multithreading achieved in Python?
Using the `threading` module (or `concurrent.futures.ThreadPoolExecutor`). Because of the **GIL**, threads don't run Python bytecode in parallel in standard CPython, so threading helps with **I/O-bound** work (network, files, DB calls), not CPU-bound work. For CPU-bound work, use `multiprocessing` / `ProcessPoolExecutor`; for high-concurrency I/O, use `asyncio`.
```python
from concurrent.futures import ThreadPoolExecutor
import requests

urls = [...]
with ThreadPoolExecutor(max_workers=8) as ex:
    results = list(ex.map(requests.get, urls))
```

### 19. Which is faster: Python list or NumPy array?
**NumPy arrays**, for numerical work. They store homogeneous data in contiguous memory (C-level), support **vectorized** operations (no Python-level loop), and use far less memory. Typically 10x to 100x faster for numeric operations. Lists are better for small, mixed-type, frequently resized collections.
```python
import numpy as np
arr = np.arange(1_000_000)
arr * 2            # vectorized, fast
[x * 2 for x in range(1_000_000)]   # slower
```

### 20. Explain inheritance in Python (also covers Q38 "What is inheritance in python?")
Inheritance lets a **child (derived) class** acquire attributes and methods of a **parent (base) class**, promoting code reuse and polymorphism. Types: single, multiple, multilevel, hierarchical, hybrid.
```python
class Animal:
    def __init__(self, name):
        self.name = name
    def speak(self):
        return "..."

class Dog(Animal):
    def __init__(self, name, breed):
        super().__init__(name)      # call parent constructor
        self.breed = breed
    def speak(self):                # method overriding
        return "Woof"

d = Dog("Rex", "Lab")
isinstance(d, Animal)   # True
```
Prefer **composition over inheritance** when the relationship is "has-a" rather than "is-a".

### 21. How are classes created in Python?
With the `class` keyword. Classes are themselves objects, instances of `type` (the default metaclass).
```python
class Employee:
    company = "Acme"                 # class attribute
    def __init__(self, name):        # initializer
        self.name = name             # instance attribute
    def greet(self):
        return f"Hi, {self.name}"

e = Employee("Arun")
```
Dynamically: `Employee = type("Employee", (), {"company": "Acme"})`.

### 22. Fibonacci series program
```python
# Iterative (O(n) time, O(1) space)
def fib_series(n):
    a, b = 0, 1
    result = []
    for _ in range(n):
        result.append(a)
        a, b = b, a + b
    return result

print(fib_series(10))   # [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

# Generator version (memory efficient)
def fib_gen(n):
    a, b = 0, 1
    for _ in range(n):
        yield a
        a, b = b, a + b

# Recursive with memoization
from functools import lru_cache
@lru_cache(maxsize=None)
def fib(n):
    return n if n < 2 else fib(n - 1) + fib(n - 2)
```
Plain recursion without memoization is O(2^n); mention that.

### 23. Shallow copy vs deep copy
- **Shallow copy:** new outer object, but nested objects are *shared* references. (`copy.copy()`, `list.copy()`, `[:]`)
- **Deep copy:** new outer object and recursively copied nested objects. (`copy.deepcopy()`)
```python
import copy
a = [[1, 2], [3, 4]]
s = copy.copy(a)
d = copy.deepcopy(a)
a[0][0] = 99
print(s[0][0])   # 99 (shared)
print(d[0][0])   # 1  (independent)
```

### 24. Process of compilation and linking in Python
Python has no separate, explicit linking step like C/C++. The flow is:
1. Source (`.py`) is **compiled to bytecode** by the compiler (cached as `.pyc` in `__pycache__`).
2. The **PVM** executes the bytecode.
3. **Imports are resolved at runtime** (the import system finds, loads, and caches modules in `sys.modules`); this is the closest thing to "linking". C extension modules are loaded dynamically as shared libraries.

### 25. `break`, `continue`, and `pass`
- `break`: exits the nearest enclosing loop.
- `continue`: skips the rest of the current iteration and moves to the next.
- `pass`: a no-op placeholder where a statement is syntactically required.
```python
for i in range(5):
    if i == 1: continue
    if i == 3: break
    print(i)         # 0, 2
def todo(): pass
```

### 26. What is PEP 8?
**Python Enhancement Proposal 8**, the official style guide for Python code: 4-space indentation, max line length 79 (72 for docstrings/comments), `snake_case` for functions/variables, `PascalCase` for classes, `UPPER_CASE` for constants, two blank lines between top-level definitions, imports at the top and grouped (stdlib, third-party, local). Tools: `flake8`, `pylint`, `black`, `ruff`.

### 27. What is an expression?
A combination of values, variables, operators, and function calls that **evaluates to a value**. Examples: `2 + 3`, `x > 5`, `len(s)`. A **statement** performs an action and doesn't necessarily produce a value (`x = 5`, `if ...`, `for ...`). Python 3.8's walrus operator `:=` is an assignment *expression*.

### 28. What is `==` in Python?
The **equality operator**; it compares **values** (calls `__eq__`). Compare with `is`, which compares **identity** (same object in memory).
```python
a = [1, 2]; b = [1, 2]
a == b   # True
a is b   # False
```
Use `is` for `None` checks: `if x is None`.

### 29. Type conversion
Converting one data type to another.
- **Implicit:** Python does it automatically (`3 + 2.5` gives `5.5`; int promoted to float).
- **Explicit (casting):** `int("5")`, `float(3)`, `str(10)`, `list("abc")`, `tuple([1,2])`, `set([1,1,2])`, `dict([(1,2)])`, `bool(0)`.
```python
int("abc")   # ValueError, so handle with try/except
```

### 30. Commonly used built-in modules
`os`, `sys`, `math`, `random`, `datetime`, `time`, `json`, `csv`, `re`, `collections`, `itertools`, `functools`, `pathlib`, `shutil`, `logging`, `argparse`, `subprocess`, `threading`, `multiprocessing`, `asyncio`, `typing`, `dataclasses`, `unittest`, `sqlite3`, `copy`, `heapq`, `uuid`, `hashlib`, `io`, `gzip`/`zipfile`, `urllib`.

---

# Part 3: Advanced and Ecosystem

### 31. Difference between `xrange` and `range`
`xrange` existed only in **Python 2**; it returned a lazy object, while `range` returned a full list. In **Python 3**, `xrange` is gone and `range` behaves like the old `xrange`: a lazy, memory-efficient immutable sequence that supports indexing, slicing, `len()`, and `in` checks in O(1) for ints.

### 32. What is the `zip` function?
Combines multiple iterables element-wise into an iterator of tuples; stops at the shortest iterable (use `itertools.zip_longest` otherwise; `zip(..., strict=True)` in 3.10+ raises if lengths differ).
```python
names = ["a", "b"]; ages = [20, 30]
list(zip(names, ages))        # [('a', 20), ('b', 30)]
dict(zip(names, ages))        # {'a': 20, 'b': 30}
a, b = zip(*[(1, 2), (3, 4)]) # unzip
```

### 33. Django architecture
Django follows **MVT (Model-View-Template)**:
- **Model:** data layer; ORM classes mapping to DB tables.
- **View:** business logic; receives a request, talks to models, returns a response.
- **Template:** presentation layer (HTML with Django template language).
- **URL dispatcher** (`urls.py`) maps URLs to views; **middleware** processes requests/responses; **settings**, **admin**, **forms**, and **migrations** complete the framework.

Flow: Request, then URL router, then View, then Model (DB), then Template, then Response.

### 34. Inheritance in Python
See Q20.

### 35. `*args` and `**kwargs`
- `*args`: collects extra **positional** arguments into a tuple.
- `**kwargs`: collects extra **keyword** arguments into a dict.
```python
def f(a, *args, **kwargs):
    print(a, args, kwargs)

f(1, 2, 3, x=10, y=20)   # 1 (2, 3) {'x': 10, 'y': 20}

nums = [1, 2]; opts = {"sep": "-"}
print(*nums, **opts)     # unpacking at call site
```
Order in a signature: positional, `*args`, keyword-only, `**kwargs`.

### 36. Do runtime errors exist in Python? Example
Yes. Python has **syntax errors** (caught at compile time) and **runtime errors / exceptions** (raised while the program runs, because syntax is fine).
```python
print(10 / 0)        # ZeroDivisionError
int("abc")           # ValueError
[1, 2][5]            # IndexError
{"a": 1}["b"]        # KeyError
undefined_var        # NameError
"a" + 1              # TypeError
```
Also **logical errors**: no exception, but wrong output.

### 37. What are docstrings?
String literals placed as the first statement of a module, class, or function to document it. Accessible via `__doc__` and `help()`. Use triple quotes; common styles: Google, NumPy, reST.
```python
def add(a, b):
    """Return the sum of a and b.

    Args:
        a (int): first number
        b (int): second number
    """
    return a + b
print(add.__doc__)
```

### 38. Capitalize the first letter of a string
```python
"hello world".capitalize()   # 'Hello world'  (rest lowercased)
"hello world".title()        # 'Hello World'  (each word)
s = "hello"; s[0].upper() + s[1:]   # 'Hello'  (rest unchanged)
```

### 39. What are generators?
Functions that use `yield` to produce values **lazily**, one at a time, keeping their state between calls. They return an iterator, use very little memory, and are ideal for large or streaming data (e.g., reading a huge file).
```python
def read_chunks(path):
    with open(path) as f:
        for line in f:
            yield line.strip()

gen_exp = (x * x for x in range(10**9))   # generator expression
```
Generators are single-use; `send()`, `yield from` are advanced features.

### 40. How to write comments
- Single line: `# comment`
- Multi-line: consecutive `#` lines (Python has no true block comment). Triple-quoted strings are sometimes used but they are really string literals/docstrings.
- Docstrings for documentation (see Q37).

### 41. What is the GIL?
The **Global Interpreter Lock** is a mutex in CPython that allows only **one thread to execute Python bytecode at a time**. It simplifies memory management (reference counting is not thread-safe otherwise). Consequences: threads don't give CPU parallelism, but release the GIL during I/O, so they work for I/O-bound tasks. Workarounds: `multiprocessing`, C extensions that release the GIL (NumPy), `asyncio`, or other implementations. **Recent update:** Python 3.13 introduced an experimental free-threaded (no-GIL) build (PEP 703), and it became officially supported (still optional) in 3.14.

### 42. Is Django better than Flask?
Neither is universally better; it depends on the use case.
| | Django | Flask |
|---|---|---|
| Philosophy | "Batteries included" full-stack | Micro-framework, minimal core |
| Built-in | ORM, admin, auth, forms, migrations | Routing, templating (Jinja2); add the rest via extensions |
| Best for | Large, database-driven apps, fast delivery | Small services, APIs, flexibility, microservices |
| Learning curve | Steeper | Gentler |

For modern async APIs, also mention **FastAPI** (type hints, automatic OpenAPI docs, high performance).

### 43. What is Flask? Benefits
A lightweight WSGI web micro-framework built on Werkzeug and Jinja2. Benefits: minimal and flexible, easy to learn, no forced structure or ORM, large extension ecosystem (Flask-SQLAlchemy, Flask-Login), built-in dev server and debugger, easy testing, great for microservices and prototypes.
```python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def home():
    return "Hello"
```

### 44. What is PIP?
**Pip Installs Packages**, the standard package manager for Python; installs from PyPI (or other indexes).
```bash
pip install pandas
pip install pandas==2.2.0
pip uninstall pandas
pip freeze > requirements.txt
pip install -r requirements.txt
pip list
```
Use inside a **virtual environment** (`python -m venv .venv`). Modern alternatives: `uv`, `poetry`, `pipenv`, `conda`.

### 45. Ensuring code is compatible with Python 2 and 3
Practically, Python 2 reached end-of-life in January 2020, so new code should target Python 3 only. If asked: use the `__future__` imports (`from __future__ import print_function, division, absolute_import, unicode_literals`), the `six` or `future` libraries, `2to3`/`python-modernize` tools, avoid `xrange`/`iteritems`/`print` statements, and test under both with `tox`.

### 46. `%` vs `/` vs `//`
- `%` modulus: remainder. `7 % 3 = 1`
- `/` true division: always float. `7 / 2 = 3.5`
- `//` floor division: rounds **down** toward negative infinity. `7 // 2 = 3`, `-7 // 2 = -4`

Identity: `a == (a // b) * b + (a % b)`.

### 47. Why doesn't Python deallocate all memory on exit?
CPython tries to clean up but cannot guarantee full deallocation because of: objects with circular references or `__del__` finalizers, objects referenced from global namespaces/module-level data, memory allocated by C libraries/extension modules, and the interpreter's own internal pools (e.g., small-object allocator, interned strings). Also, for speed, the OS reclaims process memory on exit anyway. `atexit` handlers and explicit cleanup (context managers) are the right tools for critical resources.

### 48. Why is a set unordered? Mutable or immutable?
A set is implemented as a **hash table**; the position of an element depends on its hash, not insertion order, so there's no index and no order guarantee (unlike dicts, which preserve insertion order since 3.7). A **set is mutable** (`add`, `remove`, `discard`, `update`), but its **elements must be hashable (immutable)**. `frozenset` is the immutable version. Membership test is O(1) average; sets store unique elements only.
```python
s = {3, 1, 2}
s.add(4)
# s[0]  -> TypeError (no indexing)
{[1, 2]}   # TypeError: unhashable type: 'list'
```

### 49. DataFrame vs Series
- **Series:** 1-D labelled array, a single column, with an index and one dtype.
- **DataFrame:** 2-D labelled table; a collection of Series (columns) sharing an index; columns can have different dtypes.
```python
s = pd.Series([1, 2, 3], name="x")
df = pd.DataFrame({"x": [1, 2, 3], "y": ["a", "b", "c"]})
type(df["x"])    # Series
type(df[["x"]])  # DataFrame
```

### 50. What does `len()` do?
Returns the number of items in a container (list, tuple, string, dict, set, range, DataFrame rows, etc.) by calling the object's `__len__` method. Time complexity O(1) for built-ins. Raises `TypeError` for objects without a length (like ints or generators).

---

# Part 4: Recently Asked Questions

These are frequently asked in recent interviews, especially for data engineering, backend, and automation roles.

### 51. What are decorators?
A decorator is a function that takes a function and returns a modified function, adding behaviour without changing the original code. `@decorator` is syntactic sugar for `func = decorator(func)`.
```python
import functools, time

def timer(func):
    @functools.wraps(func)          # preserves name/docstring
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        print(f"{func.__name__} took {time.perf_counter() - start:.3f}s")
        return result
    return wrapper

@timer
def load(): ...
```
Uses: logging, retry, caching, auth, timing. Decorators with arguments need one extra nesting level.

### 52. What are context managers and the `with` statement?
They guarantee setup/cleanup (files, DB connections, locks) even on exceptions. Implement via `__enter__`/`__exit__` or `contextlib.contextmanager`.
```python
from contextlib import contextmanager

@contextmanager
def db_conn():
    conn = connect()
    try:
        yield conn
    finally:
        conn.close()

with open("f.txt") as f:
    data = f.read()
```

### 53. Mutable vs immutable types, and the mutable default argument trap
- Immutable: `int, float, str, tuple, frozenset, bytes, bool`. Mutable: `list, dict, set, bytearray`, user-defined objects.
- **Trap:** default arguments are evaluated once, at function definition.
```python
def add(x, lst=[]):          # BUG: shared list
    lst.append(x); return lst
add(1); add(2)               # [1, 2] on second call!

def add(x, lst=None):        # correct
    if lst is None:
        lst = []
    lst.append(x); return lst
```

### 54. `is` vs `==`
`==` compares value (equality); `is` compares identity (`id(a) == id(b)`). Small ints (-5 to 256) and interned strings may be shared, which makes `is` unreliable for values. Use `is` only for `None`, `True`, `False`, and singletons.

### 55. List comprehension, dict/set comprehension, and generator expression
```python
squares = [x*x for x in range(10) if x % 2 == 0]
d = {k: v for k, v in zip("abc", range(3))}
s = {x % 3 for x in range(10)}
g = (x*x for x in range(10))     # lazy
```
Comprehensions are generally faster and more readable than equivalent loops with `append`. Avoid overly nested ones.

### 56. `lambda`, `map`, `filter`, `reduce`
```python
f = lambda x: x * 2
list(map(f, [1, 2, 3]))                    # [2, 4, 6]
list(filter(lambda x: x > 1, [1, 2, 3]))   # [2, 3]
from functools import reduce
reduce(lambda a, b: a + b, [1, 2, 3])      # 6
sorted(people, key=lambda p: p["age"])
```
Lambdas are single-expression anonymous functions. Comprehensions are usually preferred over `map`/`filter` for readability.

### 57. Iterator vs iterable vs generator
- **Iterable:** has `__iter__` (list, str, dict).
- **Iterator:** has `__iter__` and `__next__`; remembers position; raises `StopIteration` when done.
- **Generator:** a convenient way to create an iterator using `yield`.
```python
it = iter([1, 2])
next(it); next(it)   # then StopIteration
```

### 58. `@staticmethod`, `@classmethod`, and instance methods
| | First arg | Access | Use |
|---|---|---|---|
| Instance method | `self` | instance + class | normal behaviour |
| `@classmethod` | `cls` | class state | alternative constructors, factory methods |
| `@staticmethod` | none | neither | utility function grouped in class |
```python
class User:
    def __init__(self, name): self.name = name
    @classmethod
    def from_csv(cls, line): return cls(line.split(",")[0])
    @staticmethod
    def is_valid(name): return bool(name)
```

### 59. Dunder (magic) methods
Special methods with double underscores that customise behaviour: `__init__`, `__new__`, `__str__`, `__repr__`, `__len__`, `__getitem__`, `__iter__`, `__next__`, `__eq__`, `__hash__`, `__lt__`, `__add__`, `__call__`, `__enter__`, `__exit__`. `__repr__` is for developers (unambiguous), `__str__` for end users. `__new__` creates the instance; `__init__` initialises it.

### 60. Polymorphism, encapsulation, abstraction
- **Polymorphism:** same interface, different behaviour (method overriding, duck typing: "if it quacks like a duck").
- **Encapsulation:** bundling data and methods, restricting access via naming conventions and properties.
- **Abstraction:** hiding implementation using abstract base classes.
```python
from abc import ABC, abstractmethod
class Shape(ABC):
    @abstractmethod
    def area(self): ...
```

### 61. `@property`
Lets you expose a method as an attribute, enabling validation without breaking the public API.
```python
class Temp:
    def __init__(self, c): self._c = c
    @property
    def c(self): return self._c
    @c.setter
    def c(self, v):
        if v < -273.15: raise ValueError
        self._c = v
```

### 62. Dataclasses
`@dataclass` auto-generates `__init__`, `__repr__`, `__eq__` for data-holding classes (Python 3.7+). Options: `frozen=True` (immutable), `slots=True` (3.10+, less memory), `order=True`.
```python
from dataclasses import dataclass, field
@dataclass(frozen=True)
class Order:
    id: int
    items: list[str] = field(default_factory=list)
```

### 63. Type hints and static typing
Optional annotations that document intent and enable tools like `mypy`/`pyright`; they are **not enforced at runtime**.
```python
def total(prices: list[float], tax: float = 0.1) -> float: ...
from typing import Optional, Union
x: int | None = None      # 3.10+ union syntax
```
Pydantic uses hints for runtime validation (very common in FastAPI and data pipelines).

### 64. Multithreading vs multiprocessing vs asyncio
| | Threading | Multiprocessing | asyncio |
|---|---|---|---|
| Parallel CPU? | No (GIL) | Yes | No (single thread) |
| Best for | I/O-bound, blocking libs | CPU-bound | Massive concurrent I/O |
| Overhead | Low | High (separate memory, pickling) | Very low |
| Model | Preemptive | Separate processes | Cooperative (`await`) |
```python
import asyncio
async def fetch(i):
    await asyncio.sleep(1)
    return i
async def main():
    return await asyncio.gather(*(fetch(i) for i in range(5)))
asyncio.run(main())
```

### 65. Python 3.10 to 3.13 features worth knowing
- 3.8: walrus `:=`, positional-only params `/`, f-string `=`.
- 3.9: `dict | dict` merge, `list[int]` generics, `str.removeprefix`.
- 3.10: structural pattern matching (`match/case`), `X | Y` type unions, better error messages.
- 3.11: major speedups (about 10 to 60%), exception groups (`except*`), `tomllib`, `TaskGroup`.
- 3.12: improved f-strings, `type` statement and new generic syntax, faster comprehensions.
- 3.13: experimental free-threaded (no-GIL) mode, experimental JIT, improved REPL.
- 3.14: free-threaded build officially supported, template strings (t-strings), deferred evaluation of annotations.

If unsure about a detail, say "as far as I know" and mention you'd verify in the release notes.
```python
match command.split():
    case ["go", direction]: ...
    case ["quit"]: ...
    case _: ...
```

### 66. How do you handle large files that don't fit in memory?
- Read line-by-line or in chunks (generators): `for line in f`.
- pandas: `pd.read_csv(path, chunksize=100_000)`, specify `dtype`, `usecols`, use `category` dtype, prefer **Parquet**.
- Use **Dask**, **Polars** (lazy), **PySpark**, or **DuckDB** for out-of-core processing.
- Stream to/from S3 with `boto3` (`get_object()["Body"].iter_chunks()`), or use `smart_open`.
```python
total = 0
for chunk in pd.read_csv("big.csv", chunksize=500_000):
    total += chunk["amount"].sum()
```

### 67. How do you optimize Python code performance?
Profile first (`cProfile`, `line_profiler`, `timeit`); choose better algorithms/data structures (set/dict lookups O(1)); use built-ins and comprehensions; vectorize with NumPy/pandas; cache (`functools.lru_cache`); use generators; avoid repeated string concatenation (`"".join`); use multiprocessing/asyncio where suited; offload to C-based tools (Cython, Numba, Polars); upgrade to a newer Python version.

### 68. `apply` vs vectorization in pandas
Vectorized operations (`df["a"] * 2`, `.str`, `.dt`, `np.where`) run in optimized C code and are much faster than `df.apply(func, axis=1)`, which loops in Python row by row. Prefer vectorization, then `map`/`transform`, and use `apply` as a last resort.

### 69. Common pandas operations asked in interviews
```python
df.head(); df.info(); df.describe(); df.shape
df.isna().sum()                          # missing values
df.dropna(); df.fillna({"col": 0})
df.drop_duplicates(subset=["id"])
df[df["amount"] > 100]                   # filter
df.groupby("region").agg(total=("amount", "sum"), n=("id", "count"))
pd.merge(a, b, on="id", how="left")      # joins: inner/left/right/outer
pd.concat([a, b], axis=0)
df.pivot_table(index="r", columns="m", values="v", aggfunc="sum")
df.sort_values("amount", ascending=False)
df["date"] = pd.to_datetime(df["date"])
df.rename(columns={"a": "b"})
df.astype({"id": "int64"})
df.to_parquet("out.parquet")
```
Extra: `loc` (label-based) vs `iloc` (position-based); `merge` vs `join` vs `concat`; `SettingWithCopyWarning` (use `.loc` or `.copy()`); pandas 2.x with Arrow-backed dtypes and Copy-on-Write.

### 70. How do you handle missing data and duplicates in pandas?
Detect: `isna()`, `isnull().sum()`. Handle: `dropna()`, `fillna()` (constant, mean/median, forward/back fill `ffill`/`bfill`), `interpolate()`. Duplicates: `duplicated()`, `drop_duplicates(subset, keep="first"/"last")`. Decide based on business meaning of the missing values; document the rule.

### 71. How do you connect to AWS services with Python?
Use **boto3**.
```python
import boto3
s3 = boto3.client("s3")
s3.download_file("my-bucket", "path/file.csv", "/tmp/file.csv")
obj = s3.get_object(Bucket="my-bucket", Key="path/file.csv")

glue = boto3.client("glue")
glue.start_job_run(JobName="my_job")
```
Credentials come from the IAM role (Lambda/Glue/EC2), environment variables, or the shared config file; never hardcode keys. Use paginators for large listings (`s3.get_paginator("list_objects_v2")`) and handle `ClientError`.

### 72. How do you write production-quality ETL code in Python?
Idempotent, re-runnable jobs; config out of code (env vars/Parameter Store/Secrets Manager); structured `logging` (not `print`); retries with exponential backoff; specific exception handling; data quality checks (schema, nulls, row counts); modular functions with type hints; unit tests (`pytest`, mocks via `moto` for AWS); CI/CD; incremental loads with watermarks; partitioned columnar output (Parquet).
```python
import logging
logger = logging.getLogger(__name__)
logger.info("Loaded %d rows", len(df))
```

### 73. Retry with exponential backoff (common practical question)
```python
import time, functools

def retry(times=3, delay=1, backoff=2, exceptions=(Exception,)):
    def deco(fn):
        @functools.wraps(fn)
        def wrapper(*a, **kw):
            d = delay
            for attempt in range(1, times + 1):
                try:
                    return fn(*a, **kw)
                except exceptions:
                    if attempt == times:
                        raise
                    time.sleep(d)
                    d *= backoff
        return wrapper
    return deco
```

### 74. Virtual environments and dependency management
A virtual environment is an isolated Python environment with its own packages, preventing version conflicts between projects.
```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```
Tools: `venv`, `virtualenv`, `conda`, `poetry`, `pipenv`, `uv` (fast, increasingly popular). Pin versions for reproducibility.

### 75. How do you write unit tests in Python?
Use `pytest` (or `unittest`). Use fixtures, parametrization, and mocking (`unittest.mock`, `pytest-mock`, `moto` for AWS).
```python
import pytest

@pytest.mark.parametrize("a,b,expected", [(1, 2, 3), (0, 0, 0)])
def test_add(a, b, expected):
    assert add(a, b) == expected

def test_zero_div():
    with pytest.raises(ZeroDivisionError):
        1 / 0
```

### 76. Python's `__name__ == "__main__"`
When a file is run directly, `__name__` is `"__main__"`; when imported, it's the module name. The guard ensures code runs only on direct execution.
```python
def main(): ...
if __name__ == "__main__":
    main()
```

### 77. `deepcopy` vs assignment, and how arguments are passed
Assignment (`b = a`) copies the **reference**, not the object. Python uses **"pass by object reference"** (call by sharing): mutating a mutable argument inside a function affects the caller's object; rebinding the name does not.

### 78. Explain `collections` module highlights
`Counter` (frequency counts), `defaultdict` (default values), `deque` (O(1) appends/pops at both ends), `namedtuple`, `OrderedDict`, `ChainMap`.
```python
from collections import Counter, defaultdict
Counter("banana").most_common(2)     # [('a', 3), ('n', 2)]
d = defaultdict(list); d["k"].append(1)
```

### 79. Time complexity of common operations
| Operation | list | dict / set |
|---|---|---|
| Index / lookup | O(1) | O(1) avg |
| Append | O(1) amortized | O(1) avg |
| Insert / delete at front | O(n) | n/a |
| `x in container` | O(n) | O(1) avg |
| Sort | O(n log n) | n/a |

Use `collections.deque` for queues, `heapq` for priority queues, `bisect` for sorted inserts.

### 80. How do you read/write JSON, CSV, and Parquet?
```python
import json, csv, pandas as pd
with open("a.json") as f: data = json.load(f)
json.dumps(data, indent=2)

with open("a.csv", newline="") as f:
    for row in csv.DictReader(f): ...

df = pd.read_parquet("a.parquet")
df.to_parquet("b.parquet", index=False)
```
Parquet is columnar and compressed, so it is faster and smaller than CSV for analytics.

### 81. What are `__slots__`?
Declares a fixed set of attributes so instances don't have a per-instance `__dict__`, saving memory and slightly speeding access. Trade-off: no dynamic attributes.
```python
class P:
    __slots__ = ("x", "y")
```

### 82. What is a closure?
A nested function that remembers variables from its enclosing scope even after the outer function has returned. Use `nonlocal` to modify them.
```python
def counter():
    n = 0
    def inc():
        nonlocal n
        n += 1
        return n
    return inc
c = counter(); c(); c()   # 2
```

### 83. LEGB scope rule
Name lookup order: **L**ocal, **E**nclosing, **G**lobal, **B**uilt-in. Use `global`/`nonlocal` to rebind names in outer scopes.

### 84. How does Python's `dict` work (and ordering)?
A hash table with open addressing; keys must be hashable; average O(1) lookup/insert/delete. Since Python 3.7, insertion order is guaranteed. Hash collisions are handled by probing. Keys are compared using `__hash__` and `__eq__`.

### 85. Logging vs print
`logging` offers levels (DEBUG, INFO, WARNING, ERROR, CRITICAL), handlers (file, stream, CloudWatch), formatting, and configuration without code changes. It's the right choice for production and pipelines.
```python
import logging
logging.basicConfig(level=logging.INFO, format="%(asctime)s %(levelname)s %(message)s")
```

---

# Part 5: Coding Round Cheat Sheet

```python
# 1. Reverse a string
s[::-1]

# 2. Palindrome check
def is_pal(s):
    s = "".join(c.lower() for c in s if c.isalnum())
    return s == s[::-1]

# 3. Anagram check
from collections import Counter
def is_anagram(a, b): return Counter(a) == Counter(b)

# 4. Find duplicates in a list
def duplicates(lst):
    seen, dup = set(), set()
    for x in lst:
        (dup if x in seen else seen).add(x)
    return dup

# 5. Two Sum (O(n))
def two_sum(nums, target):
    seen = {}
    for i, n in enumerate(nums):
        if target - n in seen:
            return [seen[target - n], i]
        seen[n] = i

# 6. Word frequency
from collections import Counter
Counter(text.split()).most_common()

# 7. Flatten nested list
def flatten(lst):
    for x in lst:
        if isinstance(x, list):
            yield from flatten(x)
        else:
            yield x

# 8. Remove duplicates, preserve order
list(dict.fromkeys(lst))

# 9. Factorial
import math; math.factorial(5)

# 10. Prime check
def is_prime(n):
    if n < 2: return False
    return all(n % i for i in range(2, int(n**0.5) + 1))

# 11. Second largest
sorted(set(lst))[-2]

# 12. Merge two dicts
{**a, **b}     # or a | b (3.9+)

# 13. Group by key
from collections import defaultdict
groups = defaultdict(list)
for k, v in pairs: groups[k].append(v)

# 14. Count vowels
sum(c in "aeiou" for c in s.lower())

# 15. FizzBuzz
for i in range(1, 16):
    print("FizzBuzz" if i % 15 == 0 else "Fizz" if i % 3 == 0 else "Buzz" if i % 5 == 0 else i)

# 16. Binary search
def bsearch(a, t):
    lo, hi = 0, len(a) - 1
    while lo <= hi:
        mid = (lo + hi) // 2
        if a[mid] == t: return mid
        lo, hi = (mid + 1, hi) if a[mid] < t else (lo, mid - 1)
    return -1

# 17. Read a big file in chunks
def chunks(path, size=1024 * 1024):
    with open(path, "rb") as f:
        while block := f.read(size):
            yield block

# 18. pandas: top 3 per group
df.sort_values("sales", ascending=False).groupby("region").head(3)
```

---

# Part 6: Quick Revision Sheet

- **Compiled + interpreted:** source to bytecode to PVM.
- **Memory:** private heap + reference counting + generational cyclic GC.
- **GIL:** one thread executes bytecode at a time; threads for I/O, processes for CPU. 3.13+ has optional free-threaded builds.
- **Mutable:** list, dict, set. **Immutable:** int, str, tuple, frozenset.
- **Sorting:** Timsort, stable, O(n log n).
- **`is` vs `==`:** identity vs value.
- **Shallow vs deep copy:** shared nested objects vs fully independent.
- **Generator:** `yield`, lazy, memory efficient, single-use.
- **Decorator:** function wrapping function; use `functools.wraps`.
- **MRO:** C3 linearization, check `Class.__mro__`.
- **Mutable default args:** use `None` as default.
- **`range` (py3):** lazy sequence; `xrange` is Python 2 only.
- **Set:** unordered, mutable, unique, hashable elements only.
- **Series vs DataFrame:** 1-D vs 2-D.
- **PEP 8:** style guide; `black`/`ruff`/`flake8`.

## Interview Tips
1. Answer in the order: **definition, why it matters, tiny example, trade-off/gotcha**.
2. Tie answers to your real work (AWS Glue/Lambda pipelines, Airflow DAGs, pandas transformations, retries, logging) so answers sound experienced, not memorized.
3. For "what's the difference" questions, give a one-line contrast, then the example.
4. If you don't know, reason aloud from first principles rather than guessing.
5. Revisit the version-specific points (Q41, Q65) before the interview, since language features keep changing.
