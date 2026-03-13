# Base Images for Campkeeper

Centralized Docker base images repository for Campkeeper projects. These pre-built images significantly reduce build times and ensure consistency across all services.

## 📦 Available Images

### 1. CI Image (`campkeeper/ci`)
**Purpose**: CI/CD pipelines for linting, testing, and security scanning  
**Base**: python:3.13-slim  
**Tags**: `latest`, `YYYY-MM-DD`, `sha-<commit>`  
**Build**: Weekly (Mondays 2 AM UTC) + on push  

**Contains**:
- Python 3.13
- uv (package manager)
- ruff ≥0.8.0 (linting & formatting)
- bandit ≥1.7.0 (security scanning)
- mypy ≥1.8.0 (type checking)
- pytest ≥8.3.5 (testing)
- jq (JSON processing)
- git, curl, ca-certificates

**Used by**: 
- `bungalow_backend/.github/workflows/ci.yml` (lint, security jobs)
- `bungalow_flask_admin/.github/workflows/ci.yml` (lint job)

**Impact**: Eliminates `uv sync --dev` in CI (~3min saved per job)

---

### 2. Python Base Image (`campkeeper/python-base`)
**Purpose**: Builder stage for application Dockerfiles  
**Base**: python:3.13-slim  
**Tags**: `3.13`, `3.13-YYYY-MM-DD`, `3.13-sha-<commit>`, `latest`  
**Build**: Monthly (1st, 3 AM UTC) + on push  

**Contains**:
- Python 3.13
- build-essential (gcc, g++, make)
- uv (package manager)
- git, curl, ca-certificates

**Used by**: 
- `bungalow_backend/infrastructure/container/Dockerfile` (builder stage)

**Impact**: Removes apt-get install from Dockerfiles (~1-2min per build)

---

### 3. Flask Base Image (`campkeeper/flask-base`)
**Purpose**: Builder stage for Flask applications with WeasyPrint  
**Base**: campkeeper/python-base:3.13  
**Tags**: `3.13`, `3.13-YYYY-MM-DD`, `3.13-sha-<commit>`, `latest`  
**Build**: Monthly (1st, 4 AM UTC) + after python-base updates + on push  

**Contains** (in addition to python-base):
- libcairo2-dev (Cairo graphics)
- libpango1.0-dev (text rendering)
- libpangocairo-1.0-dev (Pango-Cairo bindings)
- libgdk-pixbuf-2.0-dev (image loading)
- libffi-dev (FFI library)
- shared-mime-info (MIME type detection)

**Used by**: 
- `bungalow_flask_admin/infrastructure/container/Dockerfile` (builder stage)

**Impact**: Pre-compiles 7 system packages for WeasyPrint (~4min saved per build)

---

## 🚀 Quick Start

### Using in Your Dockerfile

**Backend** (minimal dependencies):
```dockerfile
FROM registry.berriait.com/campkeeper/python-base:3.13 AS builder
# No need for apt-get install build-essential curl
COPY pyproject.toml uv.lock ./
RUN uv sync --frozen --no-dev
```

**Flask Admin** (with WeasyPrint):
```dockerfile
FROM registry.berriait.com/campkeeper/flask-base:3.13 AS builder
# All WeasyPrint libs already installed
COPY pyproject.toml uv.lock ./
RUN uv sync --frozen --no-dev
```

**CI Workflow**:
```yaml
jobs:
  lint:
    runs-on: [self-hosted, linux, x64, docker]
    container:
      image: registry.berriait.com/campkeeper/ci:latest
      credentials:
        username: ${{ secrets.REGISTRY_USERNAME }}
        password: ${{ secrets.REGISTRY_PASSWORD }}
    steps:
      - uses: actions/checkout@v4
      # No need for uv sync --dev, tools already installed
      - run: ruff check .
      - run: bandit -r app/
```

---

## 🔧 Building Images

### Automated Builds

Images are automatically built on schedule:
- **CI**: Weekly (Mondays 2 AM UTC)
- **Python Base**: Monthly (1st, 3 AM UTC)
- **Flask Base**: Monthly (1st, 4 AM UTC) + after python-base updates

### Manual Builds

Navigate to Actions tab and trigger via `workflow_dispatch`:

1. **Build CI Image**: `.github/workflows/build-ci.yml`
   - Optional: specify custom tag suffix

2. **Build Python Base**: `.github/workflows/build-python-base.yml`
   - Optional: specify Python version (default: 3.13)

3. **Build Flask Base**: `.github/workflows/build-flask-base.yml`
   - Optional: specify Python version (default: 3.13)

---

## 🔐 Security

All images are scanned with **Trivy** for vulnerabilities:
- Severity: CRITICAL, HIGH
- Build fails if vulnerabilities found (exit code 1)
- Images pushed only after passing security scan

**Scan locally**:
```bash
trivy image registry.berriait.com/campkeeper/ci:latest
trivy image registry.berriait.com/campkeeper/python-base:3.13
trivy image registry.berriait.com/campkeeper/flask-base:3.13
```

---

## 📊 Performance Impact

| Project | Before | After | Savings |
|---------|--------|-------|---------|
| Backend (CI lint) | ~5min | ~2min | **-60%** |
| Backend (build) | ~8min | ~6min | **-25%** |
| Flask Admin (CI lint) | ~6min | ~3min | **-50%** |
| Flask Admin (build) | ~12min | ~5min | **-58%** |

**Total time saved per full CI run**: ~15 minutes across both projects

---

## 🛠️ Development

### Directory Structure

```
base_images/
├── .github/workflows/
│   ├── build-ci.yml              # CI image build workflow
│   ├── build-python-base.yml     # Python base build workflow
│   └── build-flask-base.yml      # Flask base build workflow
├── ci/
│   └── Dockerfile                # CI image definition
├── python-base/
│   └── Dockerfile                # Python base image definition
├── flask-base/
│   └── Dockerfile                # Flask base image definition
├── .gitignore
└── README.md                     # This file
```

### Testing Changes Locally

**CI Image**:
```bash
cd ci/
docker build -t campkeeper/ci:test \
  --build-arg BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
  --build-arg VCS_REF=$(git rev-parse --short HEAD) \
  .
docker run --rm -it campkeeper/ci:test bash
```

**Python Base**:
```bash
cd python-base/
docker build -t campkeeper/python-base:test \
  --build-arg VERSION=3.13 \
  --build-arg BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
  .
```

**Flask Base**:
```bash
cd flask-base/
docker build -t campkeeper/flask-base:test \
  --build-arg PYTHON_BASE_VERSION=3.13 \
  .
```

### Verifying Tools in CI Image

```bash
docker run --rm campkeeper/ci:test bash -c "
  python --version && \
  uv --version && \
  ruff --version && \
  bandit --version && \
  mypy --version && \
  pytest --version && \
  jq --version
"
```

---

## 🔄 Update Strategy

### Python Version Updates

When updating to Python 3.14+:

1. Update `python-base/Dockerfile` base image to `python:3.14-slim`
2. Trigger `build-python-base` workflow with `python_version=3.14`
3. This automatically triggers `build-flask-base` with version 3.14
4. Update application Dockerfiles to use `:3.14` tag
5. Test deployments before switching default `:latest` tag

### Tool Version Updates

**CI Image tools** (ruff, bandit, mypy, pytest):
- Edit version constraints in `ci/Dockerfile`
- Push to main → automatic rebuild
- Test in one project before rolling out

**System dependencies** (python-base, flask-base):
- Rare changes needed
- Test locally before pushing
- Consider compatibility with production applications

---

## 📝 Migration Checklist

When migrating a project to use these images:

- [ ] Update Dockerfile builder stage to use base image
- [ ] Remove redundant `apt-get install` commands
- [ ] Remove `uv` installation (already in base)
- [ ] Test local build with new base
- [ ] Update CI workflow if using CI image
- [ ] Remove `uv sync --dev` if CI image has pre-installed tools
- [ ] Test CI pipeline on feature branch
- [ ] Measure and document build time improvements
- [ ] Deploy to preprod and verify functionality
- [ ] Merge to main after validation

---

## 🐛 Troubleshooting

### Image pull fails with 403 Forbidden

Ensure `--network=host` is set when using Kaniko in self-hosted runners:
```yaml
docker run --rm \
  --network=host \
  -v $(pwd):/workspace \
  gcr.io/kaniko-project/executor:latest ...
```

### Trivy not found during workflow

Trivy is auto-installed in workflows. If issues persist:
```bash
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | \
  sh -s -- -b /usr/local/bin
```

### WeasyPrint import errors in Flask Admin

Ensure production stage still installs **runtime** libraries:
```dockerfile
# Production stage - still needed!
RUN apt-get install -y --no-install-recommends \
  libcairo2 libpango-1.0-0 libpangocairo-1.0-0 \
  libgdk-pixbuf-2.0-0 libffi8 shared-mime-info
```

---

## 📚 References

- [Docker Multi-stage Builds](https://docs.docker.com/build/building/multi-stage/)
- [Kaniko Documentation](https://github.com/GoogleContainerTools/kaniko)
- [Trivy Security Scanner](https://trivy.dev/)
- [uv - Python Package Manager](https://github.com/astral-sh/uv)
- [WeasyPrint Requirements](https://doc.courtbouillon.org/weasyprint/stable/first_steps.html)

---

## 📄 License

Internal use only - Campkeeper project

---

## 🤝 Contributing

1. Create feature branch from `main`
2. Make changes to Dockerfile(s)
3. Test locally with `docker build`
4. Update this README if adding new images
5. Open PR with description of changes
6. Ensure CI workflows pass
7. Request review from infrastructure team
