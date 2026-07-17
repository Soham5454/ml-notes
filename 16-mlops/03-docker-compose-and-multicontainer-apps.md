# 03 — Docker Compose & Multi-container Apps

Recap file 02's closing note — extending single-container Docker into orchestrating MULTIPLE containers together, genuinely the realistic shape of most real ML applications' infrastructure (an app, a database, sometimes a separate model-serving service).

## Why a real app genuinely needs multiple containers

```
Recap Flask file 09's SQLAlchemy/database integration directly -- a real app
genuinely needs:
1. The Flask/API application itself
2. A DATABASE (PostgreSQL in production, recap Flask file 11's note)
3. Sometimes a SEPARATE caching layer (Redis) or a dedicated model-serving container
Each of these genuinely runs BEST as its own isolated container, but they need
to communicate with each other -- Docker Compose is the tool for orchestrating this
```

## A complete `docker-compose.yml` — recap Flask's whole app, containerized properly

```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/mydb      # genuinely important -- 'db' is the SERVICE
                                                                    # name, not 'localhost'
      - SECRET_KEY=${SECRET_KEY}      # pulled from the host's environment or a .env file
    depends_on:
      - db
    volumes:
      - ./app:/app      # recap this file's volumes discussion below

  db:
    image: postgres:15
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=mydb
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

Genuinely important, easy-to-miss detail worth flagging directly: inside `DATABASE_URL`, the host is `db` — the SERVICE NAME defined in this same file — NOT `localhost`. Docker Compose automatically creates an internal network where each service can reach the others BY SERVICE NAME, genuinely a real, important networking convention worth understanding rather than memorizing.

## Running the whole stack

```bash
docker-compose up                # builds (if needed) and starts ALL services together
docker-compose up -d               # detached mode, recap file 02's -d flag
docker-compose down                  # stops and removes all containers (but keeps named volumes by default)
docker-compose logs -f web             # follow logs for a SPECIFIC service
```

Genuinely the whole point of Compose — instead of manually running `docker run` multiple times with carefully matched network/environment flags (genuinely error-prone and hard to reproduce), ONE `docker-compose.yml` file declaratively describes the ENTIRE multi-container application, and `docker-compose up` brings it all up correctly, every time.

## Volumes — genuinely essential for data persistence

```yaml
volumes:
  - postgres_data:/var/lib/postgresql/data      # a NAMED volume, managed by Docker, persists across container restarts
  - ./app:/app                                     # a BIND MOUNT -- maps a LOCAL directory into the container
```

Genuinely important distinction worth being precise about: containers are, by design, EPHEMERAL — recap the general "containers are lightweight, disposable instances" framing from file 02. Without a volume, a database's DATA would genuinely be LOST every time its container is removed and recreated. Named volumes (like `postgres_data` above) persist data independently of any specific container's lifecycle — genuinely essential for any service holding real, important state.

Bind mounts (`./app:/app`) are genuinely useful specifically DURING DEVELOPMENT — code changes on the host machine are immediately reflected inside the running container WITHOUT needing to rebuild the image, a real, practical development speed improvement. Worth knowing bind mounts are typically REMOVED for production deployments (recap file 02's multi-stage build discussion) in favor of code being fully baked into the image at build time.

## Networking between services — genuinely automatic, worth understanding why

```python
# Inside the Flask app's code, connecting to the database:
DATABASE_URL = "postgresql://user:pass@db:5432/mydb"      # 'db' resolves automatically
                                                                # to the db service's container
```

Genuinely worth knowing WHY this works without any manual network configuration — Docker Compose automatically creates an internal, ISOLATED network for all services defined in the same `docker-compose.yml`, and Docker's internal DNS resolves each SERVICE NAME to that service's container IP address automatically. This is precisely why hardcoding `localhost` (which would refer to the CONTAINER itself, not the other services) is a genuinely common, real mistake for people new to Compose.

## Adding a model-serving service — connecting directly to Flask/PyTorch

```yaml
services:
  web:
    build: ./web
    ports:
      - "8000:8000"
    depends_on:
      - model-server

  model-server:
    build: ./model_server      # recap PyTorch file 18's deployment discussion, or a TorchServe container
    ports:
      - "8080:8080"
```

Genuinely a realistic, important architectural pattern worth knowing — SEPARATING the web application (Flask, handling requests/templates/auth) from a DEDICATED model-serving container (potentially using TorchServe, recap Flask file 12's bridge discussion, or a specialized GPU-enabled container) lets each piece scale INDEPENDENTLY — the web tier might need many lightweight instances, while the model-serving tier might need fewer, GPU-equipped instances, and Compose (or Kubernetes, file 11) genuinely enables this separation cleanly.

## Health checks — genuinely important for reliable multi-service startup

```yaml
services:
  db:
    image: postgres:15
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 5s
      timeout: 5s
      retries: 5

  web:
    build: .
    depends_on:
      db:
        condition: service_healthy      # genuinely waits for db to be ACTUALLY ready, not just started
```

Genuinely an important, real fix for a common problem: `depends_on` alone only waits for a container to START, not for the SERVICE INSIDE it to be genuinely ready to accept connections (a database container can be "running" for a moment before PostgreSQL itself has finished initializing). `condition: service_healthy` combined with a proper healthcheck (recap Flask file 06's `/health` endpoint concept, applied here to the database itself) genuinely solves this race condition.

## Scaling services — a genuinely important preview of Kubernetes concepts (file 11)

```bash
docker-compose up --scale web=3      # genuinely runs 3 instances of the 'web' service
```

Genuinely worth knowing this exists as a simple preview — running MULTIPLE instances of the same service for load distribution is precisely the core idea that Kubernetes (file 11) formalizes and automates at a much larger, production-grade scale. Docker Compose's `--scale` flag is a genuinely useful, simple way to experience this concept before the added complexity of a full orchestration platform.

## Try this

```yaml
# Extend the Dockerfile from file 02's exercise into a full docker-compose.yml:
# a 'web' service (the Flask app), a 'db' service (postgres), with proper
# environment variables, a named volume for database persistence, and a
# healthcheck ensuring the web service waits for the database to be truly ready
# Run docker-compose up, confirm the Flask app can successfully connect to
# the database service using the service name (not localhost)
# Then run docker-compose down, followed by docker-compose up again, and
# confirm the database DATA persisted across that full teardown/recreation
# cycle -- directly proving the named volume is working as intended
```

Actually tearing down and recreating the whole stack, then confirming data persisted, is genuinely the clearest proof that the volume configuration is correct — a real, common point of confusion worth verifying hands-on rather than assuming.

## What's next
File 04 — Experiment Tracking with MLflow, moving from INFRASTRUCTURE (Docker/Compose) into the genuinely essential discipline of tracking every training run, directly answering file 01's "which of these 47 notebook runs was the best one" problem.
