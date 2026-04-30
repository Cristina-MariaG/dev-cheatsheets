# Docker — Compose

> Define and run multi-container applications from a single YAML file.

---

## Commands

```bash
docker compose up                       # create and start all services
docker compose up -d                    # detached mode
docker compose up --build               # rebuild images before starting
docker compose down                     # stop and remove containers + networks
docker compose down -v                  # also remove named volumes
docker compose ps                       # list running services
docker compose logs                     # show logs for all services
docker compose logs -f <service>        # follow logs for a specific service
docker compose exec <service> bash      # open a shell in a running service
docker compose build                    # build/rebuild images without starting
docker compose pull                     # pull latest images for all services
docker compose restart <service>        # restart a specific service
```

> Modern Docker uses `docker compose` (plugin, no hyphen). The older standalone `docker-compose` still works but is no longer maintained.

---

## docker-compose.yml structure

```yaml
services:
  app:
    build: .                            # build from local Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgres://db:5432/mydb
    depends_on:
      - db
    volumes:
      - .:/app                          # bind mount for dev (live reload)
    networks:
      - backend

  db:
    image: postgres:16
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: user
      POSTGRES_PASSWORD: secret
    volumes:
      - db-data:/var/lib/postgresql/data  # named volume for persistence
    networks:
      - backend

volumes:
  db-data:                              # declare named volumes here

networks:
  backend:                              # declare custom networks here
```

---

## Environment variables

```bash
# .env file (auto-loaded by Compose)
POSTGRES_PASSWORD=secret
NODE_ENV=development
```

```yaml
# Reference in compose file
environment:
  - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
```

> `.env` is loaded automatically if it's in the same directory as `docker-compose.yml`. Add it to `.gitignore`.

---

## Common pitfalls

- `depends_on` only waits for the container to start, not for the service inside to be ready — use a health check or a wait script for databases
- Bind mounts in dev (`./:/app`) override the image's files — make sure `node_modules` isn't overwritten by adding a named volume for it
- Forgetting `down -v` when resetting a database — the old data persists in the named volume
