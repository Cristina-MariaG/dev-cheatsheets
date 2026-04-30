# Docker — Basics

> Core concepts and essential commands to get started.

---

## Key concepts

### Image
A read-only, layered blueprint of an environment — OS, dependencies, config, app code. Built from a Dockerfile. Once built, it doesn't change.

Think of it like a class in OOP, or a VM snapshot: it defines what the environment looks like, but it doesn't run anything by itself.

### Container
A running instance of an image. Docker takes the image, adds a thin writable layer on top, and starts a process inside an isolated environment (its own filesystem, network, and process space).

You can run many containers from the same image simultaneously — each is independent. Stop a container and its writable layer is gone (unless you used a volume).

### Image layers
Images are built in layers — each instruction in a Dockerfile adds one. Layers are cached and shared between images. If two images share the same base (`FROM node:20-alpine`), that base is stored only once on disk.

```
FROM node:20-alpine    ← layer 1 (shared, cached)
WORKDIR /app           ← layer 2
COPY package*.json ./  ← layer 3
RUN npm ci             ← layer 4 (cached until package.json changes)
COPY . .               ← layer 5
```

### Registry
A server that stores and distributes images. Docker Hub is the default public registry. Private registries (AWS ECR, GitHub Container Registry, etc.) are common in teams.

`docker pull` downloads from a registry. `docker push` uploads to one.

### Dockerfile
A text file with instructions that describe how to build an image step by step. Each instruction creates a layer.

### Compose
A tool to define and run multiple containers together from a single `docker-compose.yml` file. Handles networks, volumes, dependencies, and environment variables — one command to start the whole stack.

---

> **The key mental model:** an image is static (like a class), a container is dynamic (like an instance). The image defines the environment, the container runs in it.

---

## Install check

```bash
docker --version                # check Docker version
docker info                     # full system info (engine, storage, containers)
docker run hello-world          # verify the installation works end-to-end
```

---

## First container

```bash
# Run an interactive Ubuntu container (removed after exit)
docker run -it --rm ubuntu bash

# Run a detached Nginx server on port 8080
docker run -d -p 8080:80 --name my-nginx nginx
```

---

## Common pitfalls

- Confusing images and containers — `docker ps` shows running containers, `docker images` shows local images
- Forgetting `--rm` and accumulating stopped containers (`docker ps -a` to see them)
- Running as root inside containers — use `USER` in Dockerfile for production images
