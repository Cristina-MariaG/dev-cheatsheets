# Example — Dockerfiles

> Real-world examples, construction rules, and security practices.

---

## Rules for a good Dockerfile

### 1. Order layers by frequency of change (least → most)

Docker caches each layer. If a layer changes, all layers after it are rebuilt.

```dockerfile
# Good — dependencies cached separately from source code
COPY package*.json ./
RUN npm ci
COPY . .          # source changes often, but deps layer is reused

# Bad — any source change invalidates the npm ci layer
COPY . .
RUN npm ci
```

### 2. Use specific tags — never `latest` in production

```dockerfile
FROM node:20.12-alpine3.19   # pinned — reproducible
# not: FROM node:latest      # unpredictable, can silently break
```

### 3. Combine related RUN commands to reduce layers

```dockerfile
# Good — one layer, cache cleaned in the same step
RUN apt-get update \
    && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*

# Bad — 3 layers, cache not cleaned
RUN apt-get update
RUN apt-get install -y curl
RUN rm -rf /var/lib/apt/lists/*
```

### 4. Always use `.dockerignore`

```
node_modules
.env
.git
*.log
dist
coverage
.DS_Store
```

> Without `.dockerignore`, `COPY . .` sends everything to the build context — including secrets, git history, and large directories.

### 5. Use multi-stage builds for compiled languages and frontends

Keeps the final image lean — no build tools, no dev dependencies.

### 6. Set WORKDIR explicitly

```dockerfile
WORKDIR /app     # clean, predictable path
# not: RUN mkdir /app && cd /app  — verbose and error-prone
```

### 7. Prefer COPY over ADD

Use `ADD` only when you need tar extraction or URL fetching. For everything else, `COPY` is explicit and safer.

---

## Security rules

### Never run as root

```dockerfile
# Create a non-root user and switch to it
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser
```

> If a process inside the container is compromised, running as root gives the attacker full container control and potential host escape.

### Never store secrets in ENV or ARG

```dockerfile
# Bad — secrets visible in docker history and image layers
ENV DB_PASSWORD=secret
ARG API_KEY=secret

# Good — inject at runtime
docker run -e DB_PASSWORD=secret myapp
# or use Docker secrets / a secrets manager
```

> `docker history <image>` reveals all ENV and ARG values. Anyone with access to the image can read them.

### Use minimal base images

| Base | Size | Use case |
|---|---|---|
| `alpine` | ~5MB | General purpose, most languages |
| `distroless` | ~2MB | Production — no shell, no package manager |
| `scratch` | 0MB | Static binaries only (Go, Rust) |

> Fewer packages = smaller attack surface. Distroless images have no shell, so `docker exec` won't work — great for production, harder to debug.

### Scan your images

```bash
# Docker Scout (built into Docker Desktop)
docker scout cves <image>

# Trivy (open source, CI-friendly)
trivy image <image>
```

### Make the filesystem read-only at runtime

```bash
docker run --read-only --tmpfs /tmp myapp
```

> Prevents an attacker from writing malicious files inside the container. Combine with `--tmpfs` for directories that legitimately need writes.

---

## Node.js — production

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY --from=builder /app/dist ./dist
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

---

## Python — FastAPI

```dockerfile
FROM python:3.12-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

FROM python:3.12-slim
WORKDIR /app
COPY --from=builder /install /usr/local
COPY . .
RUN addgroup --system appgroup && adduser --system --ingroup appgroup appuser
USER appuser
EXPOSE 8000
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

> `--no-cache-dir` avoids storing pip's download cache in the image. `--prefix=/install` collects installed packages in one directory for easy copying to the final stage.

---

## Go — static binary

```dockerfile
FROM golang:1.22-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o server .

# scratch — zero-dependency base, smallest possible image
FROM scratch
COPY --from=builder /app/server /server
EXPOSE 8080
ENTRYPOINT ["/server"]
```

> `CGO_ENABLED=0` produces a fully static binary with no libc dependency — compatible with `scratch`. Final image is just the binary.

---

## Static frontend — React / Vue

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

```nginx
# nginx.conf — SPA routing (redirect all routes to index.html)
server {
    listen 80;
    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

