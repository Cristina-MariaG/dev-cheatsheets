# Python — Advanced Classes

> Dunder methods, __slots__, dataclasses, descriptors, metaclasses.

---

## Comparison and hashing

```python
from functools import total_ordering

@total_ordering   # auto-generates <=, >, >= from __eq__ + __lt__
class Version:
    def __init__(self, major: int, minor: int, patch: int = 0) -> None:
        self.major = major
        self.minor = minor
        self.patch = patch

    def __eq__(self, other: object) -> bool:
        if not isinstance(other, Version):
            return NotImplemented
        return (self.major, self.minor, self.patch) == (other.major, other.minor, other.patch)

    def __lt__(self, other: "Version") -> bool:
        if not isinstance(other, Version):
            return NotImplemented
        return (self.major, self.minor, self.patch) < (other.major, other.minor, other.patch)

    def __hash__(self) -> int:
        return hash((self.major, self.minor, self.patch))

v1 = Version(1, 2, 0)
v2 = Version(1, 3, 0)
v1 < v2    # True
v1 <= v2   # True  — from @total_ordering
v1 > v2    # False — from @total_ordering
{v1, v2}   # set works because __hash__ is defined
```

> **Rule**: if you define `__eq__`, Python sets `__hash__ = None` (object becomes unhashable). Always define `__hash__` alongside `__eq__` if you need the object in sets or as dict keys.

---

## Container protocol

Implement these dunders to make your class behave like a built-in container.

```python
class Stack:
    def __init__(self) -> None:
        self._items: list = []

    def push(self, item) -> None:
        self._items.append(item)

    def pop(self):
        return self._items.pop()

    def __len__(self) -> int:
        return len(self._items)

    def __getitem__(self, index: int):
        return self._items[index]

    def __setitem__(self, index: int, value) -> None:
        self._items[index] = value

    def __contains__(self, item) -> bool:
        return item in self._items

    def __iter__(self):
        return iter(self._items)     # delegate to list's iterator

    def __repr__(self) -> str:
        return f"Stack({self._items!r})"

s = Stack()
s.push(1); s.push(2); s.push(3)
len(s)      # 3
s[0]        # 1
2 in s      # True
for x in s: print(x)
```

| Dunder | Triggered by |
|---|---|
| `__len__` | `len(obj)`, truthiness if no `__bool__` |
| `__getitem__` | `obj[key]` |
| `__setitem__` | `obj[key] = value` |
| `__delitem__` | `del obj[key]` |
| `__contains__` | `item in obj` |
| `__iter__` | `for x in obj`, `list(obj)`, `*obj` |

---

## Context manager protocol

Implement `__enter__` and `__exit__` to use with `with`.

```python
class Timer:
    def __enter__(self):
        import time
        self.start = time.perf_counter()
        return self                      # value bound to `as` variable

    def __exit__(self, exc_type, exc_val, exc_tb) -> bool:
        import time
        self.elapsed = time.perf_counter() - self.start
        print(f"Elapsed: {self.elapsed:.3f}s")
        return False   # False = don't suppress exceptions (True would swallow them)

with Timer() as t:
    result = expensive_operation()
# Elapsed: 0.042s
```

### @contextmanager — simpler alternative

```python
from contextlib import contextmanager

@contextmanager
def managed_connection(url: str):
    conn = connect(url)
    try:
        yield conn          # everything before yield = __enter__
    finally:
        conn.close()        # everything in finally = __exit__

with managed_connection("postgres://...") as conn:
    conn.execute("SELECT 1")
```

---

## __call__

Make an instance callable like a function.

```python
class RateLimiter:
    def __init__(self, max_calls: int, period: float) -> None:
        self.max_calls = max_calls
        self.period = period
        self._calls: list = []

    def __call__(self, func):
        import functools, time

        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            now = time.time()
            self._calls = [t for t in self._calls if now - t < self.period]
            if len(self._calls) >= self.max_calls:
                raise RuntimeError("Rate limit exceeded")
            self._calls.append(now)
            return func(*args, **kwargs)
        return wrapper

@RateLimiter(max_calls=5, period=1.0)
def fetch(url: str): ...
```

---

## __slots__

Declare allowed instance attributes explicitly. Prevents accidental attributes, saves memory (no per-instance `__dict__`).

```python
class Point:
    __slots__ = ("x", "y")

    def __init__(self, x: float, y: float) -> None:
        self.x = x
        self.y = y

p = Point(1, 2)
p.z = 3     # AttributeError — z not in __slots__
p.__dict__  # AttributeError — no __dict__ on slotted classes
```

> Use `__slots__` on classes you'll create millions of instances of (e.g., data records, nodes). It reduces memory by ~40-50% per instance.

### __slots__ with inheritance

```python
class Base:
    __slots__ = ("x",)

class Child(Base):
    __slots__ = ("y",)    # only declares NEW slots — x is inherited

c = Child()
c.x = 1   # OK — inherited slot
c.y = 2   # OK — child slot
```

If any class in the hierarchy omits `__slots__`, those instances get a `__dict__` anyway.

---

## dataclasses

Auto-generate `__init__`, `__repr__`, `__eq__` from annotated fields.

```python
from dataclasses import dataclass, field

@dataclass
class Product:
    name: str
    price: float
    tags: list[str] = field(default_factory=list)
    _internal: str = field(default="", repr=False, compare=False)

    def discounted(self, pct: float) -> "Product":
        return Product(self.name, self.price * (1 - pct), self.tags)

p = Product("Widget", 9.99)
repr(p)    # "Product(name='Widget', price=9.99, tags=[])"
p2 = Product("Widget", 9.99)
p == p2    # True — __eq__ compares all fields
```

### Variants

```python
# frozen=True — immutable and hashable
@dataclass(frozen=True)
class Point:
    x: float
    y: float

pt = Point(1, 2)
pt.x = 5    # FrozenInstanceError

# order=True — enables <, <=, >, >=
@dataclass(order=True)
class Version:
    major: int
    minor: int
    patch: int = 0

# Post-init processing
@dataclass
class Circle:
    radius: float
    area: float = field(init=False)

    def __post_init__(self) -> None:
        import math
        self.area = math.pi * self.radius ** 2
```

---

## Descriptors

A descriptor is an object that customizes attribute access when stored as a class attribute.

```python
class Validated:
    """Descriptor that enforces type and value constraints."""

    def __set_name__(self, owner, name: str) -> None:
        self.name = name                    # called when class is created

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self                     # accessed from the class itself
        return obj.__dict__.get(self.name)

    def __set__(self, obj, value) -> None:
        if not isinstance(value, (int, float)):
            raise TypeError(f"{self.name} must be numeric")
        if value < 0:
            raise ValueError(f"{self.name} must be >= 0")
        obj.__dict__[self.name] = value

class Product:
    price    = Validated()
    quantity = Validated()

    def __init__(self, price: float, quantity: int) -> None:
        self.price    = price       # triggers Validated.__set__
        self.quantity = quantity

p = Product(9.99, 10)
p.price = -1    # ValueError: price must be >= 0
p.price = "x"   # TypeError: price must be numeric
```

> `@property` is syntactic sugar for a descriptor. Descriptors let you reuse the same logic across multiple attributes and multiple classes.

---

## Metaclasses

A metaclass is the class of a class. `type` is the default metaclass. By creating a custom metaclass, you control how classes themselves are created.

```python
class SingletonMeta(type):
    _instances: dict = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Database(metaclass=SingletonMeta):
    def __init__(self, url: str) -> None:
        self.url = url

db1 = Database("postgres://...")
db2 = Database("another-url")
db1 is db2   # True — always the same instance
```

### __init_subclass__ — simpler alternative to metaclasses

Called automatically on the parent when a subclass is created. Covers most real-world metaclass use cases.

```python
class PluginBase:
    _registry: dict[str, type] = {}

    def __init_subclass__(cls, plugin_name: str = "", **kwargs) -> None:
        super().__init_subclass__(**kwargs)
        if plugin_name:
            PluginBase._registry[plugin_name] = cls

class CSVPlugin(PluginBase, plugin_name="csv"):
    def parse(self, data): ...

class JSONPlugin(PluginBase, plugin_name="json"):
    def parse(self, data): ...

PluginBase._registry   # {"csv": CSVPlugin, "json": JSONPlugin}

# Auto-register pattern — no manual registration needed
def get_plugin(name: str) -> PluginBase:
    return PluginBase._registry[name]()
```

---

## Common pitfalls

- **`__eq__` without `__hash__`** — Python silently sets `__hash__ = None`. The object can no longer be used in sets or as dict keys
- **`__slots__` + `__dict__`** — if any parent doesn't define `__slots__`, the subclass still gets `__dict__` and the memory savings are lost
- **Mutable default in dataclass** — `tags: list = []` raises `ValueError`. Always use `field(default_factory=list)`
- **`__del__`** — the destructor is called by the GC at an unpredictable time, sometimes never. Don't use it for resource cleanup — use context managers instead
- **Returning `True` from `__exit__`** — it suppresses exceptions silently. Return `False` unless you intentionally want to swallow them
