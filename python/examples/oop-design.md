# Example — OOP Design

> A complete object-oriented design: an e-commerce product catalog.
> Demonstrates classes, inheritance, ABC, properties, dataclasses, and dunders.

---

## Domain model

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass, field
from functools import total_ordering
from typing import Iterator


# --- Base product ---

class Product(ABC):
    def __init__(self, sku: str, name: str, price: float) -> None:
        self.sku = sku
        self.name = name
        self.price = price     # goes through setter
        self._reviews: list[float] = []

    @property
    def price(self) -> float:
        return self._price

    @price.setter
    def price(self, value: float) -> None:
        if value < 0:
            raise ValueError(f"Price cannot be negative: {value}")
        self._price = value

    @property
    def rating(self) -> float | None:
        if not self._reviews:
            return None
        return sum(self._reviews) / len(self._reviews)

    def add_review(self, score: float) -> None:
        if not (1 <= score <= 5):
            raise ValueError("Score must be between 1 and 5")
        self._reviews.append(score)

    @abstractmethod
    def category(self) -> str: ...

    @abstractmethod
    def is_available(self) -> bool: ...

    def __repr__(self) -> str:
        return f"{type(self).__name__}(sku={self.sku!r}, name={self.name!r}, price={self.price})"

    def __str__(self) -> str:
        rating = f" ★{self.rating:.1f}" if self.rating else ""
        return f"[{self.sku}] {self.name} — ${self.price:.2f}{rating}"


# --- Concrete products ---

class PhysicalProduct(Product):
    def __init__(self, sku: str, name: str, price: float, stock: int) -> None:
        super().__init__(sku, name, price)
        self.stock = stock

    def category(self) -> str:
        return "physical"

    def is_available(self) -> bool:
        return self.stock > 0

    def sell(self, quantity: int = 1) -> None:
        if quantity > self.stock:
            raise ValueError(f"Only {self.stock} in stock")
        self.stock -= quantity


class DigitalProduct(Product):
    def __init__(self, sku: str, name: str, price: float, download_url: str) -> None:
        super().__init__(sku, name, price)
        self.download_url = download_url

    def category(self) -> str:
        return "digital"

    def is_available(self) -> bool:
        return True   # digital products never run out


class SubscriptionProduct(Product):
    def __init__(self, sku: str, name: str, price: float, period_days: int) -> None:
        super().__init__(sku, name, price)
        self.period_days = period_days
        self._active = True

    def category(self) -> str:
        return "subscription"

    def is_available(self) -> bool:
        return self._active

    def deactivate(self) -> None:
        self._active = False
```

---

## Catalog — container with dunder methods

```python
@total_ordering
class Catalog:
    def __init__(self, name: str) -> None:
        self.name = name
        self._products: dict[str, Product] = {}

    def add(self, product: Product) -> None:
        self._products[product.sku] = product

    def remove(self, sku: str) -> Product:
        return self._products.pop(sku)

    def __getitem__(self, sku: str) -> Product:
        if sku not in self._products:
            raise KeyError(f"Product {sku!r} not found")
        return self._products[sku]

    def __contains__(self, sku: str) -> bool:
        return sku in self._products

    def __len__(self) -> int:
        return len(self._products)

    def __iter__(self) -> Iterator[Product]:
        return iter(self._products.values())

    def __eq__(self, other: object) -> bool:
        if not isinstance(other, Catalog):
            return NotImplemented
        return self.name == other.name

    def __lt__(self, other: "Catalog") -> bool:
        if not isinstance(other, Catalog):
            return NotImplemented
        return len(self) < len(other)

    def __repr__(self) -> str:
        return f"Catalog({self.name!r}, {len(self)} products)"

    def available(self) -> list[Product]:
        return [p for p in self if p.is_available()]

    def by_category(self, category: str) -> list[Product]:
        return [p for p in self if p.category() == category]

    def search(self, query: str) -> list[Product]:
        q = query.lower()
        return [p for p in self if q in p.name.lower()]
```

---

## Discount mixin

```python
class DiscountMixin:
    """Adds percentage-based discount capability to any product."""

    def apply_discount(self, percent: float) -> None:
        if not (0 < percent < 100):
            raise ValueError("Discount must be between 0 and 100")
        self.price = self.price * (1 - percent / 100)

    def discounted_price(self, percent: float) -> float:
        return self.price * (1 - percent / 100)


class SaleProduct(DiscountMixin, PhysicalProduct):
    pass
```

---

## Usage

```python
catalog = Catalog("Main Store")

catalog.add(PhysicalProduct("A001", "Keyboard", 79.99, stock=50))
catalog.add(PhysicalProduct("A002", "Mouse", 39.99, stock=0))
catalog.add(DigitalProduct("D001", "Python Course", 29.99, "https://..."))
catalog.add(SubscriptionProduct("S001", "Pro Plan", 9.99, period_days=30))

# Container behavior
len(catalog)           # 4
"A001" in catalog      # True
catalog["D001"]        # DigitalProduct(...)

# Iteration
for product in catalog:
    print(product)

# Filtering
catalog.available()            # excludes Mouse (stock=0)
catalog.by_category("digital") # [Python Course]
catalog.search("pro")          # [Pro Plan]

# Reviews
kb = catalog["A001"]
kb.add_review(4.5)
kb.add_review(5.0)
kb.rating   # 4.75

# Sale item with mixin
sale_kb = SaleProduct("A001-SALE", "Keyboard (Sale)", 79.99, stock=10)
sale_kb.apply_discount(20)
sale_kb.price   # 63.99
```
