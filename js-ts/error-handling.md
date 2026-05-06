# JavaScript — Error Handling

> Throwing, catching, and propagating errors correctly.

---

## The Error object

```js
const err = new Error('Something went wrong');
err.message;   // 'Something went wrong'
err.name;      // 'Error'
err.stack;     // stack trace string
```

### Built-in error types

```js
new TypeError('Expected a string');      // wrong type
new RangeError('Value out of bounds');   // number out of range
new ReferenceError('x is not defined'); // undefined variable
new SyntaxError('Unexpected token');    // invalid syntax
```

---

## try / catch / finally

```js
try {
  const data = JSON.parse(input);       // might throw SyntaxError
  process(data);
} catch (err) {
  console.error('Parsing failed:', err.message);
} finally {
  cleanup();                            // always runs, even if error was thrown
}
```

> `finally` runs regardless of whether an error occurred — use it to release resources (close connections, hide loaders, etc.).

---

## Custom errors

Extend `Error` to create domain-specific error types.

```js
class NotFoundError extends Error {
  constructor(resource, id) {
    super(`${resource} with id ${id} not found`);
    this.name = 'NotFoundError';
    this.statusCode = 404;
  }
}

class ValidationError extends Error {
  constructor(message, fields) {
    super(message);
    this.name = 'ValidationError';
    this.fields = fields;
    this.statusCode = 400;
  }
}

// Usage
throw new NotFoundError('User', 42);

// Catch specific type
try {
  await getUser(id);
} catch (err) {
  if (err instanceof NotFoundError) {
    return res.status(404).json({ message: err.message });
  }
  throw err;   // re-throw unknown errors
}
```

---

## Error handling in async code

```js
// async/await — use try/catch
async function loadData(id) {
  try {
    const data = await fetchData(id);
    return data;
  } catch (err) {
    // handle or re-throw
    throw new Error(`Failed to load data: ${err.message}`);
  }
}

// Promises — use .catch()
fetchData(id)
  .then(process)
  .catch(err => console.error(err));
```

---

## Re-throwing — when to catch and when to propagate

```js
async function getUser(id) {
  try {
    return await db.findUser(id);
  } catch (err) {
    // Only handle what you can — re-throw the rest
    if (err instanceof DatabaseConnectionError) {
      await reconnect();
      return await db.findUser(id);    // retry once
    }
    throw err;                         // not our problem — let it bubble up
  }
}
```

> Catching everything and swallowing errors silently is one of the most common bugs. If you don't know how to handle an error, let it propagate to the layer that does.

---

## TypeScript — typing errors

In TypeScript, `catch (err)` gives `err` type `unknown` — you must narrow it before using.

```ts
try {
  await riskyOperation();
} catch (err) {
  if (err instanceof Error) {
    console.error(err.message);     // safe
  } else {
    console.error('Unknown error:', err);
  }
}

// Utility to always get an Error object
function toError(value: unknown): Error {
  if (value instanceof Error) return value;
  return new Error(String(value));
}
```

---

## Common pitfalls

- **Empty catch block** — silently swallows the error. At minimum, log it
- **Catching `Error` but checking `.message`** — fragile, message strings can change. Use `instanceof` or a custom `code` property
- **`throw 'string'`** — throwing non-Error values loses the stack trace. Always `throw new Error('message')`
- **Unhandled promise rejections** — in Node.js, unhandled rejections crash the process in recent versions. Always attach `.catch()` or use `try/catch`
