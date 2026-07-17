# 02 — Docker Fundamentals

Recap file 01's closing note and Flask file 11's brief Docker introduction — going properly deep into containerization, genuinely the packaging foundation nearly everything else in this MLOps folder builds on.

## Containers vs Virtual Machines — genuinely important to understand the distinction

```
Virtual Machine: emulates an ENTIRE computer, including its own OS kernel --
  genuinely heavy (gigabytes), slow to start (minutes)
Container: shares the HOST machine's OS kernel, only packages the APPLICATION
  and its dependencies -- genuinely lightweight (megabytes), fast to start (seconds)
```

Genuinely the core reason Docker became so dominant — containers give MOST of a VM's isolation/reproducibility benefits (recap Flask file 11's "works on my machine" motivation) at a FRACTION of the resource cost, since they don't need to boot an entire separate operating system.

## Images vs Containers — a genuinely important distinction, easy to conflate

```bash
docker build -t my-app .        # builds an IMAGE -- a genuinely static, read-only TEMPLATE
docker run my-app                 # creates and runs a CONTAINER -- a genuinely running INSTANCE of that image
```

Recap the general class-vs-instance distinction from OOP (touched on throughout this roadmap) — an IMAGE is genuinely like a CLASS (a blueprint), and a CONTAINER is like an INSTANCE (an actual running object created from that blueprint). Multiple containers can be run from the SAME image, each an independent, isolated instance.

## Dockerfile — recap Flask file 11's example, now explained line by line

```dockerfile
FROM python:3.11-slim              # base image -- start from an existing, official Python image

WORKDIR /app                          # sets the working directory INSIDE the container

COPY requirements.txt .                 # copy just this file first -- genuinely a deliberate optimization, covered below
RUN pip install --no-cache-dir -r requirements.txt

COPY . .                                   # copy the REST of the application code

EXPOSE 8000                                  # documents which port the app listens on (doesn't actually publish it)

CMD ["gunicorn", "--bind", "0.0.0.0:8000", "run:app"]      # the command that runs when a container STARTS
```

## Layer caching — genuinely worth understanding WHY the Dockerfile is ordered this way

```dockerfile
COPY requirements.txt .                 # Step A
RUN pip install -r requirements.txt       # Step B
COPY . .                                     # Step C
```

Genuinely important, real optimization: Docker builds an image in LAYERS, and CACHES each layer — if `requirements.txt` hasn't changed since the last build, Docker REUSES the cached result of Step B (skipping the potentially slow `pip install` entirely), even if the actual application CODE (Step C) has changed. This is precisely WHY `requirements.txt` gets copied and installed BEFORE the rest of the code — if the code were copied first, ANY code change would invalidate the cache and force a full, slow reinstall of every dependency on every single build.

## Building and running — the core commands

```bash
docker build -t my-ml-app:1.0 .          # -t tags the image with a name:version
docker run -p 8000:8000 my-ml-app:1.0      # -p maps HOST_PORT:CONTAINER_PORT

docker run -d -p 8000:8000 my-ml-app:1.0     # -d runs it DETACHED, in the background
docker ps                                        # lists running containers
docker stop <container_id>                          # stops a running container
docker logs <container_id>                              # recap Flask file 11's logging discussion --
                                                             # genuinely the way to see a container's output
```

## Environment variables in Docker — recap Flask file 11's secrets discipline

```bash
docker run -e SECRET_KEY="genuinely-random-value" -e DATABASE_URL="..." my-ml-app:1.0

# Or via a .env file (genuinely convenient, keeps secrets out of shell history)
docker run --env-file .env my-ml-app:1.0
```

Genuinely worth flagging directly: the SAME "never hardcode secrets" principle from Flask files 08/10/11 applies identically here — never `COPY` a `.env` file containing real secrets INTO the image itself (recap the general credential-safety habit) — secrets should be injected at RUNTIME via `-e`/`--env-file`, never baked into the image where they'd be visible to anyone who obtains the image.

## Images for ML/deep learning — genuinely important, specific considerations

```dockerfile
# A genuinely common gotcha: the DEFAULT python:3.11-slim image does NOT include
# CUDA/GPU support -- for PyTorch models needing GPU (recap PyTorch file 13),
# a specialized base image is genuinely necessary:
FROM pytorch/pytorch:2.1.0-cuda11.8-cudnn8-runtime

# OR, for a lighter CPU-only deployment (genuinely fine for many production
# inference workloads where training-time GPU speed isn't needed):
FROM python:3.11-slim
RUN pip install torch --index-url https://download.pytorch.org/whl/cpu
```

Genuinely important, real practical distinction — recap PyTorch file 13's GPU discussion directly: a CUDA-enabled base image is dramatically LARGER (often several GB) than a CPU-only one, and genuinely only needed if the container will actually run on GPU hardware. Many production INFERENCE workloads (as opposed to training) run acceptably on CPU, especially for smaller models, making the CPU-only image a genuinely reasonable default unless GPU speed is specifically required.

## .dockerignore — genuinely important, easy to forget

```
# .dockerignore
venv/
__pycache__/
*.pyc
.git/
.env
*.log
tests/
```

Recap the general `.gitignore` habit from this whole roadmap's Git workflow — `.dockerignore` works identically, EXCLUDING files from being copied into the image during `docker build`. Genuinely important for both IMAGE SIZE (no reason to bloat the image with the local git history or virtual environment) and SECURITY (accidentally including a local `.env` with real secrets into a shared image would be a genuine, serious leak).

## Multi-stage builds — a genuinely important optimization for smaller images

```dockerfile
# Stage 1: build dependencies
FROM python:3.11 AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user -r requirements.txt

# Stage 2: genuinely minimal final image, only copying what's actually needed
FROM python:3.11-slim
COPY --from=builder /root/.local /root/.local
COPY . /app
WORKDIR /app
ENV PATH=/root/.local/bin:$PATH
CMD ["gunicorn", "run:app"]
```

Genuinely worth knowing this pattern exists — the FIRST stage might need heavier build tools (compilers, dev headers) to install certain packages, but the FINAL image only needs the RESULTING installed packages, not the build tools themselves. Multi-stage builds let the final shipped image be genuinely smaller and more secure (fewer unnecessary tools present) than a single-stage build would produce.

## Inspecting and debugging containers

```bash
docker exec -it <container_id> /bin/bash      # genuinely useful -- open an interactive shell
                                                    # INSIDE a running container for debugging
docker inspect <container_id>                        # detailed metadata about a container
```

Genuinely a real, practical debugging technique — when a containerized app behaves unexpectedly, `docker exec` lets the actual file system, environment variables, and running processes INSIDE the container be inspected directly, often revealing exactly what a purely external view (logs alone) might miss.

## Try this

```dockerfile
# Take the Flask ML app from that whole folder's exercises and write a genuinely
# optimized Dockerfile: correct layer ordering (requirements before code),
# a .dockerignore excluding venv/git/cache files, and environment variables
# for SECRET_KEY and DATABASE_URL passed at runtime rather than hardcoded
# Build the image, tag it with a version (e.g. my-app:1.0), and run it locally
# Then make a SMALL code change (not touching requirements.txt), rebuild, and
# time how much faster the rebuild is compared to the first build -- confirm
# the dependency-installation layer was genuinely reused from cache
```

Actually timing the cached vs uncached rebuild is genuinely the clearest way to feel WHY layer ordering matters, rather than just accepting it as a stated best practice.

## What's next
File 03 — Docker Compose & Multi-container Apps, extending single-container Docker into orchestrating MULTIPLE containers together (an app, a database, a model-serving service) — genuinely the realistic shape of most real ML applications' infrastructure.
