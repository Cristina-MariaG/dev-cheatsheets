# JavaScript — Core

> Modern JavaScript syntax and patterns for daily use.

---

## Variables

Three ways to declare variables — `var` is legacy, use `const` and `let`.

```js
const name = 'Alice';      // immutable binding — use by default
let count = 0;             // mutable — use when reassignment is needed
var old = true;            // function-scoped, hoisted — avoid in modern code
```

> `const` doesn't mean the value is immutable — an object declared with `const` can still be mutated. It means the variable can't be reassigned.

---

## Functions

```js
// Function declaration — hoisted, available before its definition in the file
function greet(name) {
  return `Hello, ${name}`;
}

// Arrow function — shorter syntax, no own `this`
const greet = (name) => `Hello, ${name}`;
const double = n => n * 2;           // parentheses optional for single param
const noop = () => {};               // no params

// Default parameters
function createUser(name, role = 'user') {
  return { name, role };
}

// Rest parameters — collect remaining args into an array
function sum(...numbers) {
  return numbers.reduce((a, b) => a + b, 0);
}
```

---

## Destructuring

Extract values from objects or arrays into variables.

```js
// Object destructuring
const { name, age } = user;
const { name: userName } = user;          // rename
const { name, age = 25 } = user;          // default value if undefined

// Array destructuring
const [first, second] = items;
const [first, , third] = items;           // skip an element
const [head, ...tail] = items;            // rest

// In function parameters
function display({ name, age }) {
  console.log(name, age);
}
```

---

## Spread and rest

```js
// Spread — expand an iterable
const merged = { ...defaults, ...overrides };   // merge objects (right wins)
const copy = [...array];                        // shallow copy
const combined = [...arr1, ...arr2];            // concatenate

// Rest — collect into array/object
const { id, ...rest } = user;                   // remove a property
```

---

## Optional chaining and nullish coalescing

```js
// Optional chaining — stops if null/undefined instead of throwing
const city = user?.address?.city;
const first = users?.[0];
const result = obj?.method?.();

// Nullish coalescing — fallback only for null/undefined (not 0 or '')
const name = user.name ?? 'Anonymous';

// vs OR operator — fallback for any falsy value (including 0 and '')
const name = user.name || 'Anonymous';   // 0 and '' would trigger fallback
```

---

## Array methods

```js
const numbers = [1, 2, 3, 4, 5];

numbers.map(n => n * 2);               // [2, 4, 6, 8, 10] — transform each element
numbers.filter(n => n > 2);            // [3, 4, 5] — keep matching elements
numbers.reduce((acc, n) => acc + n, 0); // 15 — accumulate into a single value
numbers.find(n => n > 3);             // 4 — first match or undefined
numbers.findIndex(n => n > 3);        // 3 — index of first match or -1
numbers.some(n => n > 4);             // true — at least one matches
numbers.every(n => n > 0);            // true — all match
numbers.includes(3);                  // true — contains value
numbers.flat();                       // flatten one level
numbers.flatMap(n => [n, n * 2]);     // map + flatten one level
```

> These methods return a new array — they don't mutate the original. Exception: `sort()` and `reverse()` mutate in place.

---

## Template literals

```js
const msg = `Hello, ${name}!`;                // interpolation
const multiline = `
  Line 1
  Line 2
`;
const tag = String.raw`C:\Users\${name}`;     // tagged template — raw string
```

---

## Short-circuit patterns

```js
// Execute only if condition is truthy
isLoggedIn && showDashboard();

// Return first truthy value
const display = nickname || username || 'Anonymous';

// Conditional object property
const obj = {
  name: 'Alice',
  ...(isAdmin && { role: 'admin' }),   // only add role if isAdmin
};
```

---

## Common pitfalls

- `const` with objects — the binding is immutable but the content isn't. Use `Object.freeze()` for deep immutability
- `sort()` without comparator — converts to strings: `[10, 2, 1].sort()` → `[1, 10, 2]`. Always pass a comparator for numbers: `.sort((a, b) => a - b)`
- Arrow functions and `this` — arrow functions capture `this` from the surrounding scope, they don't have their own. Don't use them as object methods or constructors
