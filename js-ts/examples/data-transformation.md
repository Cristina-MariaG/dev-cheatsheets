# Example — Data Transformation

> Real-world data manipulation using array methods.

---

## Normalize an API response

```ts
interface ApiUser {
  user_id: number;
  full_name: string;
  is_active: number;   // 0 or 1
}

interface User {
  id: number;
  name: string;
  active: boolean;
}

const users: User[] = apiResponse.map(u => ({
  id: u.user_id,
  name: u.full_name,
  active: u.is_active === 1,
}));
```

---

## Group items by a property

```ts
const orders = [
  { id: 1, status: 'paid',    amount: 50 },
  { id: 2, status: 'pending', amount: 30 },
  { id: 3, status: 'paid',    amount: 20 },
];

const byStatus = orders.reduce<Record<string, typeof orders>>((acc, order) => {
  acc[order.status] ??= [];
  acc[order.status].push(order);
  return acc;
}, {});

// { paid: [{...}, {...}], pending: [{...}] }
```

---

## Aggregate totals

```ts
const totalByStatus = orders.reduce<Record<string, number>>((acc, order) => {
  acc[order.status] = (acc[order.status] ?? 0) + order.amount;
  return acc;
}, {});

// { paid: 70, pending: 30 }
```

---

## Filter, transform, sort — chained

```ts
const topActiveUsers = users
  .filter(u => u.active)                         // keep active only
  .map(u => ({ ...u, score: computeScore(u) }))  // add computed field
  .sort((a, b) => b.score - a.score)             // sort descending
  .slice(0, 10);                                  // top 10
```

---

## Flatten nested data

```ts
const categories = [
  { name: 'Fruits', items: ['apple', 'banana'] },
  { name: 'Vegs',   items: ['carrot', 'potato'] },
];

const allItems = categories.flatMap(c => c.items);
// ['apple', 'banana', 'carrot', 'potato']

// With metadata
const withCategory = categories.flatMap(c =>
  c.items.map(item => ({ item, category: c.name }))
);
// [{ item: 'apple', category: 'Fruits' }, ...]
```

---

## Deduplicate an array

```ts
// Primitive values
const unique = [...new Set([1, 2, 2, 3, 3])];   // [1, 2, 3]

// Objects — deduplicate by property
const uniqueUsers = users.filter(
  (user, index, arr) => arr.findIndex(u => u.id === user.id) === index
);
```
