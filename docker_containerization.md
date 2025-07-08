# Docker Containerization Documentation

## Overview

This document describes the Docker containerization setup for the **Finch** project, covering:

- Dockerfile structure for the **frontend**, **backend**, and **PostgreSQL database**.
- Image optimization techniques.
- `.dockerignore` usage.
- Docker Compose setup for local development.
- How to build and run images locally.

---

## Frontend Dockerfile (Vue.js)

**Path:** `./finch-frontend/Dockerfile`

### Structure

1. **Stage 1 – Build:**

   - Base: `node:18-alpine`
   - Installs dependencies and builds production assets into `/app/dist`

2. **Stage 2 – Serve:**

   - Base: `nginx:alpine`
   - Serves static files from the `dist` directory using Nginx

### Optimization Techniques

- **Multi-stage builds** minimize the final image size
- **Alpine base images** ensure a smaller footprint
- **Build caching** by copying only `package*.json` first before source files

---

## Backend Dockerfile (Python/Django)

**Path:** `./finch-backend/Dockerfile`

### Structure

1. **Build Stage:**

   - Base: `python:3.11-slim`
   - Installs build tools and creates a virtual environment
   - Installs Python dependencies from `requirements.txt`

2. **Runtime Stage:**

   - Also uses `python:3.11-slim`
   - Installs only runtime packages (e.g., `libpq5`)
   - Copies virtual environment and code from builder
   - Uses a non-root user (`appuser`) for enhanced security

### Optimization Techniques

- **Slim base image** and **virtual environment reuse**
- **Layered caching** for dependencies
- **Non-root user** to reduce container risk surface

---

## PostgreSQL Setup (Docker Compose Only)

**Image:** `postgres:15`

### Key Settings

- Custom DB name, user, and password
- Exposed port: `5432`
- Healthcheck ensures app containers wait for DB readiness
- Data persisted via named Docker volume (`postgres_data`)

---

## .dockerignore Usage

Both frontend and backend projects include `.dockerignore` files.

### Recommended Entries:

```plaintext
node_modules
.venv
__pycache__
*.pyc
.dockerignore
Dockerfile
docker-compose.yml
.env
.git
.gitignore
dist
*.log
```

Benefits:

- Prevents large or sensitive files from being sent to Docker daemon
- Speeds up build time
- Reduces image size

---

## Docker Compose for Local Development

**Path:** `docker-compose.yml` (project root or infrastructure repo)

### Services:

- **db:** PostgreSQL database with persistence and health check
- **web:** Backend Django application (waits for healthy DB)
- **frontend:** Vue.js app served via Nginx

### Sample Commands:

```bash
# Start all services with rebuild
docker-compose up --build

# Stop services
docker-compose down
```

---

## Building and Running Individually

### Frontend (Vue.js)

```bash
cd finch-frontend

docker build -t finch-frontend:local .
docker run -p 3000:80 finch-frontend:local
```

### Backend (Django)

```bash
cd finch-backend

docker build -t finch-backend:local .
docker run --env-file .env -p 8000:8000 finch-backend:local
```

### PostgreSQL (Optional)

Handled via Docker Compose. Manual startup example:

```bash
docker run --name pg-db -e POSTGRES_DB=fleetdb -e POSTGRES_USER=roy77 -e POSTGRES_PASSWORD=asdf1234@77 -p 5432:5432 -d postgres:15
```

---

## Conclusion

This containerization setup provides:

- Lean, secure, and production-ready Docker images
- Reproducible local development via Docker Compose
- Proper separation of concerns with frontend, backend, and DB services

For any deployment pipelines, these same Dockerfiles can be reused in CI/CD workflows without modification.

