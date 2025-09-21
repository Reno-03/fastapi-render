# 🚀 FastAPI + Poetry + Render Deployment Guide

This guide explains how we set up and fixed deployment of a **FastAPI**
project to **Render**, using **Poetry** for dependency management.

------------------------------------------------------------------------

## 📂 Project Structure

After fixing the layout, the project looks like this:

    fastapi-render/
    ├── README.md
    ├── poetry.lock
    ├── pyproject.toml
    ├── server.py          # FastAPI entrypoint
    └── orders.db          # (optional, SQLite for local dev)

We keep `server.py` at the repo root (no package folder needed).

------------------------------------------------------------------------

## ⚙️ pyproject.toml

We updated `pyproject.toml` to disable Poetry's packaging mode (since
this is an app, not a library):

``` toml
[tool.poetry]
name = "fastapi-render"
version = "0.1.0"
description = "Example of FastAPI deployment to render.com"
authors = ["Your Name <you@example.com>"]
readme = "README.md"
package-mode = false   # 👈 important

[tool.poetry.dependencies]
python = "^3.10"
fastapi = "^0.110.2"
uvicorn = "^0.29.0"
sqlalchemy = "^2.0.31"
psycopg2-binary = "^2.9.9"

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"
```

------------------------------------------------------------------------

## 🛠 Environment Variables on Render

In Render's dashboard, add an environment variable:

    db_uri = postgresql+psycopg2://USERNAME:PASSWORD@HOST:PORT/DBNAME

> ⚠️ **Important**: Use `postgresql+psycopg2://` (not `-binary`).

------------------------------------------------------------------------

## 🖥️ Database Config in `server.py`

We load the database URL from the environment and provide a safe
fallback for local dev:

``` python
import os
from sqlalchemy import create_engine

db_url = os.getenv("db_uri", "sqlite:///orders.db")
print("Using DB:", db_url)

engine = create_engine(db_url, echo=True)
```

------------------------------------------------------------------------

## 🏗️ Render Service Configuration

**Build Command**

``` bash
pip install -U poetry && poetry install --no-root
```

**Start Command**

``` bash
poetry run uvicorn server:app --host 0.0.0.0 --port $PORT
```

------------------------------------------------------------------------

## ▶️ Local Development

1.  Install dependencies:

    ``` bash
    poetry install --no-root
    ```

2.  Run FastAPI locally:

    ``` bash
    poetry run uvicorn server:app --reload
    ```

3.  Access the API at:

        http://127.0.0.1:8000

------------------------------------------------------------------------

## ✅ Key Fixes Along the Way

1.  **Poetry packaging error**\
    → Solved by adding `package-mode = false`.

2.  **Missing `pyproject.toml` on Render**\
    → Fixed by ensuring it lives in the repo root.

3.  **Invalid SQLAlchemy DB URL**\
    → Fixed by changing from\
    `postgresql+psycopg2-binary://...` → `postgresql+psycopg2://...`.

4.  **Start command issues**\
    → Finalized with\
    `poetry run uvicorn server:app --host 0.0.0.0 --port $PORT`.

------------------------------------------------------------------------

✨ That's it! With this setup, the app runs locally (SQLite fallback)
and deploys on Render with Postgres.
