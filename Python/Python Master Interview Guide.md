# Python Master Interview Guide: Core Concepts, Advanced Internals & Production Engineering

> **A consolidated, comprehensive technical reference combining core fundamentals, CPython internals, concurrency, modern typing, error handling, design patterns, and algorithmic problem-solving.**

---

## Table of Contents
1. [Language Architecture, Execution Model & Core Fundamentals](#1-language-architecture-execution-model--core-fundamentals)
2. [Data Structures, Collections & Operators](#2-data-structures-collections--operators)
3. [Functions, Scopes, Closures & Functional Programming](#3-functions-scopes-closures--functional-programming)
4. [Object-Oriented Programming (OOP), Metaprogramming & Descriptors](#4-object-oriented-programming-oop-metaprogramming--descriptors)
5. [Memory Management, CPython Internals & the GIL](#5-memory-management-cpython-internals--the-gil)
6. [Concurrency, Multithreading, Multiprocessing & AsyncIO](#6-concurrency-multithreading-multiprocessing--asyncio)
7. [Modern Type Hinting, Protocols & Static Analysis](#7-modern-type-hinting-protocols--static-analysis)
8. [Error Handling, Context Managers & Exception Groups](#8-error-handling-context-managers--exception-groups)
9. [Python Standard Library Power Tools](#9-python-standard-library-power-tools)
10. [Testing, Debugging, Profiling & Optimization](#10-testing-debugging-profiling--optimization)
11. [Packaging, Virtual Environments & Dependency Management](#11-packaging-virtual-environments--dependency-management)
12. [Software Architecture, Design Patterns & Enterprise API Design](#12-software-architecture-design-patterns--enterprise-api-design)
13. [Essential Coding & Algorithmic Problem-Solving Patterns](#13-essential-coding--algorithmic-problem-solving-patterns)

---

## 1. Language Architecture, Execution Model & Core Fundamentals

### Q1. What are the key features of Python?
Python is a high-level, interpreted, dynamically typed, and multi-paradigm language. Key features include:
* **Multi-Paradigm**: Supports Object-Oriented, Functional, and Procedural programming styles.
* **Dynamic Typing & Automatic Memory Management**: Types are bound at runtime; memory allocation and deallocation are handled automatically via reference counting and generational garbage collection.
* **Extensive Standard Library ("Batteries Included")**: Ships with built-in modules for networking, concurrency, cryptography, file systems, and serialization.
* **C Extensibility**: Seamlessly binds with C/C++, Rust, and Fortran libraries for performance-critical tasks.

---

### Q2. What happens from Python source code to execution?
The Python execution lifecycle consists of three distinct stages:
1. **Lexical Analysis & Parsing**: The interpreter tokenizes raw source text and constructs an Abstract Syntax Tree (AST).
2. **Bytecode Compilation**: The AST is compiled into platform-independent bytecode instructions (`.pyc` files or in-memory code objects).
3. **Virtual Machine Evaluation**: The CPython Virtual Machine executes bytecode instructions within its evaluation loop (`_PyEval_EvalFrameDefault` in `ceval.c`), managing the call stack, memory references, and OS interactions.

```
Source Code (.py) ──> Lexer & Parser ──> AST ──> Bytecode (.pyc) ──> CPython VM (ceval loop) ──> Hardware
```

---

### Q3. What is the difference between Python 2 and Python 3?
* `print` became a built-in function instead of a keyword statement.
* Text strings are Unicode (`str`) by default; binary data is handled via `bytes`.
* Integer division `/` performs true division (returns `float`), while `//` performs floor division.
* Legacy features removed: `xrange()` unified into `range()`, `raw_input()` replaced by `input()`, `long` merged into `int`.
* Introduced modern language features: Type hints (PEP 484), `async`/`await` (PEP 492), f-strings (PEP 498), pattern matching (PEP 634), exception groups (PEP 654), and the free-threaded no-GIL build (PEP 703). Python 2 reached End-Of-Life in January 2020.

---

### Q4. What is dynamic typing vs duck typing in Python?
* **Dynamic Typing**: Variable types are evaluated and bound at runtime. A variable name is merely a reference/label bound to an object; it can refer to objects of different types during execution.
* **Duck Typing**: Focuses on what an object can do (its methods and attributes) rather than its explicit class hierarchy (*"If it walks like a duck and quacks like a duck, it is a duck"*). Code executes operations on objects assuming they satisfy the expected interface without requiring explicit `isinstance()` checks.

---

### Q5. What is the difference between `is` and `==`?
* `==` checks for **value equality**: calls `__eq__()` to check whether two objects have equivalent data.
* `is` checks for **identity equality**: verifies whether two variables point to the exact same object in memory by comparing memory addresses (`id(a) == id(b)`).

```python
a = [1, 2, 3]
b = [1, 2, 3]
print(a == b)  # True (equal values)
print(a is b)  # False (distinct memory allocations)
```

---

### Q6. Why must you always write `is None` instead of `== None`?
`None` is a singleton in Python.
1. `is None` performs a direct pointer comparison at the bytecode level ($O(1)$).
2. `== None` executes the `__eq__()` method, which can be overridden or overloaded by custom classes, potentially returning misleading results. PEP 8 explicitly mandates `is None` and `is not None`.

---

### Q7. What are Python namespaces and scopes?
A **namespace** is a dictionary mapping symbolic names to objects. Python manages:
* **Built-in namespace**: Contains built-in functions (`len`, `range`, `print`) and exceptions.
* **Global/Module namespace**: Contains module-level definitions.
* **Local namespace**: Created during function invocation for parameters and local variables.
* **Enclosing namespace**: Namespaces of outer enclosing functions (in nested functions/closures).

---

### Q8. What is the LEGB rule?
The LEGB rule determines the lookup hierarchy when resolving variable names:
1. **L (Local)**: Names assigned inside the executing function.
2. **E (Enclosing)**: Names in the local scope of outer enclosing functions.
3. **G (Global)**: Names declared at the module root level.
4. **B (Built-in)**: Names preloaded in Python's built-in namespace.

If the identifier is not found in any of these scopes, a `NameError` is raised.

---

### Q9. What is the difference between `global` and `nonlocal`?
* `global x`: Declares that `x` refers to the module-level global variable.
* `nonlocal x`: Declares that `x` refers to a variable in the nearest enclosing (non-global) scope.
* *Note*: Neither keyword is needed to read or mutate mutable objects in-place; they are only required when **reassigning/rebinding** the variable name.

---

### Q10. What are Python's bitwise operators?
Python performs operations on integers at the binary bit level:
* `&` (Bitwise AND)
* `|` (Bitwise OR)
* `^` (Bitwise XOR)
* `~` (Bitwise NOT / Invert: `~x = -(x + 1)`)
* `<<` (Left Shift: multiplies by $2^n$)
* `>>` (Right Shift: floor divides by $2^n$)

```python
# 5 (0101) & 3 (0011) = 1 (0001)
result = 5 & 3  # 1
```

---

### Q11. What is chained comparison in Python?
Python evaluates expressions like `1 < x < 10` as `(1 < x) and (x < 10)`, where `x` is evaluated **only once**. This is more readable and efficient than writing separate boolean clauses. Works with all comparison operators (`==`, `!=`, `<`, `<=`, `>`, `>=`, `is`, `in`).

---

### Q12. What is operator precedence in Python?
Evaluation order from highest to lowest precedence:
1. Parentheses `(...)`
2. Subscripting/Slicing `x[i]`, Attribute access `x.attr`, Function calls `f()`
3. Exponentiation `**`
4. Unary `+`, `-`, `~`
5. Multiplication `*`, Division `/`, Floor Division `//`, Modulo `%`
6. Addition `+`, Subtraction `-`
7. Bitwise Shifts `<<`, `>>`
8. Bitwise AND `&`, XOR `^`, OR `|`
9. Comparisons, Identity & Membership (`==`, `!=`, `<`, `>`, `is`, `in`)
10. Boolean `not`, `and`, `or`
11. Conditional Ternary `if-else`
12. Walrus operator `:=` and Assignments `=`

---

### Q13. What is the walrus operator (`:=`, PEP 572)?
Introduced in Python 3.8, the assignment expression operator assigns a value to a variable within an expression and returns that value simultaneously, avoiding redundant computations:

```python
# Stream reading
while chunk := file.read(8192):
    process(chunk)

# Regex matching
if match := pattern.search(line):
    print(match.group(1))
```

---

### Q14. What is structural pattern matching (`match/case`, PEP 634)?
Introduced in Python 3.10, structural pattern matching evaluates data against structural patterns, class attributes, sequence lengths, and mapping keys with variable binding:

```python
def handle_event(event):
    match event:
        case {"type": "click", "position": (x, y)}:
            return f"Clicked at ({x}, {y})"
        case ["upload", filename, int(size)] if size < 10_000_000:
            return f"Uploading {filename}"
        case _:
            return "Unhandled event"
```

---

### Q15. What is the difference between `break`, `continue`, and `pass`?
* `break`: Immediately exits the innermost loop, skipping any attached `else:` clause.
* `continue`: Skips the rest of the current iteration and advances directly to the next iteration.
* `pass`: A null statement that acts as a syntactic placeholder where code is expected but no action is required.

---

### Q16. What do `help()`, `dir()`, `vars()`, and `id()` do?
* `help(obj)`: Invokes the interactive documentation viewer to display docstrings, signatures, and class methods.
* `dir(obj)`: Returns a sorted list of all valid attribute and method names on the object.
* `vars(obj)`: Returns the `__dict__` mapping for a module, class, or instance (or local variables when called without arguments).
* `id(obj)`: Returns the object's unique integer identifier for its lifetime (its actual CPython memory address).

---

## 2. Data Structures, Collections & Operators

### Q17. What is the difference between mutable and immutable objects?
* **Mutable**: Objects whose internal state can be changed in-place after creation without altering their `id()`. Examples: `list`, `dict`, `set`, `bytearray`.
* **Immutable**: Objects whose contents cannot be changed once created. Any modification returns a new object. Examples: `int`, `float`, `str`, `tuple`, `frozenset`, `bytes`. Only immutable objects are hashable.

---

### Q18. Why are strings immutable in Python?
1. **Hash Table Stability**: Strings are commonly used as dictionary keys. Immutability guarantees their hash value never changes.
2. **Thread Safety & Security**: Immutable strings can be shared across threads without locks or data race concerns.
3. **String Interning**: Allows CPython to reuse identical string literals across the entire runtime memory space.

---

### Q19. How do Python dictionaries work internally?
Modern Python (3.6+ / 3.7+ standard) uses a **compact hash table**:
* It separates storage into an `indices` sparse array and an `entries` dense table containing `(hash, key_ptr, value_ptr)`.
* Keys are hashed using `hash()`. Collisions are resolved using open addressing with pseudo-random perturbation probing.
* Average lookup, insertion, and deletion times are $O(1)$. It preserves insertion order while reducing memory overhead by ~20–25%.

```
Indices: [ None, 0, None, 1, None ]
Entries: [ (hash1, key1, val1), (hash2, key2, val2) ]
```

---

### Q20. What is the difference between `list`, `tuple`, and `array.array`?
* `list`: Dynamic, mutable, heterogeneous sequence storing pointers to objects. High flexibility, higher memory overhead.
* `tuple`: Static, immutable, heterogeneous sequence. Lower memory overhead, hashable (if all elements are hashable).
* `array.array`: Homogeneous, contiguous, mutable C-level array storing fundamental primitive numeric types (e.g., `int32`, `float64`). Highly memory-efficient for numerical data.

---

### Q21. What is the difference between `set` and `frozenset`?
* `set`: Mutable collection of unique hashable elements. Unhashable; cannot be used as a dictionary key or inside other sets.
* `frozenset`: Immutable version of a set. Hashable; can be used as a dictionary key or stored within sets.

---

### Q22. How do `append()`, `extend()`, and `insert()` differ for lists?
* `list.append(x)`: Appends object `x` as a single element to the end of the list ($O(1)$ amortized).
* `list.extend(iterable)`: Iterates over `iterable` and appends each element individually to the end ($O(k)$).
* `list.insert(i, x)`: Inserts element `x` at index `i`, shifting subsequent elements to the right ($O(N)$ linear time).

---

### Q23. What is the difference between `del`, `remove()`, and `pop()` for lists?
* `del lst[i]`: Removes the item at index `i` without returning it ($O(N)$).
* `lst.remove(val)`: Removes the **first occurrence** of value `val`. Raises `ValueError` if not found ($O(N)$).
* `lst.pop(i)`: Removes and **returns** the item at index `i` (default `-1`). $O(1)$ from the end; $O(N)$ from the middle.

---

### Q24. How does slicing work in Python?
Slicing syntax is `sequence[start:stop:step]`.
* `start` is inclusive; `stop` is exclusive; `step` controls stride and direction (`[::-1]` reverses the sequence).
* Slicing creates a **new shallow copy** of the sliced segment.

---

### Q25. What are list, dict, set comprehensions and generator expressions?
* **List Comprehension**: `[x**2 for x in data if x > 0]` -> Builds a new list in memory.
* **Dict Comprehension**: `{k: v for k, v in pairs if v is not None}` -> Builds a new dictionary.
* **Set Comprehension**: `{x % 5 for x in data}` -> Builds a new set (deduplicating values).
* **Generator Expression**: `(x**2 for x in data)` -> Returns a lazy iterator producing elements on demand with minimal memory footprint.

---

### Q26. What is the difference between `/` and `//`?
* `/` (**True Division**): Always returns a `float` (e.g., `8 / 2` evaluates to `4.0`).
* `//` (**Floor Division**): Divides operands and rounds down to the nearest mathematical floor integer (e.g., `7 // 2` is `3`, `-7 // 2` rounds to `-4`).

---

### Q27. What is Python's floating-point precision issue and how is it resolved?
Binary floating-point numbers follow IEEE 754 (64-bit double precision). Decimal numbers like `0.1` and `0.2` cannot be represented exactly in binary fractions, leading to rounding discrepancies (`0.1 + 0.2 != 0.3`).
* **Solution**: Use `decimal.Decimal` for exact decimal calculations (e.g., in financial applications) or `math.isclose(a, b)` for floating-point tolerance checks.

---

### Q28. What are `bytes`, `bytearray`, and `memoryview`?
* `bytes`: Immutable sequence of integers ($0 \le x \le 255$).
* `bytearray`: Mutable counterpart to `bytes`, supporting in-place modifications.
* `memoryview`: Implements Python's Buffer Protocol, allowing slicing and modifying binary memory buffers without copying underlying memory.

---

### Q29. What is the difference between `encode()` and `decode()`?
* `str.encode('utf-8')`: Converts Unicode character text (`str`) into raw bytes (`bytes`).
* `bytes.decode('utf-8')`: Converts raw bytes (`bytes`) back into a Unicode character string (`str`).

---

### Q30. What is string interning (`sys.intern`)?
String interning ensures that only a single instance of distinct immutable string objects is stored in memory. Python automatically interns identifiers and string literals. You can call `sys.intern(s)` manually on dynamic strings to speed up dictionary lookups and reduce memory usage.

---

## 3. Functions, Scopes, Closures & Functional Programming

### Q31. What does it mean that functions are "first-class objects"?
Functions are full objects in Python. They can be:
1. Assigned to variables.
2. Stored inside data structures (`list`, `dict`).
3. Passed as arguments to other functions.
4. Returned from other functions.

---

### Q32. What are `*args` and `**kwargs`?
* `*args`: Collects excess positional arguments into a `tuple`.
* `**kwargs`: Collects excess keyword arguments into a `dict`.
* Standard parameter ordering: `def func(pos, *args, kw_only, **kwargs): ...`

---

### Q33. What are positional-only (`/`) and keyword-only (`*`) parameters?
* **Positional-Only (`/`, PEP 570)**: Parameters placed before `/` must be passed by position, not by keyword.
* **Keyword-Only (`*`)**: Parameters placed after `*` must be passed by name (keyword), improving call-site clarity.

```python
def setup(path, /, mode="fast", *, timeout=30):
    pass

setup("config.json", "safe", timeout=60)  # Valid
# setup(path="config.json")               # TypeError (positional-only)
# setup("config.json", "safe", 60)        # TypeError (timeout is keyword-only)
```

---

### Q34. Why is using a mutable default argument dangerous?
Default parameter expressions are evaluated **once at function definition time**, not on each function call. If a mutable object (like a list or dict) is used as a default, all subsequent function calls share that exact same instance in memory.
* **Fix**: Default the parameter to `None` and instantiate the mutable object inside the function body.

```python
def append_item(val, target=None):
    if target is None:
        target = []
    target.append(val)
    return target
```

---

### Q35. What is a closure in Python?
A closure is an inner function that captures and remembers variables from its enclosing scope even after the outer enclosing function has completed execution and returned.

```python
def make_multiplier(factor):
    def multiplier(n):
        return n * factor  # 'factor' is captured in the closure
    return multiplier

double = make_multiplier(2)
print(double(5))  # 10
```

---

### Q36. What is late binding in Python closures?
Python resolves captured variables in closures when the inner function is **invoked**, not when it is created.

```python
# The loop trap:
funcs = [lambda: i for i in range(3)]
print([f() for f in funcs])  # Outputs [2, 2, 2]

# Solution: Bind early via default argument
funcs = [lambda i=i: i for i in range(3)]
print([f() for f in funcs])  # Outputs [0, 1, 2]
```

---

### Q37. What is a decorator and how does it work?
A decorator is a callable that takes a function (or class), wraps it to extend or modify its behavior without changing its source code, and returns the wrapper.
* Always wrap the inner function with `@functools.wraps(func)` to preserve function metadata (`__name__`, `__doc__`).

```python
import functools
import time

def timer(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        t0 = time.perf_counter()
        result = func(*args, **kwargs)
        print(f"{func.__name__} took {time.perf_counter() - t0:.4f}s")
        return result
    return wrapper
```

---

### Q38. What is memoization and how is it implemented?
Memoization caches function return values based on their arguments to prevent redundant recalculations.
* In Python, use `@functools.lru_cache(maxsize=128)` or `@functools.cache` (Python 3.9+ unbounded cache).

```python
import functools

@functools.cache
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)
```

---

### Q39. What are lambda functions?
Lambdas are anonymous, single-expression functions defined via `lambda x, y: x + y`. They cannot contain statements or annotations and are primarily used as short callbacks in `sorted()`, `map()`, and `filter()`.

---

## 4. Object-Oriented Programming (OOP), Metaprogramming & Descriptors

### Q40. What is the difference between `__init__` and `__new__`?
* `__new__(cls, *args, **kwargs)`: The **constructor**. A static method called to allocate and return a new instance of `cls`. It creates the object. Required when subclassing immutable types or creating Singletons.
* `__init__(self, *args, **kwargs)`: The **initializer**. Called after `__new__` creates the instance to initialize its attributes. Returns `None`.

---

### Q41. What is the difference between `__str__` and `__repr__`?
* `__repr__(self)`: Developer-facing, unambiguous representation used for debugging. Should ideally return a valid Python expression that can recreate the object.
* `__str__(self)`: User-facing, readable string representation used by `print()` and `str()`. Falls back to `__repr__` if omitted.

---

### Q42. What is the difference between `@classmethod` and `@staticmethod`?
* `@classmethod`: Receives the class (`cls`) as its first argument. Can access and modify class state; commonly used for alternative factory constructors (`cls.from_dict()`).
* `@staticmethod`: Receives no implicit first argument (neither `self` nor `cls`). Functions as a plain helper function namespaced inside the class.

---

### Q43. What is Method Resolution Order (MRO) and C3 Linearization?
MRO defines the search path Python follows to locate attributes and methods in multiple inheritance hierarchies.
* Python uses the **C3 Linearization Algorithm** to preserve local precedence order and monotonicity.
* `super()` calls the *next class in the MRO chain*, ensuring cooperative multiple inheritance without diamond-dependency bugs. Inspect via `Class.mro()`.

---

### Q44. What is the purpose of `__slots__`?
By default, Python stores instance attributes in a dictionary (`self.__dict__`). Defining `__slots__ = ('name', 'age')` replaces `__dict__` with a fixed-size array of attribute pointers:
* Significantly reduces memory usage per instance (40–60% reduction).
* Speeds up attribute access.
* Prevents dynamic addition of undeclared attributes at runtime.

---

### Q45. What are Property Decorators (`@property`)?
`@property` exposes a method like a normal attribute, supporting getter, setter (`@prop.setter`), and deleter (`@prop.deleter`) interfaces for encapsulation, validation, and computed properties.

```python
class Account:
    def __init__(self, balance):
        self._balance = balance

    @property
    def balance(self):
        return self._balance

    @balance.setter
    def balance(self, value):
        if value < 0:
            raise ValueError("Balance cannot be negative")
        self._balance = value
```

---

### Q46. What is a Descriptor in Python?
A descriptor is an object attribute whose access behavior is overridden by methods in the descriptor protocol:
* `__get__(self, instance, owner)`
* `__set__(self, instance, value)`
* `__delete__(self, instance)`
* **Data Descriptor**: Implements both `__get__` and `__set__` (takes precedence over instance `__dict__`).
* **Non-Data Descriptor**: Implements only `__get__`. Descriptors power `@property`, `@classmethod`, `@staticmethod`, and ORM field mappings.

---

### Q47. What is the difference between `__getattr__` and `__getattribute__`?
* `__getattribute__(self, name)`: Intercepts **every** attribute access unconditionally. Careless overrides cause infinite recursion; always call `super().__getattribute__(name)` inside it.
* `__getattr__(self, name)`: Fallback method called **only** when the attribute does not exist in the object's dictionary or class hierarchy.

---

### Q48. What is Name Mangling?
When an attribute starts with two leading underscores and at most one trailing underscore (e.g., `__secret`), Python renames it internally to `_ClassName__secret`. This prevents accidental attribute overriding in derived subclasses.

---

### Q49. What is a Metaclass?
A metaclass is the "class of a class". It controls how classes themselves are constructed.
* The default metaclass is `type`.
* Custom metaclasses inherit from `type` and override `__new__` or `__init__` to validate class APIs, auto-register subclasses, and build declarative models (e.g., Django and Pydantic models).

---

### Q50. What is a Mixin class?
A mixin is a class designed to supply specific reusable methods to other classes via multiple inheritance without maintaining independent instance state or being instantiated on its own.

---

## 5. Memory Management, CPython Internals & the GIL

### Q51. How does Python manage memory?
Memory management in CPython is organized into multiple layers:
1. **Raw Memory Allocator**: Interacts directly with the OS virtual memory manager (`malloc`, `free`).
2. **PyMalloc (Small Object Allocator)**: Specialized allocator for objects $\le 512$ bytes, organized into **Arenas** (256 KB), **Pools** (4 KB), and **Blocks**.
3. **Reference Counting**: Primary mechanism for immediate memory deallocation.
4. **Generational Cyclic Garbage Collector**: Detects and reclaims circular references.

---

### Q52. How does Reference Counting work?
Every Python object (`PyObject`) embeds an `ob_refcnt` field.
* Increments when assigned to a variable, passed to a function, or stored in a container (`Py_INCREF`).
* Decrements when a variable goes out of scope or is deleted (`Py_DECREF`).
* As soon as `ob_refcnt == 0`, its memory is freed immediately.

---

### Q53. How does the Generational Cyclic Garbage Collector work (`gc` module)?
Reference counting cannot collect objects with circular references (e.g., `A.ref = B` and `B.ref = A`).
* CPython groups objects into three generations (Gen 0, Gen 1, Gen 2).
* New objects start in **Gen 0**. When Gen 0 exceeds an allocation threshold, a collection cycle runs using trial deletion to detect and free isolated cyclic graphs.
* Surviving objects are promoted to older generations. Older generations are collected less frequently to maintain performance.

---

### Q54. What is the Global Interpreter Lock (GIL)?
The **GIL** is a mutual exclusion lock in CPython ensuring that only one native OS thread executes Python bytecode at any given moment.
* **Why it exists**: Protects CPython's reference counts and internal C data structures from race conditions without requiring fine-grained locks across all objects.
* **Impact**: CPU-bound tasks do not achieve multi-core parallel speedups with standard `threading`. However, I/O-bound tasks release the GIL during network and file operations, making threads effective for I/O concurrency.

---

### Q55. What is Python's Free-Threaded (No-GIL) build (PEP 703, Python 3.13+)?
Introduced experimentally in Python 3.13 (`python3.13t`), the free-threaded build removes the GIL:
* Replaces global locks with **biased reference counting**, **mimalloc** (a thread-safe scalable allocator), and fine-grained locking.
* Enables true multi-threaded parallel execution across CPU cores within a single Python process.

---

### Q56. What is the difference between a shallow copy and a deep copy?
* **Assignment (`b = a`)**: Points both variables to the same object in memory (no copy made).
* **Shallow Copy (`copy.copy(a)` or `a.copy()`)**: Creates a new outer container, but inserts references to the original nested objects.
* **Deep Copy (`copy.deepcopy(a)`)**: Recursively duplicates the outer container and all nested objects, creating a completely independent copy.

---

## 6. Concurrency, Multithreading, Multiprocessing & AsyncIO

### Q57. How do Multithreading, Multiprocessing, and AsyncIO compare?

| Feature | Multithreading (`threading`) | Multiprocessing (`multiprocessing`) | Asynchronous (`asyncio`) |
| :--- | :--- | :--- | :--- |
| **Model** | Preemptive threads, shared memory | Preemptive processes, isolated memory | Cooperative event loop, 1 thread |
| **Best For** | I/O-bound tasks | CPU-bound computation | High-concurrency I/O (Web/Sockets) |
| **GIL Constraint** | Constrained by GIL | Bypasses GIL (separate processes) | Runs on 1 thread (GIL is non-issue) |
| **Overhead** | Low memory, OS context switches | High memory, IPC serialization cost | Minimal memory, fast context switch |
| **Communication** | Shared memory (requires locks) | Queues, Pipes, Shared Memory | In-memory event loop queues |

---

### Q58. How do `async`/`await` and the `asyncio` event loop work?
* An `async def` function defines a **coroutine**.
* An **Event Loop** manages and schedules coroutines on a single thread.
* `await` pauses execution of the coroutine during an I/O operation and yields control back to the event loop, allowing other coroutines to run concurrently while waiting for I/O.

---

### Q59. What is the difference between a Coroutine object and an asyncio `Task`?
* **Coroutine**: An inactive object returned by calling an `async def` function. Does not run until explicitly awaited or scheduled.
* **Task (`asyncio.create_task(coro)`)**: Wraps a coroutine and immediately schedules it to run concurrently on the event loop in the background.

---

### Q60. What is the difference between `asyncio.gather()` and `asyncio.wait()`?
* `asyncio.gather(*coros)`: Runs coroutines concurrently and returns an **ordered list** of results matching the input sequence. Propagates exceptions immediately unless `return_exceptions=True`.
* `asyncio.wait(tasks)`: Returns two sets: `(done_tasks, pending_tasks)`. Provides lower-level control over partial completions, timeouts, and cancellations.

---

### Q61. What is `concurrent.futures`?
Provides a high-level API for running asynchronous callables across pools:
* `ThreadPoolExecutor`: Distributes tasks to worker threads (ideal for I/O-bound workloads).
* `ProcessPoolExecutor`: Distributes tasks to worker processes (bypasses GIL for CPU-bound computations).

```python
from concurrent.futures import ProcessPoolExecutor

def compute(x):
    return x * x

with ProcessPoolExecutor() as executor:
    results = list(executor.map(compute, [1, 2, 3, 4, 5]))
```

---

### Q62. What limitations does pickling place on `multiprocessing`?
Because worker processes run in separate address spaces, all function arguments, return values, and shared objects must be serializable via `pickle`. Unpicklable objects (such as open file handles, database connections, and lambda functions) cannot be passed to worker processes directly.

---

### Q63. What are `contextvars` (PEP 567)?
`contextvars` manages context-local state. Unlike `threading.local` (which is isolated per thread), a `ContextVar` maintains isolated state per asynchronous task/coroutine, preventing request-scoped data (like trace IDs or auth tokens) from leaking across concurrent coroutines.

---

## 7. Modern Type Hinting, Protocols & Static Analysis

### Q64. What are Python Type Hints (PEP 484)?
Type hints annotate variables and function signatures with expected types. They are not enforced at runtime by the interpreter, but enable static analysis tools (`mypy`, `pyright`), improve IDE autocompletion, and enhance maintainability.

```python
def find_user(user_id: int) -> dict[str, str] | None: ...
```

---

### Q65. What is `TypeVar`?
`TypeVar` defines generic type placeholders to preserve relationships between function arguments and return types:

```python
from typing import TypeVar, Sequence

T = TypeVar('T')

def get_first(items: Sequence[T]) -> T:
    return items[0]
```

---

### Q66. What is a `Protocol` (PEP 544)?
A `Protocol` enables **structural subtyping** (static duck typing). Any class that implements the methods and attributes defined in a `Protocol` satisfies that type without needing explicit inheritance.
* `@typing.runtime_checkable` allows runtime `isinstance()` checks against the protocol.

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class Serializable(Protocol):
    def serialize(self) -> str: ...
```

---

### Q67. What is `TypedDict` vs `NamedTuple` vs `dataclass`?
* `TypedDict`: A static type hint defining expected keys and value types for standard dictionaries. Has no runtime performance cost.
* `NamedTuple`: Creates an immutable `tuple` subclass with named fields and positional index access.
* `dataclass`: Generates boilerplate methods (`__init__`, `__repr__`, `__eq__`) for data-holding classes using type annotations.

---

### Q68. What is `@dataclass(slots=True)` and `field(default_factory=...)`?
* `slots=True` (Python 3.10+): Automatically generates `__slots__` on the dataclass, removing instance `__dict__` overhead.
* `default_factory=list`: Instantiates a fresh mutable container for each object, avoiding shared mutable default bugs.
* `__post_init__`: Hook called after `__init__` for post-creation validation or computed field initialization.

---

### Q69. What are `ParamSpec` and `Concatenate` (PEP 612)?
Advanced typing tools for typing decorators:
* `ParamSpec`: Captures arbitrary parameter signatures (positional and keyword) of a wrapped callable.
* `Concatenate`: Injects or prepends additional parameters to a captured `ParamSpec`.

---

## 8. Error Handling, Context Managers & Exception Groups

### Q70. What is the full `try / except / else / finally` flow?
1. `try`: Wraps code that might raise an exception.
2. `except SpecificError as e`: Catches and handles matching exceptions (evaluated top to bottom).
3. `else`: Runs **only** if no exception occurred in the `try` block.
4. `finally`: Runs **unconditionally** for cleanup (even after unhandled exceptions or early `return` statements).

---

### Q71. What is Exception Chaining (`raise ... from ...`)?
* `raise NewException() from original`: Sets `__cause__` on the new exception, explicitly preserving the original exception context.
* `raise NewException() from None`: Suppresses the contextual traceback (`__context__`), hiding internal implementation details from users.

---

### Q72. What are Exception Groups and `except*` (PEP 654, Python 3.11+)?
`ExceptionGroup` wraps multiple co-occurring exceptions into a single composite error object.
* `except* Type` enables selective handling of specific exception types within the group without swallowing other errors. Used with concurrent tasks (`asyncio.TaskGroup`).

```python
try:
    async with asyncio.TaskGroup() as tg:
        tg.create_task(job_one())
        tg.create_task(job_two())
except* ValueError as eg:
    print(f"Handled ValueErrors: {eg.exceptions}")
except* ConnectionError as eg:
    print(f"Handled ConnectionErrors: {eg.exceptions}")
```

---

### Q73. How do you write a custom Context Manager?
1. **Class-Based**: Implement `__enter__()` and `__exit__(exc_type, exc_val, exc_tb)`.
2. **Generator-Based (`@contextlib.contextmanager`)**:

```python
from contextlib import contextmanager

@contextmanager
def open_managed_resource():
    res = acquire_resource()
    try:
        yield res
    finally:
        res.cleanup()
```

---

### Q74. What is `contextlib.suppress()`?
A utility context manager that suppresses specified exception types cleanly in a single line, replacing `try/except: pass` blocks:

```python
from contextlib import suppress
import os

with suppress(FileNotFoundError):
    os.remove("temporary.lock")
```

---

## 9. Python Standard Library Power Tools

### Q75. What are the key utilities in `itertools`?
* `itertools.chain(*iterables)`: Chains multiple iterables into a single sequence.
* `itertools.cycle(iterable)`: Cycles infinitely through an iterable.
* `itertools.islice(iterable, start, stop, step)`: Lazily slices an iterator without realizing it in memory.
* `itertools.groupby(iterable, keyfunc)`: Groups consecutive elements sharing identical keys.
* `itertools.accumulate(iterable, func=operator.add)`: Generates running totals / cumulative reductions.
* `itertools.product()`, `permutations()`, `combinations()`: High-performance combinatorics.

---

### Q76. What are the key utilities in `functools`?
* `functools.reduce(func, seq)`: Cumulatively reduces a sequence to a single value.
* `functools.partial(func, *args, **kwargs)`: Freezes a portion of function arguments, returning a new callable with a smaller signature.
* `functools.singledispatch`: Transforms a function into a polymorphic single-dispatch generic function based on the type of its first argument.
* `functools.cached_property`: Caches the result of an expensive property method in the instance's `__dict__`.

---

### Q77. What are the key data structures in `collections`?
* `Counter`: A dictionary subclass for counting hashable objects.
* `defaultdict(factory)`: A dictionary that invokes a default factory when querying non-existent keys.
* `deque`: A doubly-linked double-ended queue with $O(1)$ appends and pops from both ends.
* `OrderedDict`: Dict subclass preserving ordering (provides specialized methods like `move_to_end()`).

---

### Q78. What is the `heapq` module?
Implements a binary **min-heap** over a standard Python list:
* `heapq.heappush(heap, item)` ($O(\log N)$)
* `heapq.heappop(heap)` ($O(\log N)$, pops smallest element)
* `heapq.heapify(list)` ($O(N)$ in-place heap construction)
* `heapq.nlargest(k, iterable)` / `heapq.nsmallest(k, iterable)`

---

### Q79. What is `pathlib.Path` vs `os.path`?
`pathlib` (Python 3.4+) offers an object-oriented, cross-platform interface for filesystem operations. It replaces error-prone string manipulation with the `/` path separator operator, provides `.read_text()`, `.write_bytes()`, `.exists()`, and `.glob()`, handling OS differences (Unix `/` vs Windows `\`) seamlessly.

---

### Q80. What is the difference between `random` and `secrets`?
* `random`: Pseudo-random number generator based on the Mersenne Twister algorithm ($2^{19937}-1$ period). Fast and deterministic, but **insecure** (state can be reverse-engineered). Suitable for simulations and games.
* `secrets` (Python 3.6+): Uses cryptographically secure random primitives provided by the operating system (`/dev/urandom` / `CryptGenRandom`). Mandatory for generating security tokens, session IDs, and passwords.

---

## 10. Testing, Debugging, Profiling & Optimization

### Q81. How does `pytest` differ from `unittest`?
* `unittest`: Built-in xUnit-style framework requiring test classes inheriting from `unittest.TestCase` and specialized assertion methods (`self.assertEqual(a, b)`).
* `pytest`: Third-party framework allowing plain Python functions and native `assert` statements. Features powerful modular dependency-injection via `@pytest.fixture`, parameterized testing (`@pytest.mark.parametrize`), extensive plugin support, and automatic test discovery.

---

### Q82. What is `unittest.mock` and `patch()`?
`unittest.mock` provides mock objects (`Mock`, `MagicMock`) that simulate external dependencies (databases, networks, filesystem) and assert call invocations. `patch()` acts as a function/context decorator that temporarily replaces target objects with mock instances during tests.

---

### Q83. How do you profile Python applications?
* **Deterministic Profiling (`cProfile`)**: Measures exact function call counts and cumulative execution time:
  ```bash
  python -m cProfile -s cumulative app.py
  ```
* **Line-by-line Profiling (`line_profiler`)**: Identifies execution bottlenecks line by line inside specific functions.
* **Micro-benchmarking (`timeit`)**: Accurately measures execution timing for small code snippets by repeating runs across millions of loops.

---

### Q84. How do you reduce memory usage in Python applications?
1. Use **generators** and generator expressions for lazy evaluation.
2. Define **`__slots__`** on classes with high instance counts.
3. Use **`memoryview`** and the buffer protocol to avoid copying large binary data.
4. Process tabular data with vectorized contiguous arrays (**NumPy / Polars**).
5. Explicitly clear large data structures (`del data`) and invoke `gc.collect()` where necessary.

---

## 11. Packaging, Virtual Environments & Dependency Management

### Q85. What is the difference between `venv`, `virtualenv`, and `conda`?
* `venv`: Built-in lightweight tool using the current interpreter to generate isolated directory environments.
* `virtualenv`: Third-party predecessor to `venv` with extended capabilities (supports multiple Python versions, faster virtual environment creation).
* `conda`: Language-agnostic package and environment manager capable of installing C-libraries, binaries, and distinct Python interpreter versions.

---

### Q86. What is the difference between a Wheel (`.whl`) and an Sdist?
* **Sdist (Source Distribution)**: An archive (`.tar.gz`) containing raw source code and setup scripts. Installing it requires compilation on the target machine if C extensions are present.
* **Wheel (`.whl`, PEP 427)**: A pre-compiled, zip-formatted binary package format. Installs rapidly by moving files into `site-packages` without requiring a compiler toolchain on the target host.

---

### Q87. What is `pyproject.toml` (PEP 518 / PEP 621)?
`pyproject.toml` is the modern unified configuration standard for Python projects. It replaces legacy `setup.py`, `setup.cfg`, `MANIFEST.in`, and `requirements.txt`, centralizing build tool specifications (e.g., Hatch, Flit, Poetry, setuptools) and project metadata (name, version, dependencies, linter configs) into a single declarative file.

---

### Q88. What is `PYTHONPATH` and how does module resolution work?
`PYTHONPATH` is an environment variable containing custom directory paths searched during imports. Resolution order:
1. Current working directory of the executed script.
2. Standard library modules.
3. Paths listed in `PYTHONPATH`.
4. Installed third-party `site-packages` (managed by `sys.path`).

---

### Q89. What causes circular imports and how do you resolve them?
A circular import occurs when Module A imports Module B, and Module B imports Module A during initial module execution. When Module B tries to access uninitialized attributes of Module A, an `ImportError` or `AttributeError` occurs.
* **Solutions**:
  1. Refactor shared dependencies into a separate third module.
  2. Use **deferred/local imports** (import inside a function rather than at the module top level).
  3. Use type-only imports guarded by `if typing.TYPE_CHECKING:`.

---

## 12. Software Architecture, Design Patterns & Enterprise API Design

### Q90. How do you implement the Singleton Pattern cleanly in Python?
While module imports are natural singletons in Python, a class-based Singleton is implemented via `__new__`:

```python
class Singleton:
    _instance = None

    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
```

---

### Q91. What is Dependency Injection (DI) and how is it used in Python?
Dependency Injection is a design pattern where an object receives its dependencies from external callers rather than instantiating them internally. This decouples components, simplifies mocking, and improves testability:

```python
class PaymentProcessor:
    def __init__(self, gateway: PaymentGateway):
        self.gateway = gateway  # Injected dependency

    def process(self, amount: float):
        return self.gateway.charge(amount)
```

---

### Q92. How do you design an enterprise-grade REST API in Python?
1. **Framework**: Utilize **FastAPI** for high-performance ASGI async execution and automatic OpenAPI documentation.
2. **Data Validation**: Enforce request/response schemas using **Pydantic** models.
3. **Layered Architecture**: Separate logic into API Routers $\to$ Service Layer (business logic) $\to$ Repository Layer (database access via SQLAlchemy/Tortoise ORM).
4. **Resilience**: Implement structured logging, central exception handlers, JWT authentication, rate limiting, and health check endpoints.

---

## 13. Essential Coding & Algorithmic Problem-Solving Patterns

### Q93. How do you check if a string is a palindrome?
```python
def is_palindrome(s: str) -> bool:
    cleaned = [c.lower() for c in s if c.isalnum()]
    return cleaned == cleaned[::-1]
```

---

### Q94. How do you check whether two strings are anagrams?
```python
from collections import Counter

def is_anagram(s1: str, s2: str) -> bool:
    return Counter(s1) == Counter(s2)
```

---

### Q95. How do you find the first non-repeating character in a string?
```python
from collections import Counter

def first_unique_char(s: str) -> str | None:
    counts = Counter(s)
    for ch in s:
        if counts[ch] == 1:
            return ch
    return None
```

---

### Q96. How do you merge two dictionaries in Python?
```python
# Modern Python 3.9+ Union Operator:
merged = dict_a | dict_b  # Values in dict_b overwrite dict_a

# Dictionary Unpacking:
merged = {**dict_a, **dict_b}
```

---

### Q97. How do you flatten a deeply nested list of arbitrary depth?
```python
from typing import Any, Iterable

def flatten(nested: Iterable[Any]) -> list[Any]:
    result = []
    for item in nested:
        if isinstance(item, Iterable) and not isinstance(item, (str, bytes)):
            result.extend(flatten(item))
        else:
            result.append(item)
    return result
```

---

### Q98. How do you find the largest and second-largest numbers in an array in $O(N)$ time?
```python
def find_top_two(nums: list[int]) -> tuple[int, int]:
    first = second = float('-inf')
    for n in nums:
        if n > first:
            second, first = first, n
        elif n > second and n != first:
            second = n
    return first, second
```

---

### Q99. How do you implement Binary Search in Python?
```python
def binary_search(arr: list[int], target: int) -> int:
    low, high = 0, len(arr) - 1
    while low <= high:
        mid = (low + high) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            low = mid + 1
        else:
            high = mid - 1
    return -1
```

---

### Q100. How do you check if a number is prime in $O(\sqrt{N})$?
```python
def is_prime(n: int) -> bool:
    if n <= 1:
        return False
    if n <= 3:
        return True
    if n % 2 == 0 or n % 3 == 0:
        return False
    i = 5
    while i * i <= n:
        if n % i == 0 or n % (i + 2) == 0:
            return False
        i += 6
    return True
```
