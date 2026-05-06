# Python — Error Handling

> Exceptions, custom errors, context managers.

---

## Built-in exceptions

```python
ValueError("Invalid input")          # bad value, right type
TypeError("Expected int, got str")   # wrong type
KeyError("user_id")                  # missing dict key
IndexError("list index out of range")
AttributeError("object has no attribute 'x'")
FileNotFoundError("No such file: data.csv")
PermissionError("Access denied")
RuntimeError("Something unexpected")
NotImplementedError("Subclass must implement this")
StopIteration                        # iterator is exhausted
```

---

## try / except / else / finally

```python
try:
    data = json.loads(raw)      # might raise json.JSONDecodeError
    result = process(data)
except json.JSONDecodeError as e:
    print(f"Bad JSON: {e}")     # handle specific error
except (TypeError, ValueError) as e:
    print(f"Bad value: {e}")    # handle multiple types
except Exception as e:
    print(f"Unexpected: {e}")   # catch-all — use sparingly
    raise                       # re-raise so it's not silently swallowed
else:
    save(result)                # runs only if NO exception was raised in try
finally:
    cleanup()                   # always runs — with or without exception
```

> `else` is useful to keep the "success" code separate from error handling. `finally` is for cleanup (closing files, releasing locks) regardless of what happened.

---

## Raising exceptions

```python
def get_user(user_id: int):
    if user_id <= 0:
        raise ValueError(f"user_id must be positive, got {user_id}")
    ...

# Re-raise the current exception (inside except block)
try:
    risky()
except Exception:
    log_error()
    raise            # preserves original traceback

# Chain exceptions — "X happened while handling Y"
try:
    connect()
except ConnectionError as e:
    raise RuntimeError("Service unavailable") from e
```

---

## Custom exceptions

Create a hierarchy to make `isinstance` checks easy and add structured context.

```python
class AppError(Exception):
    """Base for all app-level errors."""
    pass

class NotFoundError(AppError):
    def __init__(self, resource: str, resource_id) -> None:
        super().__init__(f"{resource} {resource_id!r} not found")
        self.resource = resource
        self.resource_id = resource_id

class ValidationError(AppError):
    def __init__(self, message: str, fields: dict | None = None) -> None:
        super().__init__(message)
        self.fields = fields or {}

class PermissionError(AppError):
    def __init__(self, action: str, user_id: int) -> None:
        super().__init__(f"User {user_id} cannot perform '{action}'")
        self.action = action
        self.user_id = user_id
```

```python
# Catching the hierarchy
try:
    update_user(user_id, payload)
except NotFoundError as e:
    return 404, str(e)
except ValidationError as e:
    return 400, {"message": str(e), "fields": e.fields}
except AppError as e:
    return 500, str(e)      # catch any remaining app error
```

---

## Context managers — with statement

```python
# File — auto-closes on exit, even if exception is raised
with open("data.csv") as f:
    content = f.read()

# Multiple resources
with open("input.txt") as src, open("output.txt", "w") as dst:
    dst.write(src.read())
```

### Writing your own context manager

```python
# Option 1 — class with __enter__ / __exit__
class Transaction:
    def __init__(self, db) -> None:
        self.db = db

    def __enter__(self):
        self.db.begin()
        return self

    def __exit__(self, exc_type, exc_val, exc_tb) -> bool:
        if exc_type is None:
            self.db.commit()
        else:
            self.db.rollback()
        return False   # don't suppress the exception

with Transaction(db) as tx:
    tx.db.execute("INSERT ...")

# Option 2 — @contextmanager (simpler for one-off use)
from contextlib import contextmanager

@contextmanager
def transaction(db):
    db.begin()
    try:
        yield db         # control passes to the `with` block here
        db.commit()
    except Exception:
        db.rollback()
        raise            # re-raise so the caller still sees the error
```

---

## Handling errors in async code

```python
import asyncio

async def fetch(url: str) -> str:
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            if response.status != 200:
                raise ValueError(f"HTTP {response.status}")
            return await response.text()

async def load_all(urls: list[str]) -> list:
    tasks = [fetch(url) for url in urls]
    results = await asyncio.gather(*tasks, return_exceptions=True)

    for url, result in zip(urls, results):
        if isinstance(result, Exception):
            print(f"Failed {url}: {result}")   # don't crash — handle per-task
        else:
            process(result)
```

---

## Common pitfalls

- **Empty except block** — silently swallows errors. At minimum, log them
- **Catching `Exception` and not re-raising** — you hide bugs. Always re-raise if you don't know how to handle it
- **`except Exception` too broad** — it also catches `KeyboardInterrupt` and `SystemExit` subclasses... use `BaseException` for those
- **`throw 'string'`** — Python lets you raise any object, but only `Exception` subclasses have a stack trace. Always `raise SomeError("message")`
- **`from e` vs `from None`** — use `raise NewError() from e` to chain, `raise NewError() from None` to hide the original cause
