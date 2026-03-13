# syntax=docker/dockerfile:1
# Base Python image with WeasyPrint dependencies for Flask applications
# Used by: bungalow_flask_admin Dockerfile (builder stage)

ARG PYTHON_BASE_VERSION=3.13
FROM registry.berriait.com/campkeeper/python-base:${PYTHON_BASE_VERSION}

LABEL org.opencontainers.image.title="Campkeeper Flask Base"
LABEL org.opencontainers.image.description="Python base image with WeasyPrint system libraries"
LABEL org.opencontainers.image.vendor="Campkeeper"
LABEL org.opencontainers.image.source="https://github.com/campkeeper/base_images"

# Switch to root for package installation
USER root

# Install WeasyPrint system dependencies
# These are required for PDF generation with WeasyPrint
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    libcairo2-dev \
    libpango1.0-dev \
    libgdk-pixbuf-2.0-dev \
    libffi-dev \
    shared-mime-info && \
    rm -rf /var/lib/apt/lists/*

# Switch back to non-root user
USER appuser

# Metadata
ARG VERSION=3.13
ARG BUILD_DATE
ARG VCS_REF

LABEL org.opencontainers.image.version="${VERSION}"
LABEL org.opencontainers.image.created="${BUILD_DATE}"
LABEL org.opencontainers.image.revision="${VCS_REF}"

WORKDIR /workspace

CMD ["/bin/bash"]
