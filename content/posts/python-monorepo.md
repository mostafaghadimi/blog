---
title: "مونورپوی پایتون که کارت رو سخت نکنه"
date: 2026-04-18
description: "یک ساختار عملی برای مونورپوی پایتون — مرزهای واضح، نصب سریع، دپلوی مستقل و یک CI که با هر push همه چیز را از نو نمی‌سازد."
tags: ["python", "monorepo", "uv", "docker", "github-actions"]
categories: ["پایتون"]
author: "مصطفی قدیمی"
showToc: true
---

Python monorepos have a reputation for being painful. Slow installs, muddy dependency graphs, one broken package blocking all your tests. It doesn't have to be this way.

Here's the layout I've settled on after a few iterations.

---

## The Core Principle: Share by Package, Not by Path

The tempting approach is to put shared code in a `libs/` folder and add it to `PYTHONPATH`. Don't. It makes imports implicit, breaks IDE tooling, and is invisible to dependency managers.

Instead, publish shared code as an **editable local package**:

```
monorepo/
├── packages/
│   └── core/              # shared utilities, models, clients
│       ├── pyproject.toml
│       └── src/core/
├── apps/
│   ├── api/
│   │   ├── pyproject.toml  # depends on packages/core
│   │   └── src/api/
│   └── worker/
│       ├── pyproject.toml
│       └── src/worker/
├── pyproject.toml         # root — dev tools only (ruff, mypy, pytest)
└── uv.lock
```

Each app declares `packages/core` as a dependency:

```toml
[project]
dependencies = ["core", "fastapi>=0.110"]

[tool.uv.sources]
core = { workspace = true }
```

## Why uv

[uv](https://github.com/astral-sh/uv) resolves the full monorepo dependency graph in one lockfile and installs it in seconds. `uv sync --package api` installs only what `api` needs.

```bash
uv sync
uv run --package api uvicorn api.main:app
uv add --package worker celery
```

## Per-app Dockerfiles

```dockerfile
FROM python:3.12-slim AS base
WORKDIR /app

COPY uv.lock pyproject.toml ./
COPY packages/core ./packages/core
COPY apps/api ./apps/api

RUN pip install uv && \
    uv sync --package api --no-dev --frozen

CMD ["uv", "run", "--package", "api", "uvicorn", "api.main:app", "--host", "0.0.0.0"]
```

## Targeted CI

```yaml
# .github/workflows/api.yml
on:
  push:
    paths:
      - "apps/api/**"
      - "packages/core/**"
      - "uv.lock"
```

Only the API pipeline triggers when `apps/api` or `packages/core` changes. The worker stays quiet.
