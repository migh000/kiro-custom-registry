---
inclusion: auto
---

# Local Power Guide

This steering file was loaded from a locally-served power.
When you see this, the local registry setup is working.

### 2.4 Add dev dependencies with uv

After creating pyproject.toml, run:

```bash
uv add --dev ruff pyright pytest pre-commit
```

For `api` type, also run:
```bash
uv add fastapi 'uvicorn[standard]'
```

For `cli` type with argument parsing:
```bash
uv add typer
```

### 2.5 .pre-commit-config.yaml

```yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.9.10
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format
```

### 2.6 .gitignore

Use a standard Python .gitignore. Must include at minimum:

```
__pycache__/
*.py[cod]
*.egg-info/
dist/
build/
.venv/
.pytest_cache/
.ruff_cache/
.pyright/
```

### 2.7 Optional: GitHub Actions CI

If requested, create `.github/workflows/ci.yml`:

```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v5
      - run: uv sync --dev
      - run: uv run ruff check .
      - run: uv run ruff format --check .
      - run: uv run pyright
      - run: uv run pytest
```

### 2.8 Optional: Dockerfile

If requested, create a multi-stage `Dockerfile`:

```dockerfile
FROM ghcr.io/astral-sh/uv:python3.12-bookworm-slim AS builder
WORKDIR /app
COPY pyproject.toml uv.lock ./
RUN uv sync --frozen --no-dev --no-install-project
COPY . .
RUN uv sync --frozen --no-dev

FROM python:3.12-slim-bookworm
WORKDIR /app
COPY --from=builder /app/.venv .venv
COPY --from=builder /app/src src
ENV PATH="/app/.venv/bin:$PATH"
```

For `api` type, add:
```dockerfile
EXPOSE 8000
CMD ["uvicorn", "<package_name>.app:app", "--host", "0.0.0.0"]
```

For `cli` type, add:
```dockerfile
ENTRYPOINT ["<project-name>"]
```

## Step 3: Initialize and verify

After generating all files:

```bash
uv sync
uv run ruff check .
uv run pytest
```

Run these and fix any issues before telling the user you're done.

## Step 4: Summary

Tell the user what was created and how to get started:

- `uv run <project-name>` (for cli)
- `uv run uvicorn <package_name>.app:app --reload` (for api)
- `uv run pytest` to run tests
- `uv run ruff check .` to lint
- `pre-commit install` to enable git hooks

## Rules

- ALWAYS use `src/` layout, never flat layout
- ALWAYS configure all tools in `pyproject.toml`, never separate config files
- ALWAYS use `uv add` to add dependencies, never edit pyproject.toml dependencies by hand
- NEVER use pip, poetry, pipenv, or conda
- NEVER use black, isort, flake8, or pylint — ruff replaces all of them
- NEVER create `setup.py` or `setup.cfg`
- Use type hints everywhere in generated code
- All generated code must pass ruff and pyright with zero errors
