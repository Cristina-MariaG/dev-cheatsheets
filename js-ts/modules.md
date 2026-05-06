# JavaScript — Modules

> ES modules, CommonJS, and import/export patterns.

---

## Concept

Modules allow splitting code into separate files. Each file has its own scope — nothing is global unless explicitly exported.

There are two module systems in use today:
- **ES Modules (ESM)** — the standard, uses `import`/`export`. Used in browsers and modern Node.js
- **CommonJS (CJS)** — the original Node.js system, uses `require`/`module.exports`. Still common in older packages

---

## Named exports

Export multiple things from a file. Import by exact name.

```js
// math.js
export function add(a, b) { return a + b; }
export function multiply(a, b) { return a * b; }
export const PI = 3.14159;

// main.js
import { add, multiply } from './math.js';
import { add as sum } from './math.js';       // rename on import
import * as math from './math.js';            // import everything as namespace
math.add(1, 2);
```

---

## Default export

One main export per file. Import under any name.

```js
// user.js
export default function createUser(name) {
  return { name, createdAt: new Date() };
}

// main.js
import createUser from './user.js';           // any name works
import makeUser from './user.js';             // same thing, different name
```

> A file can have both named and a default export, but keep it simple — mixing both can be confusing.

---

## Re-exporting — barrel files

Aggregate exports from multiple files into one entry point.

```js
// index.js — re-export everything from subdirectories
export { add, multiply } from './math.js';
export { default as createUser } from './user.js';
export * from './utils.js';

// consumers import from one place
import { add, createUser } from './lib/index.js';
```

> Barrel files are convenient but can hurt tree-shaking in bundlers — use with care in large codebases.

---

## Dynamic imports — load on demand

Load a module lazily at runtime instead of at startup.

```js
// Load only when needed (code splitting)
const { heavy } = await import('./heavy-library.js');

// Conditional import
if (process.env.NODE_ENV === 'development') {
  const { debug } = await import('./debug-tools.js');
}
```

---

## CommonJS (Node.js legacy)

```js
// Export
module.exports = { add, multiply };
module.exports.PI = 3.14;

// Import
const { add } = require('./math');
const math = require('./math');
```

---

## ESM vs CommonJS — key differences

| | ESM | CommonJS |
|---|---|---|
| Syntax | `import` / `export` | `require` / `module.exports` |
| Loading | Static, analyzed at parse time | Dynamic, executed at runtime |
| Top-level `await` | ✅ | ✗ |
| Tree-shaking friendly | ✅ | ✗ |
| File extension in Node | `.mjs` or `"type": "module"` in package.json | `.cjs` or default |

---

## Common pitfalls

- **Default export refactor trap** — renaming a default export doesn't break imports since the name is arbitrary. Named exports are safer for refactoring — rename one, the compiler catches all the import sites
- **Circular imports** — A imports B, B imports A. Often results in `undefined` at import time. Restructure to extract shared code into a third module
- **Missing file extension in ESM** — Node.js ESM requires explicit extensions: `import './utils.js'` not `import './utils'`
