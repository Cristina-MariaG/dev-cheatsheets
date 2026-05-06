# Python — Classes

> Class definition, instance and class variables, methods, and properties.

---

## Defining a class

```python
class Dog:
    species = "Canis familiaris"   # class variable — shared by ALL instances

    def __init__(self, name: str, age: int) -> None:
        self.name = name           # instance variable — unique to each instance
        self.age = age

    def bark(self) -> str:
        return f"{self.name} says woof!"

    def __repr__(self) -> str:
        return f"Dog(name={self.name!r}, age={self.age!r})"

    def __str__(self) -> str:
        return f"{self.name} ({self.age} yrs)"
```

`__init__` runs when an instance is created. `self` is always the first parameter — it refers to the instance being constructed.

```python
dog = Dog("Rex", 3)
dog.bark()          # "Rex says woof!"
repr(dog)           # "Dog(name='Rex', age=3)"   — used in REPL, logs
str(dog)            # "Rex (3 yrs)"              — used by print()
dog.species         # "Canis familiaris"
Dog.species         # same — accessible from class or instance
```

---

## Instance vs class variables

```python
class Counter:
    count = 0           # class variable — shared

    def __init__(self) -> None:
        Counter.count += 1
        self.id = Counter.count    # instance variable — unique per object

a = Counter()   # Counter.count = 1, a.id = 1
b = Counter()   # Counter.count = 2, b.id = 2
```

> If you do `self.count = ...`, Python creates a new **instance** variable that shadows the class variable on that object. The class variable stays unchanged.

---

## Instance, class, and static methods

```python
class User:
    _all: list["User"] = []

    def __init__(self, name: str, email: str) -> None:
        self.name = name
        self.email = email
        User._all.append(self)

    # Instance method — receives the instance as first arg (self)
    def display(self) -> str:
        return f"{self.name} <{self.email}>"

    # Class method — receives the CLASS as first arg (cls), not the instance
    # Main use: alternative constructors
    @classmethod
    def from_dict(cls, data: dict) -> "User":
        return cls(data["name"], data["email"])

    @classmethod
    def count(cls) -> int:
        return len(cls._all)

    # Static method — no access to instance or class
    # Use when the logic belongs conceptually to the class but needs no state
    @staticmethod
    def is_valid_email(email: str) -> bool:
        return "@" in email and "." in email
```

```python
u = User.from_dict({"name": "Alice", "email": "alice@x.com"})
u.display()                      # "Alice <alice@x.com>"
User.count()                     # 1
User.is_valid_email("bad")       # False
```

| | `self` (instance method) | `cls` (class method) | static method |
|---|---|---|---|
| Access instance attributes | ✅ | ✗ | ✗ |
| Access class attributes | ✅ | ✅ | ✗ |
| Decorator | *(none)* | `@classmethod` | `@staticmethod` |
| Called on | instance or class | instance or class | instance or class |
| Typical use | Regular behavior | Alternative constructors | Utility, no state needed |

---

## Properties

`@property` lets you control attribute access without changing the public API. The caller uses `obj.attribute` — no parentheses — but you control the get/set behavior.

```python
class Temperature:
    def __init__(self, celsius: float) -> None:
        self._celsius = celsius      # _celsius is internal storage

    @property
    def celsius(self) -> float:
        """Getter — called when you read t.celsius"""
        return self._celsius

    @celsius.setter
    def celsius(self, value: float) -> None:
        """Setter — called when you write t.celsius = x"""
        if value < -273.15:
            raise ValueError("Temperature below absolute zero")
        self._celsius = value

    @celsius.deleter
    def celsius(self) -> None:
        """Deleter — called when you do del t.celsius"""
        del self._celsius

    @property
    def fahrenheit(self) -> float:
        """Read-only computed property — no setter defined"""
        return self._celsius * 9 / 5 + 32

t = Temperature(100)
t.celsius          # 100       — calls getter
t.celsius = 200    # calls setter, validates input
t.fahrenheit       # 392.0     — computed, read-only
t.fahrenheit = 0   # AttributeError — no setter
```

> Use `@property` to add validation or computation to an attribute you previously exposed directly — callers don't need to change their code.

---

## __repr__ and __str__

```python
class Point:
    def __init__(self, x: float, y: float) -> None:
        self.x = x
        self.y = y

    def __repr__(self) -> str:
        # For developers — unambiguous, should look like constructor call
        return f"Point({self.x!r}, {self.y!r})"

    def __str__(self) -> str:
        # For end users — readable
        return f"({self.x}, {self.y})"

p = Point(1, 2)
repr(p)     # "Point(1, 2)"   — used in REPL, debugger, inside lists/dicts
str(p)      # "(1, 2)"        — used by print()
f"{p}"      # "(1, 2)"        — f-string calls __str__
[p]         # [Point(1, 2)]   — repr used inside containers
```

> If you only define `__repr__`, Python uses it as fallback for `__str__` too. If you only define `__str__`, the default `__repr__` still shows `<Point object at 0x...>`.

---

## Name mangling

```python
class BankAccount:
    def __init__(self, balance: float) -> None:
        self.owner = "Alice"       # public
        self._balance = balance    # convention: internal use, but accessible
        self.__pin = 1234          # name-mangled: becomes _BankAccount__pin

account = BankAccount(100)
account.owner       # "Alice"
account._balance    # 100.0     — accessible but signals "don't touch"
account.__pin       # AttributeError
account._BankAccount__pin   # 1234 — mangled name, still reachable if needed
```

> `__double_underscore` is for avoiding accidental override in subclasses, not for security.

---

## Common pitfalls

- **Forgetting `self`** — all instance methods must have `self` as first parameter, or you get `TypeError: takes 0 positional arguments but 1 was given`
- **Mutable class variable** — a list or dict as a class variable is shared across all instances. Initialize it in `__init__` instead
- **`__init__` returning a value** — must return `None`. Returning anything else raises `TypeError`
- **`@property` without setter** — assigning to it raises `AttributeError: can't set attribute`. Define a setter explicitly if needed
