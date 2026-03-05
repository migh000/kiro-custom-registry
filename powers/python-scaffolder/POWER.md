---
displayName: Python Project Scaffolder
description: Scaffold a modern Python project with uv, ruff, pyright, pytest, and pre-commit
author: Platform Team
keywords:
  - python
  - scaffolding
  - uv
  - project-setup
---

# Python Project Scaffolder

This power helps you set up a new Python project following modern best practices.

## What it sets up

- **uv** for dependency management and virtual environments
- **ruff** for linting and formatting (replaces black, isort, flake8, pylint)
- **pyright** for type checking
- **pytest** for testing
- **pre-commit** for git hooks
- Proper `pyproject.toml` with all tool configs in one place
- `src/` layout
- GitHub Actions CI (optional)
- Dockerfile (optional)

## Usage

Just tell Kiro what you want to build:

- "scaffold a new python CLI tool called mytool"
- "set up a python web API with FastAPI"
- "create a python library for data processing"

The power will ask clarifying questions and generate everything.
