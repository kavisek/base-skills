Audit the current project against foundational conventions, scaffold required structure, and produce a health report.

## Required Scaffolding (always set up if missing)

The following must exist in every project. Create any that are absent — do not ask for confirmation, just scaffold them:

### `Makefile`
Include at minimum:
```makefile
.PHONY: start stop clean test build logs

start:        ## Start all services
	docker compose up

stop:         ## Stop all services
	docker compose down

clean:        ## Remove containers, volumes, and build artifacts
	docker compose down -v --remove-orphans
	find . -type d -name __pycache__ -exec rm -rf {} + 2>/dev/null || true
	find . -name "*.pyc" -delete 2>/dev/null || true

build:        ## Build all service images
	docker compose build

test:         ## Run tests inside the container
	docker compose run --rm test

logs:         ## Tail logs for all services
	docker compose logs -f

help:         ## Show this help
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-15s\033[0m %s\n", $$1, $$2}'
```

### `docker-compose.yml`
- Mount source directories as volumes for hot reloading (e.g., `./backend:/app`, `./frontend:/app`)
- Include a `test` service that runs the test suite inside the container
- Define named volumes for database data persistence
- Use `depends_on` with `condition: service_healthy` for services that need a ready dependency
- Example structure:
```yaml
services:
  backend:
    build: ./backend
    volumes:
      - ./backend:/app        # hot reload
    ports:
      - "8000:8000"
    depends_on:
      db:
        condition: service_healthy

  frontend:
    build: ./frontend
    volumes:
      - ./frontend:/app       # hot reload
      - /app/node_modules     # preserve container node_modules
    ports:
      - "3000:3000"

  db:
    image: postgres:16
    volumes:
      - db_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready"]
      interval: 5s
      timeout: 5s
      retries: 5

  test:
    build: ./backend
    command: pytest
    volumes:
      - ./backend:/app
    depends_on:
      db:
        condition: service_healthy

volumes:
  db_data:
```
Adapt services to the actual stack; remove services that do not apply.

### `.devcontainer/devcontainer.json`
```json
{
  "name": "Dev Container",
  "dockerComposeFile": "../docker-compose.yml",
  "service": "backend",
  "workspaceFolder": "/app",
  "features": {},
  "postCreateCommand": "make build",
  "customizations": {
    "vscode": {
      "extensions": []
    }
  }
}
```

### `docs/` folder
Create with a `docs/README.md` that documents:
- Architecture overview (fill in based on what exists)
- How to run the project locally (`make start`)
- How to run tests (`make test`)
- Environment variables reference (derived from `.env.example`)

### Python projects — use `uv` (not `pip`)

If the project uses Python, `uv` is the required package manager. Apply this everywhere:

- **Project setup:** use `uv init`, `uv add`, and `uv sync` — never `pip install`
- **Lock file:** commit `uv.lock`; do not commit `requirements.txt` unless it is generated from `uv export`
- **All Dockerfiles generated in this project** must install `uv` and use it to install dependencies:

```dockerfile
FROM python:3.12-slim

# Install uv
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /usr/local/bin/

WORKDIR /app

# Install dependencies via uv (uses uv.lock for reproducibility)
COPY pyproject.toml uv.lock ./
RUN uv sync --frozen --no-dev

COPY . .

CMD ["uv", "run", "python", "-m", "your_module"]
```

- For development images, omit `--no-dev` to include dev dependencies
- Run application commands via `uv run <command>` rather than activating a virtualenv
- If any existing `Dockerfile` or script uses `pip`, replace it with the `uv` equivalent

### Project structure folders
Create the following when they make sense for the project type:
- `backend/` — backend application code (with its own `Dockerfile`)
- `frontend/` — frontend application code (with its own `Dockerfile`)
- `database/` — migration files, seed scripts, schema definitions

If any of these already contain code, do not create them — they exist.

---

## Audit Steps (run after scaffolding)

1. **Survey project files** — check for:
   - `README.md` with setup instructions and how to run tests
   - `.gitignore` and `.env.example` (not `.env`)
   - `LICENSE`
   - A CI configuration (`.github/workflows/`, `.circleci/`, etc.)

2. **Scan for hardcoded secrets** — search source files (excluding tests and examples) for patterns like `password =`, `api_key =`, `SECRET`, `token =`. Flag any that look like real values rather than env var references.

3. **Check dependency hygiene** — verify versions are pinned and a lock file is committed. For Python projects, flag any use of `pip install` in scripts, `Makefile`, or `Dockerfile` — these must be replaced with `uv`.

4. **Produce a Foundations Health Report**:

```
## Foundations Health Report

### Scaffolding created
- ...

### Green
- ...

### Yellow
- ...

### Red
- ...

### Recommended next steps
1. ...
```

## Arguments

- `--fix` — scaffold only, skip the audit report
- `--audit` — audit only, skip scaffolding
- No arguments — scaffold missing items first, then audit
