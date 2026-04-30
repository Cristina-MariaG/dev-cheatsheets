# Example — Multi-Container App

> Web app + database + reverse proxy, all wired together with Compose.

---

## Architecture

```
Internet → Nginx (80/443) → App (3000) → Postgres (5432)
```

---

## docker-compose.yml

```yaml
services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
    depends_on:
      - app
    networks:
      - frontend

  app:
    build: .
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgres://user:secret@db:5432/mydb
    depends_on:
      db:
        condition: service_healthy       # wait for db to pass health check
    networks:
      - frontend
      - backend

  db:
    image: postgres:16
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: user
      POSTGRES_PASSWORD: secret
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d mydb"]
      interval: 5s
      timeout: 5s
      retries: 5

volumes:
  db-data:

networks:
  frontend:                              # nginx ↔ app
  backend:                               # app ↔ db (not exposed to nginx)
```

---

## nginx.conf

```nginx
server {
    listen 80;

    location / {
        proxy_pass http://app:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

---

## Deploy

```bash
# Build and start all services
docker compose up -d --build

# Check everything is running
docker compose ps

# Watch logs
docker compose logs -f

# Scale the app (multiple instances behind nginx)
docker compose up -d --scale app=3
```

---

**Why separate networks?**
`db` is only reachable from `app` — nginx has no direct path to the database. Each service only sees what it needs.

**Why a health check on db?**
`depends_on` alone only waits for the container to start, not for Postgres to be ready to accept connections. The health check with `condition: service_healthy` actually waits for the database to be ready.
