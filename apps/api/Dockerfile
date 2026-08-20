# syntax=docker/dockerfile:1

FROM ghcr.io/astral-sh/uv:0.11.7 AS uv

FROM python:3.12-slim-trixie AS builder
COPY --from=uv /uv /uvx /usr/local/bin/

ENV UV_PYTHON_DOWNLOADS=never

WORKDIR /app

COPY pyproject.toml uv.lock ./
RUN uv sync --frozen --no-dev --no-install-project

COPY app ./app
RUN uv sync --frozen --no-dev

FROM python:3.12-slim-trixie

RUN apt-get update \
    && apt-get install --only-upgrade -y util-linux \
    && rm -rf /var/lib/apt/lists/*

RUN useradd --create-home --uid 1000 appuser
WORKDIR /app

COPY --from=builder /app/.venv .venv
COPY --from=builder /app/app ./app

ENV PATH="/app/.venv/bin:$PATH"

USER appuser
EXPOSE 8000

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]