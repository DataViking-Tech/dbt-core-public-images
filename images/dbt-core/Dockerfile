# syntax=docker/dockerfile:1

# Read python version from the build context:
#   variants/<context>/versions/python.txt
ARG PYTHON_VERSION_FILE=versions/python.txt

# dbt-core version is stored in-repo and will differ by git tag:
#   images/dbt-core/versions/dbt-core.txt
ARG DBT_VERSION_FILE=images/dbt-core/versions/dbt-core.txt

# Use a small base to keep devcontainers snappy.
FROM python:3.12-slim AS base
# We'll overwrite this in a second stage after reading python.txt; this keeps Dockerfile simple.

# ---- "selector" stage to choose python dynamically ----
FROM python:3.12-slim AS selector
ARG PYTHON_VERSION_FILE
RUN test -f "${PYTHON_VERSION_FILE}" && cat "${PYTHON_VERSION_FILE}" | tr -d '\n' > /python_version

# ---- actual runtime image ----
# Re-declare so we can use it
FROM python:3.12-slim

# Copy selected python version string from selector
COPY --from=selector /python_version /python_version
RUN PYV="$(cat /python_version)" && \
    echo "Using Python ${PYV}" && \
    sed -i "s/^deb /deb /" /etc/apt/sources.list

# Install system deps commonly needed for dbt + adapters (safe default).
# Trim later if you want.
RUN apt-get update && apt-get install -y --no-install-recommends \
      git ca-certificates \
    && rm -rf /var/lib/apt/lists/*

ARG DBT_VERSION_FILE
RUN test -f "${DBT_VERSION_FILE}" && cat "${DBT_VERSION_FILE}" | tr -d '\n' > /dbt_version

# Install dbt-core pinned to repo version file
RUN DBT_VERSION="$(cat /dbt_version)" && \
    python -m pip install --upgrade pip setuptools wheel && \
    pip install --no-cache-dir "dbt-core==${DBT_VERSION}"

# A couple nice-to-haves for devcontainers
RUN useradd -m -u 1000 vscode
USER vscode
WORKDIR /workspace

CMD ["python", "--version"]
