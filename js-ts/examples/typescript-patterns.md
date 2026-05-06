# Example — TypeScript Patterns

> Common patterns in TypeScript projects.

---

## Typed API response

```ts
interface User {
  id: number;
  name: string;
  email: string;
}

async function getUser(id: number): Promise<User> {
  const res = await fetch(`/api/users/${id}`);
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return res.json() as Promise<User>;
}
```

---

## Discriminated union — model state

```ts
type RequestState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: string };

function render(state: RequestState<User>) {
  switch (state.status) {
    case 'loading': return 'Loading...';
    case 'success': return state.data.name;  // TypeScript knows data exists here
    case 'error':   return state.error;      // TypeScript knows error exists here
    default:        return null;
  }
}
```

---

## Generic utility function

```ts
function groupBy<T, K extends string>(
  items: T[],
  key: (item: T) => K
): Record<K, T[]> {
  return items.reduce((acc, item) => {
    const group = key(item);
    acc[group] = [...(acc[group] ?? []), item];
    return acc;
  }, {} as Record<K, T[]>);
}

const byStatus = groupBy(orders, o => o.status);
// { pending: [...], paid: [...], cancelled: [...] }
```

---

## Type guard

```ts
interface Dog { type: 'dog'; bark(): void; }
interface Cat { type: 'cat'; meow(): void; }
type Animal = Dog | Cat;

function isDog(animal: Animal): animal is Dog {
  return animal.type === 'dog';
}

function makeSound(animal: Animal) {
  if (isDog(animal)) animal.bark();
  else animal.meow();
}
```

---

## Extend a third-party type

```ts
// Add a property to Express's Request type
declare global {
  namespace Express {
    interface Request {
      user?: User;
    }
  }
}

// Now req.user is typed everywhere
app.get('/profile', (req, res) => {
  res.json(req.user);
});
```
