---
globs:
  - '**/Dockerfile*'
  - '**/docker-compose*.yml'
  - '**/docker-compose*.yaml'
  - '**/.dockerignore'
---
# Docker Rules

## Build
- Always use multi-stage builds to minimize final image size
- Pin base image versions — never use `latest` (e.g., `python:3.12-slim`, not `python:latest`)
- Copy dependency files first, then code (layer cache optimization):
  ```dockerfile
  COPY requirements.txt .
  RUN pip install -r requirements.txt
  COPY . .
  ```
- Combine `RUN` commands to minimize layers
- Use `COPY` over `ADD` unless tar extraction is needed

## Security
- Run as non-root user:
  ```dockerfile
  RUN adduser --disabled-password --gecos '' appuser
  USER appuser
  ```
- Never copy `.env`, secrets, or credentials into images
- Scan images with `trivy` before pushing to registry

## Health & Runtime
- Include `HEALTHCHECK` instruction for production images
- Use `ENTRYPOINT` for the main process, `CMD` for default arguments
- Expose only necessary ports with `EXPOSE`

## Docker Compose
- Pin service image versions
- Use `env_file` for environment variables, not inline
- Define `healthcheck` for dependent services
- Use `depends_on` with `condition: service_healthy`

## .dockerignore
- Always include: `.git`, `node_modules`, `.env*`, `*.log`, `__pycache__`, `.terraform`
- Keep in sync with build context needs

## Optimization
- Use BuildKit (`DOCKER_BUILDKIT=1`) for parallel stage builds
- Use `--cache-from` and `--cache-to` for CI cache reuse
- Target `linux/amd64` for production deployments
