# TypeScript — Types and Patterns

> Static typing for JavaScript — catch errors at compile time instead of runtime.

---

## Basic types

```ts
const name: string = 'Alice';
const age: number = 30;
const active: boolean = true;
const id: string | number = 42;        // union type
const nothing: null = null;
const missing: undefined = undefined;
const anything: unknown = getData();   // safer than any — must narrow before using
const unsafe: any = getData();         // disables type checking — avoid
```

---

## Arrays and tuples

```ts
const names: string[] = ['Alice', 'Bob'];
const ids: Array<number> = [1, 2, 3];

// Tuple — fixed-length array with specific types per position
const point: [number, number] = [10, 20];
const entry: [string, number] = ['age', 30];
```

---

## Type aliases

Name a type for reuse.

```ts
type UserId = string;
type Status = 'active' | 'inactive' | 'pending';   // literal union
type Callback = (err: Error | null, data: string) => void;
```

---

## Interfaces

Describe the shape of an object. Extendable and mergeable.

```ts
interface User {
  id: number;
  name: string;
  email?: string;        // optional property
  readonly createdAt: Date;  // can't be reassigned after creation
}

// Extend an interface
interface Admin extends User {
  role: 'admin' | 'superadmin';
}
```

### Interface vs Type alias

| | `interface` | `type` |
|---|---|---|
| Object shape | ✅ | ✅ |
| Union types (`A \| B`) | ✗ | ✅ |
| Intersection | ✅ (`extends`) | ✅ (`&`) |
| Declaration merging | ✅ | ✗ |
| Use case | Objects, classes, APIs | Everything else |

---

## Generics

Write code that works with any type while staying type-safe.

```ts
// Generic function
function first<T>(arr: T[]): T | undefined {
  return arr[0];
}

first([1, 2, 3]);        // returns number | undefined
first(['a', 'b']);       // returns string | undefined

// Generic interface
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

// Generic with constraint — T must have a name property
function getName<T extends { name: string }>(obj: T): string {
  return obj.name;
}
```

---

## Utility types

Built-in generic types that transform existing types.

```ts
interface User {
  id: number;
  name: string;
  email: string;
  role: string;
}

Partial<User>              // all properties optional
Required<User>             // all properties required
Readonly<User>             // all properties read-only

Pick<User, 'id' | 'name'>  // keep only these properties → { id, name }
Omit<User, 'role'>         // remove these properties → { id, name, email }

Record<string, number>     // { [key: string]: number }
Record<'admin' | 'user', User>  // specific keys

ReturnType<typeof fetchUser>    // infer return type of a function
Parameters<typeof fetchUser>    // infer parameter types of a function

NonNullable<string | null | undefined>  // string
```

---

## Type narrowing

TypeScript narrows the type based on runtime checks.

```ts
function process(value: string | number) {
  if (typeof value === 'string') {
    return value.toUpperCase();   // TypeScript knows it's a string here
  }
  return value * 2;               // TypeScript knows it's a number here
}

// instanceof narrowing
function handle(error: unknown) {
  if (error instanceof Error) {
    console.log(error.message);   // TypeScript knows it's an Error
  }
}

// Type guard function
function isUser(obj: unknown): obj is User {
  return typeof obj === 'object' && obj !== null && 'id' in obj;
}
```

---

## Common pitfalls

- Using `any` — it disables all type checking on that value. Prefer `unknown` and narrow the type
- `as` type assertions — `data as User` tells TypeScript to trust you without checking. A lie to the compiler is a runtime error waiting to happen
- Forgetting to handle `undefined` — optional chaining (`?.`) and nullish coalescing (`??`) are your friends
