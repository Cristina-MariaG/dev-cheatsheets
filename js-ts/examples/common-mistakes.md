# Example — Common Mistakes

> Frequent JS/TS bugs and how to fix them.

---

## Mutating state directly

```js
// Bug — mutates the original array, React won't detect the change
state.items.push(newItem);
setState(state);

// Fix — create a new array
setState({ ...state, items: [...state.items, newItem] });
```

---

## async/await in forEach

```js
// Bug — forEach doesn't await, all items process concurrently without control
items.forEach(async (item) => {
  await save(item);
});
console.log('done'); // prints before save() completes

// Fix — sequential
for (const item of items) {
  await save(item);
}

// Fix — parallel with control
await Promise.all(items.map(item => save(item)));
```

---

## Comparing objects and arrays

```js
// Bug — objects are compared by reference, not value
const a = { id: 1 };
const b = { id: 1 };
a === b; // false

// Fix — compare specific properties
a.id === b.id;
// or use a deep comparison library (lodash.isEqual)
```

---

## sort() without comparator

```js
// Bug — sort converts to strings: 10 comes before 2
[10, 2, 1].sort();         // [1, 10, 2]

// Fix
[10, 2, 1].sort((a, b) => a - b);  // [1, 2, 10] ascending
[10, 2, 1].sort((a, b) => b - a);  // [10, 2, 1] descending
```

---

## Floating point arithmetic

```js
// Bug
0.1 + 0.2 === 0.3;   // false (0.30000000000000004)

// Fix — compare with a tolerance
Math.abs(0.1 + 0.2 - 0.3) < Number.EPSILON;

// Or round before comparing
parseFloat((0.1 + 0.2).toFixed(10)) === 0.3;
```

---

## TypeScript — as type assertion hiding a bug

```ts
// Bug — forces TS to treat the value as User with no verification
const user = JSON.parse(data) as User;
user.email.toLowerCase();  // runtime crash if email is undefined

// Fix — validate before asserting
function isUser(obj: unknown): obj is User {
  return typeof obj === 'object' && obj !== null
    && 'id' in obj && 'email' in obj;
}

const parsed = JSON.parse(data);
if (isUser(parsed)) {
  parsed.email.toLowerCase();  // safe
}
```
