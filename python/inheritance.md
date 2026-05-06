# Python — Inheritance

> Single inheritance, super(), multiple inheritance, MRO, ABC, Protocol, mixins.

---

## Single inheritance

```python
class Animal:
    def __init__(self, name: str) -> None:
        self.name = name

    def speak(self) -> str:
        raise NotImplementedError

    def __repr__(self) -> str:
        return f"{type(self).__name__}({self.name!r})"

class Dog(Animal):
    def speak(self) -> str:
        return f"{self.name} says woof!"

class Cat(Animal):
    def speak(self) -> str:
        return f"{self.name} says meow!"

dog = Dog("Rex")
dog.speak()               # "Rex says woof!"
isinstance(dog, Animal)   # True
isinstance(dog, Dog)      # True
type(dog) is Dog          # True (not Animal)
type(dog) is Animal       # False
```

---

## super()

Call a method from the parent class. Always use `super()` — never hardcode the parent class name.

```python
class Vehicle:
    def __init__(self, make: str, model: str) -> None:
        self.make = make
        self.model = model

    def describe(self) -> str:
        return f"{self.make} {self.model}"

class ElectricCar(Vehicle):
    def __init__(self, make: str, model: str, range_km: int) -> None:
        super().__init__(make, model)   # run Vehicle.__init__ first
        self.range_km = range_km       # then add the extra attribute

    def describe(self) -> str:
        base = super().describe()      # reuse parent's method
        return f"{base} (electric, {self.range_km} km range)"

car = ElectricCar("Tesla", "Model 3", 550)
car.describe()   # "Tesla Model 3 (electric, 550 km range)"
```

> Always call `super().__init__()` in the child's `__init__` to make sure the parent is fully initialized before you add child-specific attributes.

---

## Overriding methods

```python
class Shape:
    def area(self) -> float:
        return 0.0

class Rectangle(Shape):
    def __init__(self, width: float, height: float) -> None:
        self.width = width
        self.height = height

    def area(self) -> float:             # override
        return self.width * self.height

class Square(Rectangle):
    def __init__(self, side: float) -> None:
        super().__init__(side, side)     # reuse Rectangle.__init__
    # area() is inherited from Rectangle — no need to override
```

---

## Multiple inheritance

```python
class Flyable:
    def fly(self) -> str:
        return "I can fly"

class Swimmable:
    def swim(self) -> str:
        return "I can swim"

class Duck(Flyable, Swimmable):
    def quack(self) -> str:
        return "Quack!"

duck = Duck()
duck.fly()    # "I can fly"
duck.swim()   # "I can swim"
```

---

## Method Resolution Order (MRO)

When a method is called, Python walks the MRO — a linearized list of classes — until it finds the method. The order follows the **C3 linearization** algorithm: left-to-right among parents, parents always after their children.

```python
class A:
    def hello(self): return "A"

class B(A):
    def hello(self): return "B"

class C(A):
    def hello(self): return "C"

class D(B, C):
    pass

D().hello()    # "B" — follows MRO: D → B → C → A → object
D.__mro__      # (<class 'D'>, <class 'B'>, <class 'C'>, <class 'A'>, <class 'object'>)
```

This is the **diamond problem** solved: even though both B and C inherit from A, A only appears once in the MRO.

```python
# Always check MRO when debugging multiple inheritance
print(D.__mro__)
```

### Cooperative super() with multiple inheritance

For `super()` to work correctly across all classes in the MRO, every class must call `super().__init__()` — including mixins.

```python
class A:
    def __init__(self):
        super().__init__()    # must be here, even though A inherits from object
        print("A init")

class B(A):
    def __init__(self):
        super().__init__()
        print("B init")

class C(A):
    def __init__(self):
        super().__init__()
        print("C init")

class D(B, C):
    def __init__(self):
        super().__init__()
        print("D init")

D()
# A init → C init → B init → D init   (MRO order, reversed)
```

---

## Abstract Base Classes (ABC)

Force subclasses to implement specific methods. You cannot instantiate an abstract class.

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self) -> float: ...

    @abstractmethod
    def perimeter(self) -> float: ...

    def describe(self) -> str:
        return f"Area={self.area():.2f}, Perimeter={self.perimeter():.2f}"

class Circle(Shape):
    def __init__(self, radius: float) -> None:
        self.radius = radius

    def area(self) -> float:
        import math
        return math.pi * self.radius ** 2

    def perimeter(self) -> float:
        import math
        return 2 * math.pi * self.radius

# Shape()            # TypeError: Can't instantiate abstract class Shape
Circle(5).area()     # 78.54
```

> If a subclass doesn't implement all abstract methods, it's still abstract — it also cannot be instantiated.

---

## Protocol — structural subtyping

Define an interface by duck typing. A class satisfies a Protocol simply by having the right methods — no explicit inheritance needed.

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class Drawable(Protocol):
    def draw(self) -> None: ...
    def resize(self, factor: float) -> None: ...

class Circle:
    def draw(self) -> None:
        print("Drawing circle")

    def resize(self, factor: float) -> None:
        self.radius *= factor

class Square:
    def draw(self) -> None:
        print("Drawing square")

    def resize(self, factor: float) -> None:
        self.side *= factor

def render_all(shapes: list[Drawable]) -> None:
    for shape in shapes:
        shape.draw()

render_all([Circle(), Square()])   # works — no inheritance needed
isinstance(Circle(), Drawable)     # True — because of @runtime_checkable
```

| | ABC | Protocol |
|---|---|---|
| Subclass required | ✅ must inherit explicitly | ✗ just have the right methods |
| Enforcement | At class definition | At type-check time (mypy) |
| `isinstance` check | ✅ | ✅ with `@runtime_checkable` |
| When to use | You control the subclasses | You don't control the classes |

---

## Mixins

Small, focused classes that add a specific behavior. They are never instantiated alone.

```python
class JsonMixin:
    def to_json(self) -> str:
        import json
        return json.dumps(self.__dict__, default=str)

    @classmethod
    def from_json(cls, data: str) -> "JsonMixin":
        import json
        return cls(**json.loads(data))

class ValidateMixin:
    def validate(self) -> list[str]:
        errors = []
        for field, expected_type in self.__class__.__annotations__.items():
            value = getattr(self, field, None)
            if value is None:
                errors.append(f"{field} is required")
        return errors

class User(JsonMixin, ValidateMixin):
    name: str
    email: str

    def __init__(self, name: str, email: str) -> None:
        self.name = name
        self.email = email

user = User("Alice", "alice@x.com")
user.to_json()     # '{"name": "Alice", "email": "alice@x.com"}'
user.validate()    # []
```

> Mixins go **first** (leftmost) in the class definition so their methods take priority over the main parent.

---

## Common pitfalls

- **Calling parent by name** — `Vehicle.__init__(self, ...)` breaks with multiple inheritance. Always use `super()`
- **Forgetting `super()` in mixins** — if a mixin doesn't call `super().__init__()`, the rest of the MRO chain won't run
- **ABC vs Protocol** — use ABC when you own all the classes; Protocol when you want to type-check third-party classes
- **`isinstance` with Protocol** — only works if the Protocol has `@runtime_checkable`
