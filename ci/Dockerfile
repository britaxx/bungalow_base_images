# syntax=docker/dockerfile:1
# Base image for CI/CD pipelines with all development tools
# Used by: bungalow_backend and bungalow_flask_admin CI workflows

FROM python:3.13-slim

LABEL org.opencontainers.image.title="Campkeeper CI Image"
LABEL org.opencontainers.image.description="CI/CD image with Python dev tools: uv, ruff, bandit, mypy, pytest"
LABEL org.opencontainers.image.vendor="Campkeeper"
LABEL org.opencontainers.image.source="https://github.com/campkeeper/base_images"

# Install system dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    git \
    curl \
    jq \
    ca-certificates && \
    rm -rf /var/lib/apt/lists/*

# Install uv (Python package manager)
RUN curl -LsSf https://astral.sh/uv/install.sh | sh && \
    mv /root/.cargo/bin/uv /usr/local/bin/uv

# Create virtual environment for CI tools
ENV VIRTUAL_ENV=/opt/ci-venv
RUN python -m venv $VIRTUAL_ENV
ENV PATH="$VIRTUAL_ENV/bin:$PATH"

# Install CI/CD tools
RUN pip install --no-cache-dir \
    ruff>=0.8.0 \
    bandit[toml]>=1.7.0 \
    mypy>=1.8.0 \
    pytest>=8.3.5 \
    safety>=3.0.0

# Create non-root user matching GitHub Actions runner
RUN groupadd -g 1002 appuser && \
    useradd -r -u 1002 -g appuser -m -d /home/appuser -s /bin/bash appuser && \
    mkdir -p /home/appuser/.cache && \
    chown -R appuser:appuser /home/appuser

# Configure git for user
USER appuser
RUN git config --global --add safe.directory '*' && \
    git config --global user.email "ci@campkeeper.local" && \
    git config --global user.name "CI User"

WORKDIR /workspace

# Metadata
ARG VERSION=latest
ARG BUILD_DATE
ARG VCS_REF

LABEL org.opencontainers.image.version="${VERSION}"
LABEL org.opencontainers.image.created="${BUILD_DATE}"
LABEL org.opencontainers.image.revision="${VCS_REF}"

CMD ["/bin/bash"]
