---
title: "Docker Build Cache and Multi-Stage Builds: Three Mechanisms"
date: 2026-08-31 10:46:52 +0900
categories: [Infra]
tags: [docker, build-cache, multi-stage, buildkit, image-size]
---

A Docker image is not one blob — it is a stack of read-only layers, roughly one per filesystem-changing instruction (`RUN`, `COPY`, `ADD`). Almost every "why is my build slow / my image huge" question reduces to three mechanisms. Naming the mechanism, not the symptom, is what actually makes this stick.

## 1. A cache miss propagates downward

On each build, Docker asks per layer: *is this cached?* The cache is a chain — once one layer misses, **every layer below it rebuilds too**. So order matters: rarely-changing steps go up top, frequently-changing steps go to the bottom.

The canonical win is *dependencies before source*:

```dockerfile
# Bad: editing one line of code reinstalls everything
COPY . .
RUN pip install -r requirements.txt

# Good: the install layer stays cached unless requirements.txt changes
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
```

`COPY . .` hashes the files it copies, so any code edit misses the cache and cascades into `pip install`. Lift the dependency step above the source copy and code edits no longer touch it.

A `.dockerignore` (dropping `.git`, `__pycache__`, `.venv`) keeps irrelevant files from busting that same cache.

## 2. A `RUN`'s cache key is the command string

Docker does **not** inspect what a `RUN` does — its cache key is the literal command text. So this is a trap:

```dockerfile
RUN apt-get update              # string never changes → cached forever → stale index
RUN apt-get install -y curl
```

Combine them so a single key covers both, and changing the install list re-runs the update:

```dockerfile
RUN apt-get update && apt-get install -y --no-install-recommends curl \
    && rm -rf /var/lib/apt/lists/*
```

BuildKit adds a cache mount that survives across builds without baking into the image — downloaded wheels are reused even when `requirements.txt` changes:

```dockerfile
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install -r requirements.txt
```

## 3. Multi-stage discards the build layers

Build time needs compilers and headers; runtime needs only the artifact. Use multiple `FROM`s and copy just the result forward:

```dockerfile
FROM python:3.12 AS builder
WORKDIR /build
COPY requirements.txt .
RUN pip wheel --no-cache-dir --wheel-dir /wheels -r requirements.txt

FROM python:3.12-slim
WORKDIR /app
COPY --from=builder /wheels /wheels
RUN pip install --no-cache-dir /wheels/*
COPY . .
CMD ["python", "app.py"]
```

The final image contains only the last stage's layers plus whatever `COPY --from` pulled in. Every build layer — compilers, source, intermediate files — is thrown away. The Go version of this pattern goes from ~800 MB (compiler included) to ~10 MB (just the binary).

## Checklist

- Dependencies before source; add a `.dockerignore`.
- One `RUN` for `apt-get update && install`, clean up `/var/lib/apt/lists`.
- Multi-stage + a `slim`/`distroless` base; `--no-install-recommends`, `--no-cache-dir`.
- Prefer `COPY` over `ADD` unless you need tar extraction or a remote URL.
