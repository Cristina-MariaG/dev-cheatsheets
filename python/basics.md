# Python — Basics

> Syntax, types, control flow, and functions.

---

## Variables and types

```python
name: str = "Alice"
age: int = 30
score: float = 9.5
active: bool = True
nothing = None

# Type is dynamic — annotation is optional but recommended
x = 42        # int
x = "hello"   # now a str — valid but avoid in practice
```

### Built-in types

| Type | Example | Notes |
|---|---|---|
| `int` | `42`, `-3` | Arbitrary precision |
| `float` | `3.14`, `-0.5` | 64-bit IEEE 754 |
| `bool` | `True`, `False` | Subclass of `int` |
| `str` | `"hello"` | Immutable, Unicode |
| `bytes` | `b"data"` | Raw bytes |
| `NoneType` | `None` | Singleton |

```python
int("42")      # 42      — convert str to int
float("3.14")  # 3.14
str(42)        # "42"
bool(0)        # False   — 0, "", [], {}, None are falsy
bool("hi")     # True    — everything else is truthy
```

---

## String formatting

```python
name = "Alice"
score = 9.5

# f-string — preferred
f"Hello, {name}!"
f"Score: {score:.2f}"      # 2 decimal places → "9.50"
f"{1_000_000:,}"           # thousands separator → "1,000,000"
f"{0.753:.1%}"             # percentage → "75.3%"
f"{'right':>10}"           # right-align in 10 chars
f"{name!r}"                # repr → "'Alice'"

# Multi-line f-string
message = (
    f"Name:  {name}\n"
    f"Score: {score}"
)
```

---

## Control flow

```python
# if / elif / else
if score >= 90:
    grade = "A"
elif score >= 70:
    grade = "B"
else:
    grade = "C"

# Ternary
label = "pass" if score >= 50 else "fail"

# match (Python 3.10+)
match status:
    case "active":
        process()
    case "pending" | "waiting":
        queue()
    case _:
        reject()
```

### Loops

```python
# for — iterate any iterable
for item in ["a", "b", "c"]:
    print(item)

# range
for i in range(5):          # 0, 1, 2, 3, 4
    pass
for i in range(2, 10, 2):   # 2, 4, 6, 8
    pass

# enumerate — index + value
for i, item in enumerate(["a", "b", "c"]):
    print(i, item)   # 0 a / 1 b / 2 c

# zip — iterate multiple in parallel
for name, score in zip(names, scores):
    print(f"{name}: {score}")

# while
while condition:
    if done:
        break
    if skip:
        continue
```

---

## Functions

```python
def greet(name: str, greeting: str = "Hello") -> str:
    return f"{greeting}, {name}!"

greet("Alice")                       # "Hello, Alice!"
greet("Bob", "Hi")                   # "Hi, Bob!"
greet(greeting="Hey", name="Carol")  # keyword args — any order
```

### Multiple return values

Python returns a tuple — unpack on the receiving end.

```python
def min_max(numbers: list[int]) -> tuple[int, int]:
    return min(numbers), max(numbers)

low, high = min_max([3, 1, 4, 1, 5])
```

---

## Common pitfalls

- **`==` vs `is`** — `==` compares value, `is` compares identity. Never `x is "hello"`, always `x == "hello"`
- **Integer division** — `/` always returns `float` (5/2 = 2.5), `//` does floor division (5//2 = 2)
- **Mutable default argument** — `def f(items=[])` shares the list across calls. Use `None` and initialize inside the body
- **`not in` for membership** — prefer `x not in collection` over `not x in collection`
- **Indentation** — Python uses indentation to define blocks. Mixing tabs and spaces causes `TabError`
