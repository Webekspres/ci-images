# ci-python: Python + uv untuk proyek FastAPI/Django dan skrip.
ARG PYTHON_VERSION=3.12
FROM docker.io/library/python:${PYTHON_VERSION}-bookworm

COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /usr/local/bin/
RUN apt-get update -qq \
    && apt-get install -y -qq --no-install-recommends ca-certificates git openssh-client rsync \
    && rm -rf /var/lib/apt/lists/* \
    && git config --system --add safe.directory '*'

ENV UV_LINK_MODE=copy PIP_DISABLE_PIP_VERSION_CHECK=1
WORKDIR /workspace
