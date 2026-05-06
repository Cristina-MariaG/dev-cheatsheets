# Python — Iterators and Generators

> Iterator protocol, generator functions, yield, lazy evaluation.

---

## Iterator protocol

Any object with `__iter__` and `__next__` is an iterator.

```python
class Countdown:
    def __init__(self, start: int) -> None:
        self.current = start

    def __iter__(self):
        return self      # the object is its own iterator

    def __next__(self) -> int:
        if self.current <= 0:
            raise StopIteration
        value = self.current
        self.current -= 1
        return value

for n in Countdown(3):
    print(n)    # 3, 2, 1

list(Countdown(5))   # [5, 4, 3, 2, 1]
```

The `for` loop calls `iter()` on the object, then repeatedly calls `next()` until `StopIteration`.

---

## Generator functions

A function with `yield` is a generator. It returns a generator object that produces values lazily — one at a time, on demand.

```python
def countdown(start: int):
    while start > 0:
        yield start          # pause here, send value to caller
        start -= 1           # resume here on next next() call

for n in countdown(3):
    print(n)    # 3, 2, 1

gen = countdown(3)
next(gen)    # 3
next(gen)    # 2
next(gen)    # 1
next(gen)    # StopIteration
```

> A generator function returns a generator object when called. The function body doesn't run until you call `next()`.

### Reading a large file line by line

```python
def read_lines(path: str):
    with open(path) as f:
        for line in f:
            yield line.strip()    # only one line in memory at a time

for line in read_lines("huge_file.txt"):
    process(line)
```

---

## Generator expressions

Like list comprehensions, but lazy — no list is built in memory.

```python
# List comprehension — builds full list immediately
squares_list = [x**2 for x in range(1_000_000)]    # 8 MB in memory

# Generator expression — computes on demand
squares_gen  = (x**2 for x in range(1_000_000))    # almost no memory

# Useful with aggregation functions
total   = sum(x**2 for x in range(1_000_000))
maximum = max(len(line) for line in open("file.txt"))
any_odd = any(x % 2 for x in numbers)
```

---

## yield from

Delegate to another iterable — cleaner than looping and re-yielding.

```python
def chain(*iterables):
    for it in iterables:
        yield from it           # equivalent to: for x in it: yield x

list(chain([1, 2], [3, 4], [5]))   # [1, 2, 3, 4, 5]

# Flatten nested lists
def flatten(items):
    for item in items:
        if isinstance(item, list):
            yield from flatten(item)   # recurse
        else:
            yield item

list(flatten([1, [2, [3, 4]], 5]))   # [1, 2, 3, 4, 5]
```

---

## Sending values into a generator

Generators can receive values via `.send()`.

```python
def accumulator():
    total = 0
    while True:
        value = yield total    # yield sends total out, receives value in
        if value is None:
            break
        total += value

gen = accumulator()
next(gen)        # 0  — prime the generator (advance to first yield)
gen.send(10)     # 10
gen.send(20)     # 30
gen.send(5)      # 35
```

---

## itertools — standard library essentials

```python
from itertools import (
    chain, islice, takewhile, dropwhile,
    groupby, product, combinations, permutations, count, cycle
)

# chain — join iterables
list(chain([1, 2], [3, 4]))        # [1, 2, 3, 4]

# islice — slice any iterable (including infinite ones)
list(islice(count(0), 5))          # [0, 1, 2, 3, 4]

# takewhile / dropwhile
list(takewhile(lambda x: x < 5, [1, 3, 5, 7]))   # [1, 3]
list(dropwhile(lambda x: x < 5, [1, 3, 5, 7]))   # [5, 7]

# groupby — group consecutive elements (sort first)
data = [{"dept": "A"}, {"dept": "A"}, {"dept": "B"}]
data.sort(key=lambda x: x["dept"])
for dept, group in groupby(data, key=lambda x: x["dept"]):
    print(dept, list(group))

# combinations / permutations
list(combinations([1, 2, 3], 2))    # [(1,2), (1,3), (2,3)]
list(permutations([1, 2, 3], 2))    # [(1,2), (1,3), (2,1), ...]

# product — cartesian product
list(product([0, 1], repeat=3))     # all 3-bit binary numbers
```

---

## Common pitfalls

- **Generators are single-use** — once exhausted, they don't reset. Create a new one or use a list
- **Generator is lazy** — the function body doesn't run at all until you iterate it. Side effects are deferred
- **`yield from` vs `return`** — you can combine them: `return` inside a generator raises `StopIteration` with the return value
- **Priming with `send()`** — you must call `next()` (or `send(None)`) once before sending real values, to advance to the first `yield`
