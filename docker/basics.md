# Docker — Basics

> Core concepts and essential commands to get started.

---

## Key concepts

| Concept | What it is |
|---|---|
| **Image** | Read-only blueprint — defines the environment (OS, dependencies, app code) |
| **Container** | Running instance of an image — isolated process on the host |
| **Registry** | Storage for images — Docker Hub is the default public registry |
| **Dockerfile** | Script that describes how to build an image |
| **Compose** | Tool to define and run multi-container apps from a single YAML file |

> An image is to a container what a class is to an object.

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
