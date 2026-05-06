# TypeScript — tsconfig.json

> Key compiler options explained.

---

## Minimal config to start

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "NodeNext",
    "strict": true,
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src"]
}
```

---

## Output options

| Option | What it does |
|---|---|
| `target` | JS version to compile to (`ES2020`, `ES2022`, `ESNext`) |
| `module` | Module system (`CommonJS` for Node, `ESNext` for bundlers, `NodeNext` for modern Node) |
| `outDir` | Where compiled `.js` files go |
| `rootDir` | Root of your source files — mirrors structure in `outDir` |
| `declaration` | Generate `.d.ts` type declaration files (useful for libraries) |
| `sourceMap` | Generate `.map` files for debugging in original TS source |
| `removeComments` | Strip comments from output |

---

## Type checking options

| Option | What it does |
|---|---|
| `strict` | Enables all strict checks below — always use this |
| `strictNullChecks` | `null` and `undefined` are not assignable to other types |
| `noImplicitAny` | Error when a variable implicitly gets type `any` |
| `strictFunctionTypes` | Stricter checking for function parameter types |
| `noUncheckedIndexedAccess` | Array/object access returns `T \| undefined` instead of `T` |
| `noImplicitReturns` | Error if a function doesn't return in all code paths |
| `noUnusedLocals` | Error on unused local variables |
| `noUnusedParameters` | Error on unused function parameters |

> `"strict": true` is shorthand for enabling: `strictNullChecks`, `noImplicitAny`, `strictFunctionTypes`, `strictBindCallApply`, `strictPropertyInitialization`, `noImplicitThis`, `alwaysStrict`.

---

## Module resolution

| Option | What it does |
|---|---|
| `moduleResolution` | How imports are resolved (`Node`, `NodeNext`, `Bundler`) |
| `baseUrl` | Base for non-relative imports |
| `paths` | Aliases for imports (`@/components` → `./src/components`) |
| `esModuleInterop` | Allows `import x from 'module'` for CommonJS modules |
| `allowSyntheticDefaultImports` | Allow default imports from modules with no default export |

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

---

## Project references and includes

```json
{
  "include": ["src/**/*"],          // files to compile
  "exclude": ["node_modules", "dist", "**/*.test.ts"],
  "compilerOptions": {
    "skipLibCheck": true            // skip type checking of .d.ts in node_modules (faster builds)
  }
}
```

---

## Configs for common setups

### Node.js (modern)
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "outDir": "dist",
    "rootDir": "src",
    "declaration": true,
    "sourceMap": true
  }
}
```

### Frontend with bundler (Vite, webpack)
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "strict": true,
    "jsx": "react-jsx",
    "esModuleInterop": true,
    "skipLibCheck": true
  }
}
```
