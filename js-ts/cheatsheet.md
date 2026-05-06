# JavaScript / TypeScript — Cheatsheet

> Most used patterns, grouped by situation.

---

## Variables & syntax

```js
const name = 'Alice';                        // immutable binding
let count = 0;                               // mutable
const { id, name: userName } = user;         // destructuring + rename
const { a, ...rest } = obj;                  // remove a property
const merged = { ...defaults, ...override }; // merge objects
const copy = [...array, newItem];            // copy + append
```

---

## Functions

```js
const fn = (x, y = 0) => x + y;             // arrow + default param
const sum = (...nums) => nums.reduce((a, b) => a + b, 0);  // rest params
const double = n => n * 2;                   // single param, implicit return
```

---

## Array methods

```js
arr.map(x => x * 2)           // transform
arr.filter(x => x > 0)        // keep matching
arr.reduce((acc, x) => acc + x, 0)  // accumulate
arr.find(x => x.id === id)    // first match or undefined
arr.some(x => x.active)       // at least one matches
arr.every(x => x > 0)         // all match
arr.flat()                    // flatten one level
arr.flatMap(x => [x, x * 2]) // map + flatten
```

---

## Optional chaining & nullish coalescing

```js
const city = user?.address?.city;      // stops if null/undefined
const name = user.name ?? 'Anonymous'; // fallback for null/undefined only
```

---

## Async / await

```js
const data = await fetch(url).then(r => r.json());

// Error handling
try {
  const data = await fetchUser(id);
} catch (err) {
  console.error(err);
}

// Parallel requests
const [users, posts] = await Promise.all([fetchUsers(), fetchPosts()]);

// All settle (never throws)
const results = await Promise.allSettled([p1, p2, p3]);
```

---

## TypeScript

```ts
type Status = 'active' | 'inactive';       // literal union
interface User { id: number; name: string; email?: string; }
function first<T>(arr: T[]): T | undefined { return arr[0]; }

Partial<User>                // all optional
Omit<User, 'email'>          // remove property
Pick<User, 'id' | 'name'>   // keep only these
Record<string, number>       // { [key: string]: number }
ReturnType<typeof fetchUser> // infer return type
```

---

## Type narrowing

```ts
if (typeof value === 'string') { /* value is string here */ }
if (value instanceof Error)   { /* value is Error here */ }
if ('id' in obj)              { /* obj has id property */ }
```
