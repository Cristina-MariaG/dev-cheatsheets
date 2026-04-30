# Docker — Images

> Building, tagging, and managing images.

---

## Registry commands

```bash
docker pull <image>:<tag>               # download an image (default tag: latest)
docker push <image>:<tag>               # push to a registry
docker images                           # list local images
docker image rm <image>                 # remove an image
docker image prune                      # remove all dangling images
docker image prune -a                   # remove all unused images
```

---

## Build

```bash
docker build -t <name>:<tag> .          # build from Dockerfile in current dir
docker build -t <name>:<tag> -f <path>  # build from a specific Dockerfile
docker build --no-cache -t <name> .     # build without cache
```

---

## Tag

```bash
docker tag <source>:<tag> <target>:<tag>  # create an alias for an image
```

> Useful before pushing to a registry: `docker tag myapp:latest registry.io/myapp:1.0`

---

## Dockerfile essentials

```dockerfile
FROM node:20-alpine             # base image (always start here)
WORKDIR /app                    # set working directory for subsequent commands
COPY package*.json ./           # copy specific files
COPY . .                        # copy everything else
RUN npm ci                      # run a command during build
ENV NODE_ENV=production         # set an environment variable
EXPOSE 3000                     # document which port the app listens on (informational only)
CMD ["node", "server.js"]       # default command when container starts
```

### CMD vs ENTRYPOINT

| | CMD | ENTRYPOINT |
|---|---|---|
| Purpose | Default command — easily overridden | Main process — always runs |
| Override | `docker run <image> <new-command>` replaces it | `docker run --entrypoint <cmd> <image>` replaces it |
| Use case | Flexible containers | Fixed-purpose containers |

> Combine both: `ENTRYPOINT ["node"]` + `CMD ["server.js"]` — the CMD becomes default args to the entrypoint.

### COPY vs ADD

| | COPY | ADD |
|---|---|---|
| Copies local files | ✅ | ✅ |
| Extracts tar archives | ✗ | ✅ |
| Fetches remote URLs | ✗ | ✅ |

> Prefer `COPY` — it's explicit. Use `ADD` only when you need its extra features.

---

## Multi-stage builds

Reduce final image size by separating build from runtime:

```dockerfile
# Stage 1 — build
FROM node:20 AS builder
WORKDIR /app
COPY package*.json ./           # copy deps manifest first to leverage cache
RUN npm ci                      # install all deps (including devDependencies)
COPY . .
RUN npm run build               # produce /app/dist

# Stage 2 — runtime (lean image, no build tools or devDependencies)
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev           # install production deps only
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/server.js"]
```

---

## Common pitfalls

- Using `latest` tag in production — not reproducible, can silently break
- Copying everything before installing dependencies — breaks layer caching (`COPY package*.json` first, then `RUN npm ci`, then `COPY . .`)
- Large images from forgetting to clean up after `RUN apt-get install` — add `&& rm -rf /var/lib/apt/lists/*`
