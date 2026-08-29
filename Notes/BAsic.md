# 🐍 Python Complete Course — Revision Notes

> Quick-reference README distilled from the full Python course (foundation for AI & Agentic AI). Use this to revise concepts fast before interviews or before diving into AI/agentic development.

---

## 1. What is Python?

- Python is an **interpreted, high-level language**. It runs on top of the **Python Virtual Machine (PVM)**, which interprets and executes code **line by line**.
- Being line-by-line makes debugging easy (you can execute a single line and see the result immediately).
- **High-level** = lots of abstraction, easy to write.
- **Platform independent** — runs on Windows/Linux/Mac because of its own virtual machine.

### When to use Python vs other languages
- Use Python when execution latency / memory footprint isn't performance-critical (general apps, AI/LLM orchestration, scripting).
- Use **C++** for performance-critical systems like game engines — those compile directly to hardware, no interpreter in between.

### Python & LLMs — important mental model
- Python does **not** run the LLM itself. Python is mostly the **orchestrator**: managing chat history, context, memory, calling the model via SDKs (Flask, FastAPI, Transformers, LangChain, etc.).
- Actual LLM computation happens on **GPUs using CUDA kernels**, using libraries like PyTorch/TensorFlow to train/optimize the model.
- Building an LLM = CUDA + GPU + training algorithms + data. Using an LLM in an agentic app = Python.

---

## 2. Installation & Setup

- Download from **python.org** (latest version at recording time: 3.14.6).
- Windows/Linux: may need to add Python to **environment variables (PATH)**.
- Check install: `python --version` (or `python3 --version` on Mac).
- **pip** = Python's package manager (like `npm` for Node). Install a library:
  ```bash
  pip install <library-name>
  ```

### How a `.py` file runs internally
1. Source code (`.py`) is read.
2. Tokenized → syntax tree built (parsing).
3. Converted into **bytecode**.
4. Bytecode is cached as a `.pyc` file (inside `__pycache__`), then executed by the PVM.
5. If nothing changed in a file, Python reuses the cached `.pyc` instead of recompiling — saves time on large projects.

Manual compile: `python -m py_compile hello.py` → creates `__pycache__/hello.cpython-314.pyc`
Run compiled file: `python __pycache__/hello.cpython-314.pyc`

### CPython
- **CPython** = the reference interpreter/compiler of Python, written in **C** (not C++).
- You *can* write Python extensions in C++ for speed (e.g., pybind11, Boost.Python).
- Internals: everything is a **PyObject**, stored on the **heap**, memory managed via **reference counting** + generational garbage collection.
- The **GIL (Global Interpreter Lock)** ensures only one thread executes Python bytecode at a time.

---

## 3. Basics: print, input, comments

```python
print("Hello")          # simple print
name = input("Name? ")  # takes input, always returns a string
print(name)
```

- **Comments**: `#` for single line; `''' ... '''` or `""" ... """` for multi-line (also used as docstrings, shown on hover in VS Code).
- **Indentation matters** — Python has **no curly braces `{}`** and **no semicolons `;`**. Blocks are defined purely by indentation. Missing indentation → `IndentationError`.
- Python is **case-sensitive**.

---

## 4. Variables & Type Hints

```python
x: int = 10      # this is just an annotation — Python does NOT enforce it!
x = "hello"      # this still works, no error
```

- Type annotations are **ignored at runtime** (Python is not like TypeScript).
- To **strictly enforce** a type:
  ```python
  if not isinstance(x, int):
      raise TypeError("x must be int")
  ```
- Libraries like **Pydantic** or `TypeGuard` can help enforce/validate types.
- Python is **dynamically typed** — a variable can be reassigned to any type at any time.

---

## 5. Data Types Overview

| Category | Types |
|---|---|
| Numeric | `int`, `float`, `complex` (e.g. `3+4j`) |
| Text | `str` |
| Boolean | `bool` (`True` / `False`, capitalized) |
| Sequence | `list`, `tuple`, `range` |
| Mapping | `dict` |
| Set | `set`, `frozenset` |
| Binary | `bytes`, `bytearray`, `memoryview` |
| Special | `NoneType` (`None`) |

### Truthy / Falsy values
**Falsy:** `0`, `0.0`, `False`, `None`, `""`, `[]`, `{}`, `()`, `set()`
**Truthy:** anything else (non-zero numbers, non-empty collections/strings)

```python
print(bool([]))    # False
print(bool([1]))   # True
```

### `None`
- Special reserved keyword — Python's equivalent of `null`/`nil`.
- `type(None)` → `NoneType`.
- `None` is **not** `0`, **not** `False`, **not** `""` — it's a completely separate concept meaning "no value."
- A function with no explicit `return` returns `None`.

---

## 6. Strings

- **Immutable** — you cannot change a character in place; you can only create a new string.
  ```python
  name = "Ritik"
  # name[0] = "R"  ❌ TypeError: doesn't support item assignment
  name = "Robot"   # ✅ full reassignment works
  ```

### Slicing
Syntax: `string[start:end:step]` — **`end` is exclusive** (goes up to `end - 1`).
```python
text = "Ritik"
text[0:3]     # "Rit"  (0,1,2 — index 3 excluded)
text[::-1]    # reversed string
text[::2]     # every 2nd character
```

### Common string methods
| Method | Purpose |
|---|---|
| `.upper()` / `.lower()` | change case |
| `.strip()` / `.lstrip()` / `.rstrip()` | trim whitespace |
| `.capitalize()` | first letter capital |
| `.title()` | capitalize each word |
| `.swapcase()` | flip upper/lower |
| `.replace(old, new)` | replace substring |
| `.find()` / `.rfind()` | index of substring (leftmost/rightmost), returns `-1` if not found |
| `.index()` / `.rindex()` | like find, but **raises an error** if not found |
| `.split(sep)` | split into a list |
| `.join(iterable)` | join list into string with separator |
| `.partition(sep)` | returns a tuple `(before, sep, after)` |
| `in` / `not in` | membership check |
| `.isalnum()`, `.isalpha()`, `.isdigit()`, `.istitle()`, etc. | boolean checks |
| `.encode()` | convert string → bytes |

### f-strings (Python 3+, modern standard)
```python
name = "Ritik"
price = 35.456
print(f"My name is {name}")
print(f"{price:.2f}")     # 35.46 → 2 decimal places
print(f"{5:05}")          # zero-padded: 00005
print(f"{'x':<10}")       # left align
print(f"{'x':>10}")       # right align
print(f"{'x':^10}")       # center align
```

### Concatenation
```python
a + " " + b     # manual space
print(a, b)     # comma auto-adds a space
```

---

## 7. Booleans

```python
True + True     # 2  (True behaves like 1)
False + 5       # 5  (False behaves like 0)
bool(0)         # False
bool(3)         # True
not True        # False   (Python uses `not`, not `!`)
```

---

## 8. Collections: List, Tuple, Set, Dictionary

### Comparison table
| Type | Ordered | Mutable | Duplicates | Indexing |
|---|---|---|---|---|
| **List** `[]` | ✅ | ✅ | ✅ | 0,1,2... |
| **Tuple** `()` | ✅ | ❌ | ✅ | 0,1,2... |
| **Set** `{}` | ❌ (no guaranteed order) | ✅ | ❌ | ❌ |
| **Dictionary** `{k:v}` | ✅ (since 3.7) | ✅ | Keys must be unique | By key |
| **String** | ✅ | ❌ | ✅ | 0,1,2... |

### List
```python
lst = [1, "Ritik", 3.5]     # can mix data types
print(*lst)                 # unpack & print without brackets
lst.append(x)                # add single item at end
lst.extend([a, b])           # merge another list's items
lst.insert(index, value)
lst.remove(value)            # removes by value (first match)
lst.pop()                    # removes & returns last item (or by index)
lst.index(value)
lst.count(value)
lst.clear()
sorted(lst)                  # returns NEW sorted list
lst.sort()                   # sorts IN PLACE, returns None
lst.reverse()                # reverses IN PLACE
reversed(lst)                # returns an iterator
lst * 3                      # repeat elements
```

**Shallow vs Deep Copy — important interview concept!**
```python
a = [1, 2]
b = a              # b points to SAME memory as a → changes to b affect a
c = a.copy()       # shallow copy → new outer list, but nested objects still shared
d = copy.deepcopy(a)  # fully independent copy, no shared references at any level
```
- Shallow copy: outer list is new, but **nested/inner lists are still shared**.
- Deep copy: everything is duplicated, no shared references anywhere.

### Tuple
```python
t = (1, 2, 3)
single = (5,)        # ⚠️ need a trailing comma, else it's just an int, not a tuple!
```
- Ordered, **immutable** (safer for constant data), can be used as dictionary keys.
- To "update" a tuple: convert to list → modify → convert back to tuple.
- Packing/unpacking: `a, *b = [1,2,3,4]` → `a=1`, `b=[2,3,4]`

### Set
```python
s = {1, 2, 3}
s.add(x)
s.update([...])
s.remove(x)      # error if not present
s.discard(x)     # no error if not present
s.pop()          # removes a random element
a | b            # union
a & b            # intersection
a - b            # difference
a ^ b            # symmetric difference
x in s           # membership check — O(1) average, faster than list's O(n)
```
- `{}` alone creates an **empty dictionary**, not a set! Use `set()` for an empty set.
- `frozenset` = immutable version of a set.

### Dictionary
```python
d = {"a": 1, "b": 2}
d["a"]            # KeyError if missing
d.get("a")        # returns None if missing (safer)
d["c"] = 3        # add/update
d.update({...})
d.pop("a")        # remove by key
del d["a"]
d.clear()
d.keys()          # iterable of keys
d.values()        # iterable of values
d.items()         # iterable of (key, value) pairs
for k, v in d.items():
    print(k, v)
```

---

## 9. Identity vs Equality — `is` vs `==`

```python
a = [1, 2]
b = [1, 2]
a == b      # True  → same VALUES
a is b      # False → different memory locations
```

- `==` compares **values**.
- `is` compares whether two variables point to the **same memory location** (identity).
- **Small integers/short strings** are cached by Python (interning), so `c = 2; d = 2; c is d` → `True` — but this behavior shouldn't be relied on for logic!

---

## 10. Operators

| Category | Operators |
|---|---|
| Arithmetic | `+ - * / // % **` (`//` = floor division, `%` = modulus, `**` = power) |
| Comparison | `== != > < >= <=` (no `===` in Python) |
| Logical | `and`, `or`, `not` (not `&&`, `||`, `!`) |
| Bitwise | `& \| ^ ~ << >>` |
| Assignment | `= += -= *= /=` etc. (no `i++`, use `i += 1`) |
| Walrus `:=` | assign + use value in same expression |

```python
if (n := len([1,2,3])) > 2:
    print(n)     # assigns AND checks in one line
```

**Operator precedence**: `()` → `**` → `* / // %` → `+ -` → comparisons → `not/and/or` → assignment.

---

## 11. Conditionals

```python
if condition:
    ...
elif other_condition:
    ...
else:
    ...

# Ternary / conditional expression
label = "Even" if x % 2 == 0 else "Odd"

# Good practice: write `if x:` not `if x == True:`
```

### `match` / `case` (Python 3.10+, like switch-case)
```python
match status_code:
    case 200:
        print("OK")
    case 301 | 304:
        print("Redirect")
    case _:
        print("Unknown")   # `_` = default / placeholder
```

### `all()` and `any()`
```python
all([True, True, False])   # False — every element must be True
any([False, False, True])  # True  — at least one element True
```

### Underscore `_` as placeholder
```python
for _ in range(10):
    print("Hi")   # loop var not needed, so use _
```

---

## 12. Loops

```python
for item in some_list:
    print(item)

for i in range(start, stop, step):    # stop is EXCLUSIVE
    print(i)

for index, value in enumerate(some_list):
    print(index, value)

while condition:
    ...

break       # exits the loop entirely
continue    # skips rest of current iteration, moves to next
pass        # does nothing — a placeholder statement
```

---

## 13. Functions

```python
def add(a: int, b: int) -> int:
    return a + b
```

- Keyword `def` defines a function.
- **Positional arguments** — must be passed in correct order.
- **Keyword arguments** — pass by name, order doesn't matter:
  ```python
  greet(name="Ritik", age=10)
  ```
- **Default arguments**: `def greet(age=10):`
- **`*args`** — collects extra positional args into a tuple.
- **`**kwargs`** — collects extra keyword args into a dict.
- **Docstrings**:
  ```python
  def add(a, b):
      """Adds two numbers and returns the result."""
      return a + b
  print(add.__doc__)
  ```

### Global keyword & the LEGB rule
```python
x = 10
def update():
    global x
    x = 20   # without `global`, this creates a NEW local variable instead
```
- **LEGB rule** — variable lookup order: **L**ocal → **E**nclosing → **G**lobal → **B**uilt-in.
- Prefer **passing values in & returning values out** of functions over using `global` (better practice).

### Lambda (anonymous functions)
```python
square = lambda x: x * x
```
- Equivalent to a normal function but written in one line. Avoid overusing — hurts readability.

### Generators
```python
def counter():
    yield 1
    yield 2
    yield 3

gen = counter()
next(gen)   # 1, then 2, then 3
```
- Produce **one value at a time**, don't store everything in memory upfront.
- Ideal for large datasets, streams, infinite sequences.

### Decorators
```python
def decorator(func):
    def wrapper():
        print("Before")
        func()
        print("After")
    return wrapper

@decorator
def hello():
    print("Hello")

hello()
# Output: Before → Hello → After
```
- A decorator wraps a function to add extra behavior without modifying its code.

---

## 14. JSON

- **JSON** = JavaScript Object Notation — the universal data exchange format.

| Python type | JSON type |
|---|---|
| `dict` | Object |
| `list` / `tuple` | Array |
| `str` | String |
| `int` / `float` | Number |
| `True` / `False` | `true` / `false` |
| `None` | `null` |

```python
import json

json.dumps(python_obj)          # Python → JSON string
json.dumps(obj, indent=4)       # pretty-print
json.dumps(obj, sort_keys=True) # sorted keys
json.loads(json_string)         # JSON string → Python object

json.dump(obj, file)            # Python → JSON file
json.load(file)                 # JSON file → Python object
```

---

## 15. Exception Handling

```python
try:
    risky_code()
except ValueError as e:
    print(e)
finally:
    print("Always runs")
```

---

## 16. Object-Oriented Programming (OOP)

- OOP organizes code using **classes, objects, methods, attributes**.
- Four pillars: **Encapsulation, Inheritance, Polymorphism, Abstraction**.
- **Object** = an instance of a class.

```python
class Student:
    def __init__(self, name):     # constructor
        self.name = name           # instance attribute

    def show(self):
        print(self.name)

s1 = Student("Ritik")
s1.show()
```

### Why `self`?
- `self` refers to the **current object/instance** — it lets each object keep its own separate data.
- **`self` is NOT a reserved keyword** — it's just a strong convention. You *could* name it anything, but never do.
- Without `self`, a variable becomes just a **local variable** inside that method and disappears after the method ends — it won't be attached to the object.

### Static methods
```python
class MathUtils:
    @staticmethod
    def add(a, b):
        return a + b

MathUtils.add(2, 3)   # no need to create an instance
```

### Access modifiers (convention-based, not enforced)
```python
class Demo:
    def __init__(self):
        self.public_var = 1      # public
        self._protected_var = 2  # protected (single underscore)
        self.__private_var = 3   # private (double underscore)
```

### Dataclasses (boilerplate reducer)
```python
from dataclasses import dataclass

@dataclass
class User:
    name: str
    age: int
```
- Auto-generates `__init__` and more. `@dataclass(frozen=True)` makes it immutable.

---

## 17. Import / Export

```python
from dataclasses import dataclass   # import specific item from a module
import json
```

---

## 📝 Homework / Topics to Explore Further
- [ ] Regular expressions (`re` module)
- [ ] Virtual environments (`venv`) — create your own
- [ ] OOP deep-dive: **Inheritance**, **Method Overloading/Overriding**, **`super()`**, **Duck Typing**, **Abstraction**, **Composition/Aggregation**, **MRO (Method Resolution Order)**
- [ ] `property` decorator
- [ ] File handling (`os` module, reading/writing files)
- [ ] Multi-threading / multiprocessing (not covered in this course)
- [ ] Practice DSA (Data Structures & Algorithms) in Python

## 🎯 Common Interview Traps to Remember
- `{}` creates a dict, not a set — use `set()` for an empty set.
- `2 * True == 2` and `False + 5 == 5` (bool behaves like int).
- `a == b` checks value; `a is b` checks memory identity.
- Type annotations are **not enforced** by default in Python.
- Tuples need a trailing comma for single-element tuples: `(5,)`.
- `.find()` returns `-1` if not found; `.index()` raises an error instead.
- `sort()` is in-place (returns `None`); `sorted()` returns a new list.
- Slicing/range `end` is always **exclusive** (`end - 1` is the last included index).