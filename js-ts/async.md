# JavaScript — Async

> Handling asynchronous operations with callbacks, promises, and async/await.

---

## Concept

JavaScript is single-threaded — it can only do one thing at a time. Async operations (network requests, timers, file I/O) are offloaded and their results handled via callbacks, promises, or async/await when they complete.

---

## Callbacks

The original pattern — pass a function to be called when the operation completes.

```js
setTimeout(() => console.log('done'), 1000);

fs.readFile('file.txt', 'utf8', (err, data) => {
  if (err) throw err;
  console.log(data);
});
```

> Callbacks nest deeply for sequential operations — this is "callback hell". Promises solve this.

---

## Promises

A Promise represents a value that will be available in the future. It can be pending, fulfilled, or rejected.

```js
fetch('/api/users')
  .then(response => response.json())     // runs if fulfilled
  .then(data => console.log(data))
  .catch(err => console.error(err))      // runs if any step rejects
  .finally(() => setLoading(false));     // always runs
```

### Creating a promise

```js
const delay = (ms) => new Promise(resolve => setTimeout(resolve, ms));

const fetchUser = (id) => new Promise((resolve, reject) => {
  if (!id) reject(new Error('id required'));
  else resolve({ id, name: 'Alice' });
});
```

---

## async/await

Syntactic sugar over promises — makes async code read like synchronous code.

```js
async function loadUser(id) {
  const response = await fetch(`/api/users/${id}`);

  if (!response.ok) {
    throw new Error(`HTTP error: ${response.status}`);
  }

  return response.json();
}
```

`await` pauses execution inside the async function until the promise resolves. It doesn't block the rest of the program.

### Error handling with try/catch

```js
async function loadUser(id) {
  try {
    const response = await fetch(`/api/users/${id}`);
    const data = await response.json();
    return data;
  } catch (err) {
    console.error('Failed to load user:', err);
    throw err;   // re-throw if caller needs to handle it
  }
}
```

---

## Running multiple promises

```js
// All in parallel — waits for all, fails if any fails
const [users, posts] = await Promise.all([
  fetch('/api/users').then(r => r.json()),
  fetch('/api/posts').then(r => r.json()),
]);

// First to resolve wins
const result = await Promise.race([fetchFast(), fetchSlow()]);

// All settle — never rejects, returns status of each
const results = await Promise.allSettled([p1, p2, p3]);
results.forEach(r => {
  if (r.status === 'fulfilled') console.log(r.value);
  else console.error(r.reason);
});

// First to succeed — ignores rejections until all fail
const first = await Promise.any([p1, p2, p3]);
```

---

## Common pitfalls

- **Missing await** — `const data = fetch(url)` returns a Promise, not the data. Easy to miss in complex code
- **Sequential awaits when parallel is possible** — two independent requests awaited one after the other take 2× longer than `Promise.all`

```js
// Slow — sequential
const user = await fetchUser(id);
const posts = await fetchPosts(id);

// Fast — parallel
const [user, posts] = await Promise.all([fetchUser(id), fetchPosts(id)]);
```

- **Unhandled promise rejections** — always add `.catch()` or `try/catch`. In Node.js, unhandled rejections crash the process in recent versions
- **async/await in forEach** — `forEach` doesn't await the callbacks. Use `for...of` or `Promise.all` with `map`

```js
// Bug — won't wait for async operations
items.forEach(async (item) => await process(item));

// Fix
for (const item of items) await process(item);           // sequential
await Promise.all(items.map(item => process(item)));     // parallel
```
