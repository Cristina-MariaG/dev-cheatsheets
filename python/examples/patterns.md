# Example — Python Patterns

> Common patterns that are idiomatic in Python.

---

## Singleton via metaclass

```python
class SingletonMeta(type):
    _instances: dict = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Config(metaclass=SingletonMeta):
    def __init__(self) -> None:
        self.debug = False
        self.db_url = "postgres://localhost/app"

c1 = Config()
c2 = Config()
c1 is c2   # True
```

---

## Factory via __init_subclass__

Auto-register subclasses without a separate registry call.

```python
class Serializer:
    _registry: dict[str, type] = {}

    def __init_subclass__(cls, format: str = "", **kwargs) -> None:
        super().__init_subclass__(**kwargs)
        if format:
            Serializer._registry[format] = cls

    @classmethod
    def for_format(cls, format: str) -> "Serializer":
        if format not in cls._registry:
            raise ValueError(f"Unknown format: {format}")
        return cls._registry[format]()

    def serialize(self, data) -> str:
        raise NotImplementedError

class JSONSerializer(Serializer, format="json"):
    def serialize(self, data) -> str:
        import json
        return json.dumps(data)

class CSVSerializer(Serializer, format="csv"):
    def serialize(self, data) -> str:
        import csv, io
        out = io.StringIO()
        writer = csv.DictWriter(out, fieldnames=data[0].keys())
        writer.writeheader()
        writer.writerows(data)
        return out.getvalue()

s = Serializer.for_format("json")
s.serialize({"name": "Alice"})   # '{"name": "Alice"}'
```

---

## Observer / event system

```python
from collections import defaultdict
from typing import Callable

class EventBus:
    def __init__(self) -> None:
        self._listeners: dict[str, list[Callable]] = defaultdict(list)

    def on(self, event: str, callback: Callable) -> None:
        self._listeners[event].append(callback)

    def off(self, event: str, callback: Callable) -> None:
        self._listeners[event].remove(callback)

    def emit(self, event: str, **payload) -> None:
        for callback in self._listeners[event]:
            callback(**payload)

bus = EventBus()

def on_user_created(user_id: int, email: str) -> None:
    print(f"Sending welcome email to {email}")

bus.on("user.created", on_user_created)
bus.emit("user.created", user_id=42, email="alice@x.com")
# Sending welcome email to alice@x.com
```

---

## Descriptor for validated attributes

Reuse validation logic across multiple classes and attributes.

```python
class TypedField:
    def __init__(self, expected_type: type) -> None:
        self.expected_type = expected_type

    def __set_name__(self, owner, name: str) -> None:
        self.name = name

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return obj.__dict__.get(self.name)

    def __set__(self, obj, value) -> None:
        if not isinstance(value, self.expected_type):
            raise TypeError(
                f"{self.name} must be {self.expected_type.__name__}, "
                f"got {type(value).__name__}"
            )
        obj.__dict__[self.name] = value

class User:
    name  = TypedField(str)
    age   = TypedField(int)
    score = TypedField(float)

    def __init__(self, name: str, age: int, score: float) -> None:
        self.name = name
        self.age = age
        self.score = score

User("Alice", "30", 9.5)   # TypeError: age must be int, got str
```

---

## Context manager for timing / profiling

```python
from contextlib import contextmanager
import time

@contextmanager
def timer(label: str = ""):
    start = time.perf_counter()
    try:
        yield
    finally:
        elapsed = time.perf_counter() - start
        tag = f"[{label}] " if label else ""
        print(f"{tag}{elapsed:.4f}s")

with timer("database query"):
    results = db.execute("SELECT ...")
# [database query] 0.0023s
```

---

## Lazy property with __set_name__

Compute an attribute only once, on first access, then cache it.

```python
class lazy_property:
    def __init__(self, func) -> None:
        self.func = func
        self.name = None

    def __set_name__(self, owner, name: str) -> None:
        self.name = name

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        value = self.func(obj)
        obj.__dict__[self.name] = value    # replace descriptor with value
        return value

class Document:
    def __init__(self, text: str) -> None:
        self.text = text

    @lazy_property
    def word_count(self) -> int:
        print("computing...")              # only runs once
        return len(self.text.split())

doc = Document("hello world foo bar")
doc.word_count   # "computing..." → 4
doc.word_count   # 4  — cached in __dict__, descriptor not called again
```

---

## Chained builder pattern

```python
class QueryBuilder:
    def __init__(self, table: str) -> None:
        self._table = table
        self._conditions: list[str] = []
        self._order: str | None = None
        self._limit: int | None = None

    def where(self, condition: str) -> "QueryBuilder":
        self._conditions.append(condition)
        return self   # return self to allow chaining

    def order_by(self, column: str) -> "QueryBuilder":
        self._order = column
        return self

    def limit(self, n: int) -> "QueryBuilder":
        self._limit = n
        return self

    def build(self) -> str:
        sql = f"SELECT * FROM {self._table}"
        if self._conditions:
            sql += " WHERE " + " AND ".join(self._conditions)
        if self._order:
            sql += f" ORDER BY {self._order}"
        if self._limit:
            sql += f" LIMIT {self._limit}"
        return sql

query = (
    QueryBuilder("users")
    .where("active = true")
    .where("age > 18")
    .order_by("created_at DESC")
    .limit(10)
    .build()
)
# "SELECT * FROM users WHERE active = true AND age > 18 ORDER BY created_at DESC LIMIT 10"
```
