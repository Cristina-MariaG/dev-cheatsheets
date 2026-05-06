# Python — Quick Reference

---

## Essentials

```python
# Unpack
a, b, *rest = [1, 2, 3, 4, 5]        # a=1, b=2, rest=[3,4,5]
first, *_, last = iterable            # first and last only

# Swap
a, b = b, a

# Ternary
x = value_if_true if condition else value_if_false

# Walrus operator (Python 3.8+) — assign and test in one
if (n := len(data)) > 10:
    print(f"Too many items: {n}")

# Safe attribute / item access
value = getattr(obj, "attr", default)
value = data.get("key", default)
```

---

## Classes — patterns at a glance

```python
class MyClass:
    class_var = []                              # shared

    def __init__(self, x):
        self.x = x                             # instance var

    def method(self):         ...              # self = instance
    @classmethod
    def from_x(cls, x):      ...              # cls = class, factory
    @staticmethod
    def helper(x):            ...              # no self/cls, utility

    @property
    def computed(self):       return self.x * 2
    @computed.setter
    def computed(self, v):    self.x = v // 2

    def __repr__(self):       return f"MyClass({self.x!r})"
    def __str__(self):        return str(self.x)
    def __eq__(self, other):  return self.x == other.x
    def __hash__(self):       return hash(self.x)
    def __len__(self):        return self.x
    def __contains__(self, v): return v == self.x
    def __iter__(self):       return iter([self.x])
    def __call__(self, *a):   return self.x
    def __enter__(self):      return self
    def __exit__(self, *exc): return False
```

---

## Inheritance summary

```python
class Child(Parent):
    def __init__(self):
        super().__init__()   # always call super first

# Multiple
class D(B, C): pass
D.__mro__                    # check resolution order

# Abstract
from abc import ABC, abstractmethod
class Base(ABC):
    @abstractmethod
    def run(self): ...

# Protocol (duck typing)
from typing import Protocol
class Runnable(Protocol):
    def run(self) -> None: ...
```

---

## Dataclass quick reference

```python
from dataclasses import dataclass, field

@dataclass
class User:
    name: str
    email: str
    tags: list[str] = field(default_factory=list)

@dataclass(frozen=True)    # immutable + hashable
class Point:
    x: float; y: float

@dataclass(order=True)     # enables <, <=, >, >=
class Version:
    major: int; minor: int; patch: int = 0
```

---

## Error handling

```python
try:
    risky()
except SpecificError as e:
    handle(e)
except (TypeA, TypeB) as e:
    handle(e)
else:
    success()              # no exception was raised
finally:
    cleanup()              # always runs

raise ValueError("msg")
raise RuntimeError("X") from original_error
```

---

## Generators

```python
# Generator function
def gen():
    yield 1
    yield 2

# Generator expression
(x**2 for x in range(10))

# yield from
def chain(a, b):
    yield from a
    yield from b
```

---

## Comprehensions

```python
[x for x in items if condition]              # list
{k: v for k, v in pairs}                    # dict
{x for x in items}                          # set
(x for x in items)                          # generator
```

---

## Type hints quick reference

```python
from typing import Optional, Union, Any
from collections.abc import Callable, Iterable, Sequence

def f(x: int) -> str: ...
def f(x: int | None) -> str | None: ...       # Python 3.10+
def f(items: list[int]) -> dict[str, int]: ...
def f(cb: Callable[[int, str], bool]) -> None: ...

# Useful aliases
type UserId = int                              # Python 3.12+
UserId = int                                   # before 3.12
```
