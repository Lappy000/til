# Docker BuildKit Cache Mounts

## Problem

Every `pip install` in Docker rebuilds from scratch when deps change.

## Solution: --mount=type=cache

```dockerfile
# syntax=docker/dockerfile:1
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .

RUN --mount=type=cache,target=/root/.cache/pip \
    pip install -r requirements.txt

COPY . .
```

## Go Example

```dockerfile
FROM golang:1.22
WORKDIR /app
COPY go.mod go.sum ./

RUN --mount=type=cache,target=/go/pkg/mod \
    --mount=type=cache,target=/root/.cache/go-build \
    go mod download

COPY . .
RUN --mount=type=cache,target=/root/.cache/go-build \
    go build -o /bin/app ./cmd/app
```

## Key Details

- Requires DOCKER_BUILDKIT=1 (default Docker 23+)
- Cache persists across builds on same host
- NOT included in final image
- Typical improvement: 60-90% faster rebuilds
