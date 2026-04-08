# syntax=docker/dockerfile:1
# Base Python image with build tools for application builds
# Used by: bungalow_backend and bungalow_flask_admin Dockerfiles (builder stage)

FROM python:3.13-slim

LABEL org.opencontainers.image.title="Campkeeper Python Base"
LABEL org.opencontainers.image.description="Python 3.13 with build-essential and common build tools"
LABEL org.opencontainers.image.vendor="Campkeeper"
LABEL org.opencontainers.image.source="https://github.com/campkeeper/base_images"

# Install build dependencies
RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y --no-install-recommends \
    build-essential \
    curl \
    git \
    ca-certificates && \
    rm -rf /var/lib/apt/lists/*

# Install uv (Python package manager)
RUN curl -LsSf https://astral.sh/uv/install.sh | sh && \
    mv /root/.local/bin/uv /usr/local/bin/uv && \
    mv /root/.local/bin/uvx /usr/local/bin/uvx

# Create non-root user for builds
RUN groupadd -g 1002 appuser && \
    useradd -r -u 1002 -g appuser -m -d /home/appuser -s /bin/bash appuser

# Metadata
ARG VERSION=3.13
ARG BUILD_DATE
ARG VCS_REF

LABEL org.opencontainers.image.version="${VERSION}"
LABEL org.opencontainers.image.created="${BUILD_DATE}"
LABEL org.opencontainers.image.revision="${VCS_REF}"

WORKDIR /workspace
USER appuser

CMD ["/bin/bash"]
