# Example — API Service

> A typed, reusable API service in TypeScript with error handling and response typing.

---

## Base HTTP client

```ts
class HttpError extends Error {
  constructor(
    public statusCode: number,
    message: string
  ) {
    super(message);
    this.name = 'HttpError';
  }
}

async function request<T>(url: string, options?: RequestInit): Promise<T> {
  const response = await fetch(url, {
    headers: { 'Content-Type': 'application/json' },
    ...options,
  });

  if (!response.ok) {
    throw new HttpError(response.status, `Request failed: ${response.status} ${response.statusText}`);
  }

  return response.json() as Promise<T>;
}
```

---

## Typed service

```ts
interface User {
  id: number;
  name: string;
  email: string;
}

interface CreateUserPayload {
  name: string;
  email: string;
}

const userService = {
  getAll(): Promise<User[]> {
    return request<User[]>('/api/users');
  },

  getById(id: number): Promise<User> {
    return request<User>(`/api/users/${id}`);
  },

  create(payload: CreateUserPayload): Promise<User> {
    return request<User>('/api/users', {
      method: 'POST',
      body: JSON.stringify(payload),
    });
  },

  update(id: number, payload: Partial<CreateUserPayload>): Promise<User> {
    return request<User>(`/api/users/${id}`, {
      method: 'PATCH',
      body: JSON.stringify(payload),
    });
  },

  delete(id: number): Promise<void> {
    return request<void>(`/api/users/${id}`, { method: 'DELETE' });
  },
};
```

---

## Usage with error handling

```ts
async function loadUserProfile(id: number) {
  try {
    const user = await userService.getById(id);
    return user;
  } catch (err) {
    if (err instanceof HttpError && err.statusCode === 404) {
      return null;   // user not found — not an error in this context
    }
    throw err;       // unexpected error — propagate
  }
}

// Load multiple resources in parallel
async function loadDashboard(userId: number) {
  const [user, posts] = await Promise.all([
    userService.getById(userId),
    postService.getByUser(userId),
  ]);

  return { user, posts };
}
```

---

## With authentication header

```ts
function createAuthenticatedClient(getToken: () => string) {
  return async function request<T>(url: string, options?: RequestInit): Promise<T> {
    const response = await fetch(url, {
      ...options,
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${getToken()}`,
        ...options?.headers,
      },
    });

    if (response.status === 401) {
      throw new HttpError(401, 'Unauthorized — token may have expired');
    }

    if (!response.ok) {
      throw new HttpError(response.status, response.statusText);
    }

    return response.json() as Promise<T>;
  };
}

const request = createAuthenticatedClient(() => localStorage.getItem('token') ?? '');
```
