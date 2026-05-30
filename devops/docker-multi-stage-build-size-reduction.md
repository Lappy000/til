# Docker Multi-Stage Build Size Reduction

Reducing image size from 1.2GB to 45MB using multi-stage builds.

## Before (1.2GB)

```dockerfile
FROM python:3.12
COPY . /app
RUN pip install -r requirements.txt
CMD ["python", "/app/main.py"]
```

## After (45MB)

```dockerfile
# Stage 1: Builder
FROM python:3.12-slim AS builder
WORKDIR /build
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Stage 2: Runtime
FROM python:3.12-alpine
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY . .
ENV PATH=/root/.local/bin:$PATH
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
CMD ["python", "main.py"]
```

## Size Breakdown

| Image | Size |
|-------|------|
| python:3.12 | 1.2GB |
| python:3.12-slim | 150MB |
| python:3.12-alpine | 45MB |
| Final (alpine + deps) | 52MB |

## Tips

- Use `--no-cache-dir` with pip
- Use `--user` to install in `/root/.local` (copyable)
- Use `.dockerignore` to exclude tests, docs, .git
- Use `dive` tool to inspect layer sizes
