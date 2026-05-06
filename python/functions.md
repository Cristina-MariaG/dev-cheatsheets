# Python — Functions

> Arguments, closures, decorators, and functional tools.

---

## Arguments

```python
def create_user(name: str, age: int, role: str = "user") -> dict:
    return {"name": name, "age": age, "role": role}

create_user("Alice", 30)
create_user("Bob", 25, role="admin")
create_user(age=25, name="Carol")    # keyword args — any order
```

### *args and **kwargs

```python
def add(*numbers: int) -> int:
    return sum(numbers)            # numbers is a tuple

add(1, 2, 3, 4)   # 10

def log(message: str, **context) -> None:
    print(message, context)        # context is a dict

log("error", user_id=42, action="delete")
# "error" {'user_id': 42, 'action': 'delete'}
```

### Keyword-only and positional-only

```python
# Keyword-only — args after * must be passed as keywords
def connect(host: str, *, port: int = 5432, timeout: int = 30):
    ...

connect("localhost", port=5433)      # OK
connect("localhost", 5433)           # TypeError

# Positional-only — args before / cannot be passed by name (Python 3.8+)
def normalize(x: float, y: float, /, scale: float = 1.0):
    ...

normalize(1.0, 2.0)            # OK
normalize(x=1.0, y=2.0)        # TypeError
```

---

## Closures

A function that captures variables from its enclosing scope.

```python
def make_multiplier(factor: int):
    def multiply(x: int) -> int:
        return x * factor        # factor is captured from outer scope
    return multiply

double = make_multiplier(2)
triple = make_multiplier(3)
double(5)   # 10
triple(5)   # 15
```

### Late binding — common pitfall

```python
# BUG — all lambdas capture the same i reference
funcs = [lambda x: x * i for i in range(3)]
funcs[0](1)   # 2 — NOT 0, because i=2 at call time

# FIX — capture value at definition time with a default argument
funcs = [lambda x, i=i: x * i for i in range(3)]
funcs[0](1)   # 0
```

---

## Decorators

A function that wraps another to add behavior — without modifying the original.

```python
import functools

def log_call(func):
    @functools.wraps(func)    # preserves __name__, __doc__ of the original
    def wrapper(*args, **kwargs):
        print(f"→ {func.__name__}({args}, {kwargs})")
        result = func(*args, **kwargs)
        print(f"← {result}")
        return result
    return wrapper

@log_call
def add(a, b):
    return a + b

add(1, 2)
# → add((1, 2), {})
# ← 3
```

### Decorator with arguments

```python
def retry(max_attempts: int = 3, delay: float = 0.5):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            import time
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception:
                    if attempt == max_attempts - 1:
                        raise
                    time.sleep(delay)
        return wrapper
    return decorator

@retry(max_attempts=5, delay=1.0)
def fetch(url: str):
    ...
```

### Class as decorator

```python
class Cache:
    def __init__(self, func):
        functools.update_wrapper(self, func)
        self.func = func
        self._cache = {}

    def __call__(self, *args):
        if args not in self._cache:
            self._cache[args] = self.func(*args)
        return self._cache[args]

@Cache
def expensive(n: int) -> int:
    return n ** 2
```

---

## Lambda

Anonymous single-expression function.

```python
square = lambda x: x ** 2

# Useful as a sort key
users.sort(key=lambda u: u["score"], reverse=True)

# With map/filter — comprehensions are usually cleaner
doubled = list(map(lambda x: x * 2, numbers))
odds    = list(filter(lambda x: x % 2, numbers))
```

---

## functools

```python
from functools import lru_cache, partial, reduce

# lru_cache — memoize repeated calls
@lru_cache(maxsize=128)
def fibonacci(n: int) -> int:
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

fibonacci.cache_info()   # hits, misses, maxsize, currsize

# partial — fix some arguments ahead of time
power_of_2 = partial(pow, 2)
power_of_2(10)   # 1024

# reduce — fold a sequence into a single value
product = reduce(lambda acc, x: acc * x, [1, 2, 3, 4])  # 24
```

---

## Common pitfalls

- **Mutable default argument** — `def f(items=[])`: the list is created once at function definition and shared across calls. Use `def f(items=None): items = items if items is not None else []`
- **Forgetting `@functools.wraps`** — without it, the wrapper function replaces `func.__name__` and `func.__doc__`
- **Returning `None` implicitly** — a function without `return` returns `None`. Don't assign the result of an in-place method like `list.sort()`
