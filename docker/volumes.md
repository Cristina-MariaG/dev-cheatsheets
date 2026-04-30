# Docker — Volumes

> Persisting and sharing data between containers and the host.

---

## Three types of mounts

| Type | Syntax | Managed by | Use case |
|---|---|---|---|
| **Named volume** | `-v mydata:/app/data` | Docker | Database storage, persistent app data |
| **Bind mount** | `-v /host/path:/container/path` | Host OS | Dev workflows, live code reload |
| **tmpfs** | `--tmpfs /app/tmp` | Memory | Temporary data, secrets, sensitive files |

> Named volumes are stored in Docker's managed area (`/var/lib/docker/volumes/`). Bind mounts point directly to the host filesystem.

---

## Volume commands

```bash
docker volume create <name>             # create a named volume
docker volume ls                        # list all volumes
docker volume inspect <name>            # details (mount point, labels)
docker volume rm <name>                 # remove a volume
docker volume prune                     # remove all volumes not used by a container
```

---

## Mount syntax

```bash
# Named volume
docker run -v db-data:/var/lib/postgresql/data postgres

# Bind mount (absolute path required)
docker run -v /home/user/app:/app node

# Read-only bind mount
docker run -v /home/user/config:/app/config:ro myapp

# tmpfs
docker run --tmpfs /app/tmp:size=100m myapp
```

---

## In docker-compose.yml

```yaml
services:
  db:
    image: postgres:16
    volumes:
      - db-data:/var/lib/postgresql/data   # named volume

  app:
    image: myapp
    volumes:
      - .:/app                             # bind mount (dev)
      - /app/node_modules                  # anonymous volume — prevents host from overwriting node_modules

volumes:
  db-data:                                 # must be declared at top level
```

---

## Common pitfalls

- Bind mounting `.:/app` in dev and overwriting `node_modules` from the image — add `- /app/node_modules` as an anonymous volume to protect it
- Removing a container thinking data is gone — named volumes persist until explicitly deleted
- Using bind mounts in production — paths are host-dependent and not portable
