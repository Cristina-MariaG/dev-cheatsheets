# Example — Local Dev Environment

> Containerized development setup with live reload, no local installs needed.

---

## Project structure

```
myapp/
├── Dockerfile
├── docker-compose.yml
├── .env
├── .dockerignore
└── src/
```

---

## Dockerfile

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
EXPOSE 3000
CMD ["npm", "run", "dev"]
```

---

## docker-compose.yml

```yaml
services:
  app:
    build: .
    ports:
      - "3000:3000"
    volumes:
      - .:/app                    # bind mount — live code changes without rebuild
      - /app/node_modules         # prevent host from overwriting container's node_modules
    env_file:
      - .env
    depends_on:
      - db

  db:
    image: postgres:16
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - db-data:/var/lib/postgresql/data
    ports:
      - "5432:5432"               # expose to host for local DB clients (dev only)

volumes:
  db-data:
```

---

## .env

```bash
POSTGRES_DB=mydb
POSTGRES_USER=user
POSTGRES_PASSWORD=secret
```

> Add `.env` to `.gitignore` — never commit credentials.

---

## .dockerignore

```
node_modules
.env
.git
*.log
```

---

## Workflow

```bash
# Start everything
docker compose up -d

# Watch logs
docker compose logs -f app

# Open a shell in the app container
docker compose exec app sh

# Run a one-off command (e.g. migrations)
docker compose exec app npm run migrate

# Stop and clean up
docker compose down

# Full reset (delete database too)
docker compose down -v
```
