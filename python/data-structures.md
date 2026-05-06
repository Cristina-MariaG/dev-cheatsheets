# Python — Data Structures

> Lists, dicts, sets, tuples, and comprehensions.

---

## Lists

Ordered, mutable sequence.

```python
items = [1, 2, 3]

# Add
items.append(4)           # [1, 2, 3, 4]
items.insert(0, 0)        # insert at index
items.extend([5, 6])      # add multiple

# Remove
items.remove(3)           # remove first occurrence by value
items.pop()               # remove and return last
items.pop(0)              # remove and return by index
del items[1]              # delete by index

# Query
items.index(2)            # first index of value — ValueError if missing
items.count(1)            # count occurrences
2 in items                # True

# Sort
items.sort()              # in-place — returns None
items.sort(key=lambda x: -x, reverse=False)
sorted(items)             # returns new sorted list — original unchanged
items.reverse()           # in-place reverse
items[::-1]               # reversed copy
```

### Slicing

```python
items = [0, 1, 2, 3, 4]
items[1:3]    # [1, 2]     — index 1 up to (not including) 3
items[:2]     # [0, 1]     — from start
items[2:]     # [2, 3, 4]  — to end
items[-2:]    # [3, 4]     — last 2
items[::2]    # [0, 2, 4]  — every other element
items[::-1]   # [4, 3, 2, 1, 0]
```

---

## Dictionaries

Key-value store. Keys must be hashable. Preserves insertion order (Python 3.7+).

```python
user = {"name": "Alice", "age": 30}

# Access
user["name"]                       # "Alice" — KeyError if missing
user.get("email")                  # None if missing
user.get("email", "n/a")           # default value

# Add / update
user["email"] = "a@b.com"
user.setdefault("role", "user")    # set only if key absent

# Remove
del user["age"]
user.pop("age", None)              # returns value, no error if missing

# Iterate
for key in user:                   # iterate keys
    pass
for key, value in user.items():
    pass
user.keys()    # dict_keys view
user.values()  # dict_values view

# Merge (Python 3.9+)
merged = dict1 | dict2             # dict2 overrides dict1
merged = {**dict1, **dict2}        # same, works in older versions

# Build from pairs
dict(zip(keys, values))
{k: v for k, v in pairs}
```

---

## Sets

Unordered, unique values. O(1) membership check.

```python
a = {1, 2, 3}
b = {2, 3, 4}

a | b     # union           {1, 2, 3, 4}
a & b     # intersection    {2, 3}
a - b     # difference      {1}        (in a, not in b)
a ^ b     # symmetric diff  {1, 4}     (in one but not both)

a.add(5)
a.discard(99)   # no error if missing
a.remove(99)    # KeyError if missing
1 in a          # True — O(1)

# Deduplicate a list
unique = list(set([1, 2, 2, 3, 3]))
```

---

## Tuples

Immutable, ordered sequence. Faster than lists for fixed data.

```python
point = (10, 20)
x, y = point       # unpack
x, *rest = (1, 2, 3, 4)   # x=1, rest=[2,3,4]

# Single-element tuple needs a trailing comma
singleton = (42,)   # NOT (42) — that's just parentheses

# Named tuple — readable, still immutable
from collections import namedtuple
Point = namedtuple("Point", ["x", "y"])
p = Point(10, 20)
p.x    # 10 — clearer than p[0]
p._asdict()  # OrderedDict
```

---

## Comprehensions

Concise, readable way to build collections.

```python
# List
squares = [x**2 for x in range(10)]
evens   = [x for x in range(20) if x % 2 == 0]

# Nested
matrix = [[i * j for j in range(3)] for i in range(3)]

# Dict
lengths = {word: len(word) for word in ["apple", "banana"]}

# Set
unique_lengths = {len(word) for word in words}

# Generator expression — lazy, no memory allocation
total = sum(x**2 for x in range(1_000_000))   # doesn't build a list
```

---

## Common pitfalls

- **`list.sort()` returns `None`** — it mutates in-place. `sorted()` returns a new list
- **Shallow copy** — `b = a` is a reference, not a copy. Use `b = a.copy()` or `b = a[:]`
- **Dict `.get()` vs `[]`** — always use `.get()` when the key might be missing
- **Empty set** — `{}` creates a dict, not a set. Use `set()` for an empty set
- **Set from string** — `set("hello")` gives `{'h', 'e', 'l', 'o'}`, not a set of one string
