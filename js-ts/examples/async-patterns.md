# Example — Async Patterns

> Fetch, error handling, and parallel requests in practice.

---

## Basic fetch with error handling

```js
async function getUser(id) {
  const response = await fetch(`/api/users/${id}`);

  if (!response.ok) {
    throw new Error(`Request failed: ${response.status}`);
  }

  return response.json();
}
```

---

## Parallel independent requests

```js
async function loadDashboard(userId) {
  const [user, posts, notifications] = await Promise.all([
    getUser(userId),
    getPosts(userId),
    getNotifications(userId),
  ]);

  return { user, posts, notifications };
}
```

> Three requests run simultaneously instead of sequentially — total time = slowest request, not sum of all.

---

## Handle partial failures with allSettled

```js
async function loadWidgets() {
  const results = await Promise.allSettled([
    fetchSalesData(),
    fetchTrafficData(),
    fetchRevenueData(),
  ]);

  return results.map(result =>
    result.status === 'fulfilled' ? result.value : null
  );
}
```

> `Promise.all` fails entirely if one request fails. `Promise.allSettled` lets you handle each result individually.

---

## Retry on failure

```js
async function fetchWithRetry(url, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      const res = await fetch(url);
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      return await res.json();
    } catch (err) {
      if (i === retries - 1) throw err;
      await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
    }
  }
}
```

---

## Sequential processing of a list

```js
// Process items one by one (order preserved, slower)
for (const item of items) {
  await process(item);
}

// Process all in parallel (faster, order not guaranteed)
await Promise.all(items.map(item => process(item)));
```
