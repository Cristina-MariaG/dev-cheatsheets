# Example — Networking

> Practical scenarios for container-to-container communication and network isolation.

---

## Two containers talking to each other

The default bridge network has no DNS — containers can't reach each other by name. Always use a custom network.

```bash
# Create a shared network
docker network create mynet

# Start a database
docker run -d --name db --network mynet postgres:16 \
  -e POSTGRES_USER=user \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=mydb

# Start the app on the same network
docker run -d --name app --network mynet myapp

# Inside the app container, reach the database by container name
docker exec -it app psql -h db -U user mydb
```

> Container name `db` resolves automatically via Docker's built-in DNS — no IP needed.

---

## Isolating services (frontend / backend / database)

```
Browser → frontend (80) → backend (3000) → db (5432)
```

```bash
docker network create frontend-net
docker network create backend-net

# db is only on backend-net
docker run -d --name db --network backend-net postgres:16

# backend is on both networks
docker run -d --name backend --network backend-net mybackend
docker network connect frontend-net backend

# frontend is only on frontend-net
docker run -d --name frontend -p 80:80 --network frontend-net mynginx
```

`db` is completely unreachable from `frontend` — it has no interface on `frontend-net`.

---

## Inspect and debug connectivity

```bash
# See which networks a container is connected to
docker inspect --format '{{json .NetworkSettings.Networks}}' <container>

# Check a container's IP on a specific network
docker network inspect mynet

# Test connectivity from inside a container
docker exec -it app ping db
docker exec -it app curl http://backend:3000/health

# Check open ports on a container
docker exec -it app ss -tlnp
# or if ss is not available:
docker exec -it app netstat -tlnp
```

---

## Common scenario — app can't reach database

```bash
# 1. Confirm both containers are running
docker ps

# 2. Check they're on the same network
docker network inspect mynet

# 3. Try pinging from app to db
docker exec -it app ping db

# 4. Check the database is actually listening
docker exec -it db pg_isready

# 5. Check environment variables in the app
docker exec -it app env | grep DATABASE
```

> Most "can't connect to database" issues come from: wrong network, wrong hostname, or the database not being ready yet (use health checks in Compose).
